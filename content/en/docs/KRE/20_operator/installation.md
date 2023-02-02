---
title: "Install KAI Server"
linkTitle: "Install KAI Server"
weight: 40
description: >
  Install KAI Server into Kubernetes
---

## Prerequisites

* Kubernetes 1.19+
* Nginx ingress controller. See [Ingress Controller](#ingress-controller).
* Helm 3+

## Install chart

```bash
$> helm repo add konstellation-io https://charts.konstellation.io
$> helm repo update
$> helm install [RELEASE_NAME] konstellation-io/kre
```

*See [helm repo](https://helm.sh/docs/helm/helm_repo/) and [helm install](https://helm.sh/docs/helm/helm_install/) for command documentation.*

## Dependencies

By default, this chart installs [InfluxDB](https://github.com/influxdata/helm-charts/tree/master/charts/influxdb) and [Kapacitor](https://github.com/influxdata/helm-charts/tree/master/charts/kapacitor) chart as dependencies.

However, **Kapacitor** is an optional dependency. To disable it during installation, set `kapacitor.enabled` to `false`.

## Uninstall chart

```bash
$> helm uninstall [RELEASE_NAME]
```

This removes all the Kubernetes components associated with the chart and deletes the release.

*See [helm uninstall](https://helm.sh/docs/helm/helm_uninstall/) for command documentation.*

## Upgrading Chart

```bash
$> helm upgrade [RELEASE_NAME] konstellation.io/kre
```

*See [helm upgrade](https://helm.sh/docs/helm/helm_upgrade/) for command documentation.*

## Ingress controller

This Chart has been developed using **Nginx Ingress Controller**. So using the default ingress annotations ensures its correct operation.

*See [values.yaml](https://github.com/konstellation-io/kre/blob/main/helm/kre/values.yaml) file and [Nginx Ingress controller](https://kubernetes.github.io/ingress-nginx/) for additional documentation**.

However, users could use any other ingress controller (for example, [Traefik](https://doc.traefik.io/traefik/providers/kubernetes-ingress/)). In that case, ingress configurations equivalent to the default ones must be provided.

Keep in mind that even using equivalent ingress configurations the correct operation of the appliance is not guaranteed.
