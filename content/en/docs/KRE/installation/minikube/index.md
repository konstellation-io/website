---
title: "Minikube"
linkTitle: "Minikube"
description: >
  How install KRE on MiniKube.
weight: 20
---

# Minikube deployment
Minikube implements a local Kubernetes cluster on macOS, Linux, and Windows. Minikube runs the latest stable release of Kubernetes, with support for standard Kubernetes features.

## Installation with kre_ctl script
The local installation is complex, therefore there is a script called kre_ctl.sh at the root of our [repository](https://github.com/konstellation-io/kre) to help us throughout the process.
```bash
./krectl.sh dev --skip-build 
```

## Set /etc/hosts
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
To check if everything is working fine follow the [Validate]({{< relref "docs/KRE/installation/validate" >}}) section.
