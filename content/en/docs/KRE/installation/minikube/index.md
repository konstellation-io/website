---
title: "Minikube"
linkTitle: "Minikube"
description: >
  How install KRE on MiniKube.
weight: 20
---

# Minikube deployment

Minikube implements a local Kubernetes cluster on your OS. It runs the latest stable release of Kubernetes, with support for all standard Kubernetes features.



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
