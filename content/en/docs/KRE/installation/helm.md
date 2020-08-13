---
title: "Helm basics"
linkTitle: "Helm basics"
description: >
  Basic steps to deploy KRE with Helm on an already running Kubernetes cluster.
weight: 5
---

KRE can be installed on top of a Kubernetes cluster using the [Helm](https://helm.sh/) package manager.

## Prerequisites

- Helm v3 or later
- Kubernetes v1.15+

## Install the chart

1. Add the Konstellation Helm repository:

```bash
helm repo add konstellation-io https://charts.konstellation.io
```

2. Optionally, create a namespace to deploy all KRE components or skip this step using a created one:

```bash
kubectl create namespace kre
```

3. Run the following command, providing a name for your KRE release (in this case `kre`) and specifying the namespace:

```bash
helm upgrade --install kre --namespace kre konstellation-io/kre
```

## Uninstall the chart

To uninstall the `kre` deployment, use the following command:

```bash
helm uninstall kre --namespace kre
```

This command removes all the Kubernetes components associated with the chart and deletes the release. It only removes main KRE components, you may need to remove other resources created by KRE. 
