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

Deploy an EKS cluster is not the goal of this guide, only the detail some specific configuration needed to run KRE on top of it. It is recommend to use IaC (Infrastructure As Code) approach using Terraform to automate the creation of your cluster, [here](https://learn.hashicorp.com/tutorials/terraform/eks) you can find usefull resources about that. Also you can follow the instructions from the official [AWS site](https://docs.aws.amazon.com/eks/latest/userguide/create-cluster.html). 

The final EKS deployment should be something like the below diagram.

{{< imgproc eks_diagram Resize "600x" >}}

{{< /imgproc >}}

After deploy your EKS cluster you are going to need the `kubeconfig` file. This file is the way to configure the `kubectl` and `helm` CLIs to access to your cluster. In the case of EKS used to be required an extra plugin called [AWS IAM authenticator](https://github.com/kubernetes-sigs/aws-iam-authenticator) to authenticate via the IAM account. Please follow the steps detailed in hte [AWS site](https://docs.aws.amazon.com/eks/latest/userguide/create-kubeconfig.html).

# Storage

An important amount of features of KRE are based on the use of shared storage with `ReadWriteMany` volumes. Therefore, is required to add a storageClass to Kubernetes that support this kind of volumes. 

In AWS there are a service called EFS (Elastic File System) that bring to us a network shared storage. As was mentioned before, the recommended way to create resources is using the approach of IaC, for EFS you can find examples of Terraform code [here](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/efs_mount_target), or follow the manual steps from the [AWS site](https://docs.aws.amazon.com/efs/latest/ug/whatisefs.html). 

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

Network shared storage can be a bottleneck of performance, so depends on your usecase you should use only for the pieces that require 
`ReadWriteMany` volume, for the rest of the components you can use the default storageClass that will create an EBS resource on AWS. In the [Helm deployment](#helm-deployment) section is detailed the best option for your usecase and which pieces can use which storageClass.

# Kubernetes required components

Some additional components are required on Kubernetes to get a full featured KRE deployment running. We are going to describe how to install those.

## Ingress controller

The use of Ingress Controller in Kubernetes that are deployed on cloud providers is very common, due to the reduction of costs on
load balancers, also the ingress objects help with the automation of some task when publish service to outside of our cluster, and many more.

There are multiple choices of Ingress Controller (NGINX, Traefik, HAProxy, Kong, ...), you can find a full list of those 
in the [Kubernetes site](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/), and all of them have pros and cons, in this guide we are going to explain how to deploy NGINX Ingress Controller. It is possible to use KRE with other Ingress Controller than NGINX, but this is maintained by the CNCF, and the maturity of NGINX itself is quite important.



```bash
helm upgrade --install \
     --namespace kube-system \
     --version 1.40.2 \
     nginx-ingress \
     stable/nginx-ingress
```

## Cert manager

The access to the web admin interface of KRE and all the endpoints that are exposed to the end users required of a minimum level of security,
this is the reason why add a piece to automate the management of the lifecycle of all required certificates.

Cert Manager allow creating certificates just with some configuration on the deployment process, and manage the lifecycle of those, updating those when expired without any human interaction. Moreover, with
the use of `DNS01` challenge method can be created certificates for environment that are not exposed to Internet behind a firewall in a private network.

In order to install Cert Manager we are going to use the official Helm chart. Below are the commands to deploy it within the Kubernetes namespace `cert-manager`, be aware this is just a convention, you can deploy it in the namespace where you feel more comfortable.

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

All the access to KRE services require of a hostname due to the use of Ingress objects and certificates. In order to get a 
deployment and management processes easier we recommend delegating a subdomain `kre` of a domain owned by you to a `Route53` DNS hosted zone, and create a wildcard entry pointing to the Load Balancer of the Ingress Controller.

## Get Ingress Controller hostname

First of all you need to konw the name of the ELB where we have to point our DNS entry. With the below command you will get name of this Load Balancer.

```bash
kubectl -n kube-system get svc -l app=nginx-ingress,component=controller -o jsonpath="{.items[0].status.loadBalancer.ingress[0].hostname}"
```

The output of this command should be something like `xxxxxxxxxxxxxxxxxxxxx-00000000.us-east-1.elb.amazonaws.com`.

## Create wildcard entry in your hosted zone

With the name of the ELB go to your hosted zone in Route53 and create a new A entry of type Alias pointing to the ELB name. That's it.

Depend of your environment you can use the `DNS01` challenge to validate that you are the owner of the DNS name, in that case with Cert Manager there are a plugin that allow to perform this validation with a Route53, so this is another interesting point to take in account. In the next section is detailed how to automate this in the KRE deployment.

After a while the resolution of your domain should point to the ELB. KRE require of the following subdomains.

| Subdomain    | Description                                            |
| ------- | ----------------------------------------------------------- | 
| `admin.kre.yourdomain.com` | Web admin console  | 
| `api.kre.yourdomain.com` | API  |
 

## Validate 

To validate that the DNS configuration is working fine you can use the tool `dig` to query the DNS as follow.

```bash
dig admin.kre.yourdamin.com
```
The output should be something as below.

```
; <<>> DiG 9.16.1-Ubuntu <<>> admin.kre.yourdamin.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 901
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;admin.kre.yourdamin.com.	IN	A

;; ANSWER SECTION:
admin.kre.yourdamin.com.	1799 IN	A	1.2.3.4

;; Query time: 64 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
;; WHEN: mié ago 12 16:18:49 CEST 2020
;; MSG SIZE  rcvd: 75

```

# Helm deployment

Once you have your EKS cluster ready with all the required extra components installed and all the credentials to access to your cluster is time to start the KRE deployment. The first step is to define a `values.yaml` file that fit all the requirements for our installation. So we are going to describe an example of this file to deploy KRE in your cluster. After that we can apply the Helm chart with this value to deploy KRE and afterward we are going to describe how to validate the installation.

## Create values.yaml

```yaml
config:
  smtp:
    enabled: true
    sender: "<YOUR_SMTP_EMAIL_ACCOUNT>"
    senderName: "<SENDER_NAME>"
    user: "<SMTP_USER>"
    pass: "<SMTP_PASSWORD>"
    host: "<SMTP_HOST>"
    port: "<SMTP_PORT>"
  baseDomainName: kre."<YOURDOAMIN>"
  admin:
    apiBaseURL: api.kre."<YOURDOAMIN>"
    frontendBaseURL: https://admin.kre."<YOURDOAMIN>"
  runtime:
    sharedStorageClass: hostpath
    sharedStorageSize: 10Gi
    nats_streaming:
      storage:
        className: gp2
        size: 2Gi
    mongodb:
      persistentVolume:
        enabled: true
        storageClass: gp2
        size: 5Gi
    chronograf:
      persistentVolume:
        enabled: true
        storageClass: gp2
        size: 1Gi
    influxdb:
      persistentVolume:
        enabled: true
        storageClass: gp2
        size: 5Gi
  acme:
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    email: "<YOUR_EMAIL_ADDRESS>"
  auth:
    verificationCodeDurationInMinutes: 5
    jwtSignSecret: int_jwt_secret
    secureCookie: true
    cookieDomain: kre."<YOURDOAMIN>"

adminApi:
  tls:
    enabled: true
  host: api.kre."<YOUR_DOAMIN>"
  storage:
    class: hostpath

adminUI:
  tls:
    enabled: true
  host: admin.kre."<YOURDOAMIN>"

mongodb:
  mongodbDatabase: "KRE"
  rootCredentials:
    username: "admin"
    password: "<MONGODB_PASS>"
  image:
    repository: mongo
    tag: 4.2.8
  persistence:
    mountPath: /data/db
  storage:
    className: gp2
    size: 6Gi

certManager:
  enabled: true
  acme:
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    email: "<YOUR_EMAIN_ADDRESS>"
  dns01:
    route53:
      region: "<AWS_REGION>"
      hostedZoneID: "<AWS_ROUTE53_HOSTEDZONE_ID>"
      accessKeyID: "<AWS_ACCESS_KEY_ID>"
      secretAccessKey: "<AWS_ACCESS_SECRET>"
```

## Install Helm chart

```bash
kubectl create namespace kre
helm repo add konstellation-io https://charts.konstellation.io
helm repo update
helm upgrade --install kre --namespace kre -values values.yaml konstellation-io/kre
```

# Validate the installation

## Check pods on KRE namespace

Command:

```bash
kubectl -n kre get pods
```
Expected output:

```
NAME                                                     READY   STATUS    RESTARTS   AGE
alertmanager-kre-local-prometheus-opera-alertmanager-0   2/2     Running   0          28h
kre-local-admin-api-75d8bdc6b9-n4qcv                     1/1     Running   5          27h
kre-local-admin-ui-5d96987f95-9g59q                      1/1     Running   0          27h
kre-local-grafana-67f89f8977-pqdmw                       2/2     Running   0          28h
kre-local-k8s-manager-bddc586c4-kjhgf                    1/1     Running   0          27h
kre-local-kube-state-metrics-78fbbcbfb8-64ds8            1/1     Running   0          28h
kre-local-prometheus-node-exporter-sx9sb                 1/1     Running   0          28h
kre-local-prometheus-opera-operator-5b6f794b67-8lrdj     2/2     Running   0          28h
kre-mongo-0                                              1/1     Running   0          28h
prometheus-kre-local-prometheus-opera-prometheus-0       3/3     Running   1          28h

```