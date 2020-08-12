---
title: "EKS"
linkTitle: "EKS"
description: >
  Installation instructions to deploy KRE on EKS.
weight: 30
---
# EKS deployment

The flavor of Kubernetes on AWS is called EKS (Elastic Kubernetes Service) which allow to deploy a cluster managed by Amazon. This means that Amazon will manage the lifecyle of the Master nodes of our cluster. 

Currently there are two ways of run loads on top of EKS, using EC2 instances as compute nodes that are added to to the cluster or using the Fargate mode, where AWS also manage these compute nodes. In this guide is just described the first one, adding our own compute nodes with EC2 instances.

Deploy an EKS cluster is not the goal of this guide, only the detail some specific configuration needed to run KRE on top of it. It is recomend to use IaC (Infrastructure As Code) approach using Terraform to automate the creation of your cluster, [here](https://learn.hashicorp.com/tutorials/terraform/eks) you can find usefull resources about that. Also you can follow the instructions from the official [AWS site](https://docs.aws.amazon.com/eks/latest/userguide/create-cluster.html). 

The final EKS deployment should be something like the below diagram.

{{< imgproc eks_diagram Resize "600x" >}}

{{< /imgproc >}}

# Storage

An important amount of features of KRE are based on the use of shared storage with `ReadWriteMany` volumes. Therefore is required to add a storageClass to Kubernetes that support this kind of volumes. 

In AWS there are a service called EFS (Elastic File System) that bring to us a network shared storage. As was mentioned before, the recomended way to create resources is using the approach of IaC, for EFS you can find examples of Terraform code [here](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/efs_mount_target), or follow the manual steps from the [AWS site](https://docs.aws.amazon.com/efs/latest/ug/whatisefs.html). 

The common way to use this from Kubernetes is deploying what is called `efs-provisioner` that create the interface between Kubernetes `PersistentVolumeClaim` and EFS. 

In our experience we have had some issues with the `efs-provisioner`, therefore instead of deploy an `efs-provisioner` to support the creation of volumes on EFS we prefer to add a script to the `UserData` of each EC2 instance to mount the shared EFS on a local mount point, for example on `/mnt/efs/kre`, and create a `HostPath` storageClass that will create all the volumes within this path. This way we can create `ReadWriteMany` volumes that are accesible from all the nodes of our cluster. The `UserData` script example is below, and is good practice to set it in the `Launch Configuration` that manage the EC2 instance which are the compute nodes of our cluster.

```bash
#!/bin/bash
set -o xtrace
/etc/eks/bootstrap.sh --apiserver-endpoint 'https://xxxxxxxxxxx.gr7.us-east-1.eks.amazonaws.com' --b64-cluster-ca 'xxxxxxxxxxxxx' 'kre'
mkdir -p /mnt/efs/kre
yum install amazon-efs-utils -y
echo "fs-xxxxxxxx.efs.us-east-1.amazonaws.com:/ /mnt/efs/kre efs tls,_netdev" >> /etc/fstab
mount -a -t efs defaults
```
In the next section is described how to install the `hostPath` provisioner.

Network shared storage can be a bottle neck of performance, so depend of your usecase you should use only for the pieces that require 
`ReadWriteMany` volume, for the rest of the components you can use the default storageClass that will create an EBS resource on AWS. In the [Helm deployment](#helm-deployment) section is detailed the best option for your usecase and which pieces can use which storageClass.

# Kubernetes required components

Some additional components are required on Kubernetes to get a full featured KRE deployment running. We are going to describe how to install those.

## Ingress controller

The use of Ingress Controller in Kubernetes that are deployed on cloud providers is very common, due to the reduction of costs on
load balancers, also the ingress objects help with the automation of some task when publish service to outside of our cluster, and many more.

There are multiple choices of Ingress Controller (NGINX, Traefik, HAProxy, Kong, ...), you can find a full list of those 
in the [Kubernetes site](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/), and all of them have pros and cons, in this guide we are going to explain how to deploy NGINX Ingress Controller. It is posible to use KRE with other Ingress Controller than NGINX, but this is matained by the CNCF and the maturity of NGINX itself is quite important.



```bash
helm upgrade --install \
     --namespace kube-system \
     --version 1.40.2 \
     nginx-ingress \
     stable/nginx-ingress
```

## Cert manager

The access to the web admin interface of KRE and all the endpoints that are exposed to the end users required of a minimun level of security,
this is the reason why add a piece to automate the management of the lifecycle of all required certificates.

Cert Manager allow to create certificates just with some configuration on the deployment process, and manage the lifecycle of those, updating those when expired without any human interaction. Moreover, with
the use of `DNS01` challenge method can be created certificates for environment that are not exposed to Internet behind a firewall in a private network.

In order to install Cert Manager we are going to use the official Helm chart. Below are the commands to deploy it within the Kubernetes namespace `cert-manager`, be aware that this is just a convention, you can deploy it in the namespace where you feel more confortable.

```bash
kubectl create namespace cert-manager --dry-run -o yaml | kubectl apply -f -
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm upgrade --install \
     --namespace cert-manager \
     --version v0.15.0 \
     --set installCRDs=true \
     cert-manager \
     jetstack/cert-manager
```

## Storage provisioner

In order to create the `hostPath` provisioner just install the Helm chart as shown below.

```bash
helm upgrade --install hostpath-provisioner --namespace kube-system rimusz/hostpath-provisioner
```

# DNS

All the access to KRE services require of hostnames, due to the use of Ingress object and certificates. In order to get a 
deployment and management processes easier we recommend to delegate a subdomain to a `Route53` DNS zone, and create a wildcard 
entry pointing to the Load Balancer of the Ingress Controller.

# Helm deployment


