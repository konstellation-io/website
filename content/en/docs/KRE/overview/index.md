---
title: "KRE Overview"
linkTitle: "Overview"
date: 2020-08-04
description: >
  How KRE helps you to deploy your ML solutions
weight: 10
---

Konstellation Runtime Engine is an application that allow to run AI/ML models for inference based on the content of a `.krt` file.

| Component            | Description |
| -------------------- | ----------- |
| Admin UI             |             |
| Admin API            |             |
| K8s Manager          |             |
| Runtime API          |             |
| K8s Runtime Operator |             |
| Runner Python        |             |
| Runner Go            |             |

## Architecture

KRE is designed based on a microservice pattern to be run on top of a Kubernetes cluster.

The following diagram describes the main components and the relationship each other:

{{< imgproc architecture Resize "1000x" >}}
KRE Architecture.
{{< /imgproc >}}
