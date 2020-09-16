---
title: "Local"
linkTitle: "Local"
description: >
  How to install KRE on your local machine for testing or development purpose.
weight: 10
---

# Requirements

The recomended way to test KRE is deploy it on top of Minikube with VirtualBox driver. Deploying this way you can check a full featured installation of KRE.

Install the below required software following the steps in their respective sites, depend of your OS you will need one method or other described on each one.

[Docker](https://docs.docker.com/engine/install/)

[kubectl](https://kubernetes.io/es/docs/tasks/tools/install-kubectl/)

[Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/) 

[VirtualBox](https://www.virtualbox.org/wiki/Downloads)

[Helm](https://helm.sh/docs/intro/install/)



## Installation with krectl script

To hide all the complexity of installing and setting a local cluster, we've created a script called `krectl.sh` in the [project repository](https://github.com/konstellation-io/kre) that do all the needed steps.

This script is a development tool used for several tasks. From an end user point of view, you can run this command:

```bash
./krectl.sh dev --skip-build 
```

NOTE: the `--skip-build` option means that the script won't run a docker build for each component's image. This is the default 
behaviour but is only needed if you are going to develop any KRE component. 

After a couple of minutes you will have a running Minikube profile with a KRE installed and running. The script will also try an automatic login into the local admin section `http://admin.kre.local` (see #validate-the-installation). 


## Edit your hosts file

Get the cluster ip with the command:
```bash
minikube -p kre-local ip
# Output example
# 192.168.99.100
```

Add this lines to `/etc/hosts`
```
192.168.99.100 admin.kre.local
192.168.99.100 api.kre.local
```


## Validate the installation

If every run correctly and you edited your hosts file, you should be able to login into KRE admin section. In a local environment there are no outgoing emails, but you can login with this command:

```bash
./krectl.sh login 
```
