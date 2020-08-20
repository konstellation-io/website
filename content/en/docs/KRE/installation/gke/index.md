---
title: "GKE"
linkTitle: "GKE"
description: >
  Required configuration to installing KRE on GKE.
weight: 20
---

# GKE deployment

The flavor of Kubernetes on Google is called GKE (Google Kubernetes Engine) which allow to deploy a cluster managed by Google. This means that Google will manage the lifecyle of the Master nodes of our cluster. 

When you create a GKE cluster by default is created an Instance Group, which is responsible to start the GCE (Google Compute Engine) instances and scale up and down depending of your configuration. In these instances of GCE is where the loads that we deploy on GKE will run. Therefore is needed to ajust the type of instances on this instance group, to support the load of your deployment.  For a PoC of KRE you can setup an Instance Group with min instances 1 up to 3 of type `n1-standard-2` and everything will works fine. Be aware about the autoscaling configuration of the Instances Group, because if you only have one Instance Group added to your GKE cluster, at least is required one instance up in order to run some Kubernetes componentes that run as PODS.

Deploy an GKE cluster is not the goal of this guide, only the detail some specific configuration needed to run KRE on top of it. It is recommend to use IaC (Infrastructure As Code) approach using Terraform to automate the creation of your cluster, [here](https://learn.hashicorp.com/tutorials/terraform/gke) you can find usefull resources about that. Also you can follow the instructions from the official [Google site](https://cloud.google.com/cloud-build/docs/deploying-builds/deploy-gke). 


# Storage

KRE uses a shared storage with `ReadWriteMany` volumes, for this reason is necessary to have a NFS storage(aka Filestore on Google) and get `filestore_ip` and `File share name` values to use later on.


# Kubernetes required components

KRE uses some additional components required for the tool, The basic configuration is documented below:

## Ingress controller

Ingress Controller is component create a load balancer on our cloud provider and links the load balancer with the ingress controller service, saving costs on extra network components, and allows to publish services outside our cluster.

```
helm upgrade --install \
     --namespace kube-system \
     --version 1.40.2 \
     nginx-ingress \
     stable/nginx-ingress
```

## Cert manager

Cert Manager automatize the creation and maintenance of certificates, allowing the use of TLS in the ingress controller. [+ Info]({{< relref "docs/KRE/installation/customization.md#cert-manager" >}})

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

In order to connect our [Filestore](#storage) with the cluster storage needs install a nfs-provisioner.

```bash
helm upgrade --install \
    --namespace kube-system \
    --set nfs.server=<FILESTORE_IP_ADDRESS> \
    --set nfs.path=/<FILESTORE_FILE_SHARE_NAME> \
    nfs-provisioner \
    stable/nfs-client-provisioner
```

Replace values `FILESTORE_IP_ADDRESS` and `FILESTORE_FILE_SHARE_NAME` with the `Filestore` you created earlier or with a pre-existing one.


# DNS


## Get Ingress Controller hostname

First of all you need to know the name of the ELB where we have to point our DNS entry. With the below command you will get name of this Load Balancer.

```bash
kubectl -n kube-system get svc -l app=nginx-ingress,component=controller -o jsonpath="{.items[0].status.loadBalancer.ingress[0].ip}"
```

The output of this command should show a `load balancer IP address`.

## Create wildcard entry in your DNS provider
Once obtained a `load balancer IP address` add in our DNS provider a new A Record, with the following values:

| Subdomain              | Load Balancer IP           |
| -----------------------| -------------------------- | 
| `*.kre.yourdomain.com` | `<load balancer IP address>` | 

After a while the resolution of your domain should point to the load balancer. KRE require of the following subdomains.

| Subdomain    | Description                                            |
| ------- | ----------------------------------------------------------- | 
| `admin.kre.yourdomain.com` | Web admin console  | 
| `api.kre.yourdomain.com` | API  |
 
If you `dns provider doesn't have wildcard entries` you can use an approach based on hosted zones.
 
## Validate 

To validate that the DNS configuration is working fine you can use the tool `dig` to query the DNS as follows.

```bash
dig admin.kre.yourdomain.com
```
The output should be something as below.

```
; <<>> DiG 9.16.1-Ubuntu <<>> admin.kre.yourdomain.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 901
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;admin.kre.yourdomain.com.	IN	A

;; ANSWER SECTION:
admin.kre.yourdomain.com.	1799 IN	A	1.2.3.4

;; Query time: 64 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
;; WHEN: mié ago 12 16:18:49 CEST 2020
;; MSG SIZE  rcvd: 75

```

# Helm deployment

## Create values.yaml


```yaml
config:
  smtp:
    enabled: true
    sender: "<YOUR_SMTP_EMAIL_ACCOUNT>" # If you use a smtp provider on cloud and your email is not verified, is possible that you can't login because the smtp authentication fail.
    senderName: "<SENDER_NAME>"
    user: "<SMTP_USER>"
    pass: "<SMTP_PASSWORD>"
    host: "<SMTP_HOST>"
    port: "<SMTP_PORT>"
  baseDomainName: kre."<YOUR_DOMAIN>"
  admin:
    apiBaseURL: api.kre."<YOUR_DOMAIN>"
    frontendBaseURL: https://admin.kre."<YOUR_DOMAIN>"
    userEmail: "<ADMIN_EMAIL_ADDRESS>" #Required, initial admin user, must match with the first user to log in.
  runtime:
    sharedStorageClass: nfs-client
    sharedStorageSize: 10Gi
    nats_streaming:
      storage:
        size: 2Gi
    mongodb:
      persistentVolume:
        enabled: true
        size: 5Gi
    chronograf:
      persistentVolume:
        enabled: true
        size: 1Gi
    influxdb:
      persistentVolume:
        enabled: true
        size: 5Gi
  auth:
    verificationCodeDurationInMinutes: 5
    jwtSignSecret: int_jwt_secret
    secureCookie: true
    cookieDomain: kre."<YOUR_DOMAIN>"

adminApi:
  tls:
    enabled: true
  host: api.kre."<YOUR_DOMAIN>"
  storage:
    class: nfs-client

adminUI:
  tls:
    enabled: true
  host: admin.kre."<YOUR_DOMAIN>"

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
    size: 6Gi

certManager:
  enabled: true
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: "<YOUR_EMAIL_ADDRESS>"
```

## Install Helm chart
```bash
kubectl create namespace kre
helm repo add konstellation-io https://charts.konstellation.io
helm repo update
helm upgrade --install kre --namespace kre --values values.yaml konstellation-io/kre
```

## Validate the installation
To check if everything is working fine follow the [Validate]({{< relref "docs/KRE/installation/validate" >}}) section.