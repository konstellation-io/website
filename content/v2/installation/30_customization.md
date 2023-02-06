---
title: "Customization"
linkTitle: "Customization"
weight: 30
description: >
  Customize KAI Server
---

## About customization

Helm chart parameters are configured via a `values.yaml` file or the `--set` option.

It is possible to configure a lot of aspects of a KAI Server deployment, as depending on your environment and your use case the default values could not be enough. If that is the case, one or more custom `values.yaml` files in conjunction with the `--set` option can be used with these parameters to be applied when running Helm.

## Chart Values

Check [here](https://github.com/konstellation-io/kre/blob/main/helm/kre/CHART.md) for a complete list of chart values.
