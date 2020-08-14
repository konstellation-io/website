---
title: "GKE"
linkTitle: "GKE"
description: >
  Required configuration to installing KRE on GKE.
weight: 20
---

# GKE deployment

The flavor of Kubernetes on Google is called GKE (Google Kubernetes Engine) which allow to deploy a cluster managed by Google. This means that Goolge will manage the lifecyle of the Master nodes of our cluster. 

When you create a GKE cluster by default is created an Instance Group, which is resposible to start the GCE (Goolge Compute Engine) instances and scale up and down depending of your configuration. In these instances of GCE is where the loads that we deploy on GKE will run. Therefore is needed to ajust the type of instances on this instance group, to support the load of your deployment.  For a PoC of KRE you can setup an Instance Group with min instances 1 up to 3 of type `n1-standard-2` and everything will works fine. Be aware about the autoscaling configuration of the Instances Group, because if you only have one Instance Group added to your GKE cluster, at least is required one instance up in order to run some Kubernetes componentes that run as PODS.

Deploy an GKE cluster is not the goal of this guide, only the detail some specific configuration needed to run KRE on top of it. It is recommend to use IaC (Infrastructure As Code) approach using Terraform to automate the creation of your cluster, [here](https://learn.hashicorp.com/tutorials/terraform/gke) you can find usefull resources about that. Also you can follow the instructions from the official [Google site](https://cloud.google.com/cloud-build/docs/deploying-builds/deploy-gke). 

# Storage

# Kubernetes required components

## Ingress controller

## Cert manager

## Storage provisioner

# DNS

## Get Ingress Controller hostname

First of all you need to konw the name of the ELB where we have to point our DNS entry. With the below command you will get name of this Load Balancer.

```bash
kubectl -n kube-system get svc -l app=nginx-ingress,component=controller -o jsonpath="{.items[0].status.loadBalancer.ingress[0].hostname}"
```

The output of this command should be something like `-`.

## Create wildcard entry in your hosted zone


 
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

## Create values.yaml

## Install Helm chart

## Validate the installation
