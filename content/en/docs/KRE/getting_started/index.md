---
title: "Getting started"
description: A quick guide to start working with KRE.
weight: 5
---

## Pre requisites

In order to work with KRE you will need these requisites. This is not completely mandatory but if you read all docs before starting, it would help to understand better what you are doing on each step.

- [Basic helm knowledge]({{< relref "docs/KRE/installation/helm" >}})
- [KRE basic concepts]({{< relref "docs/KRE/concepts" >}})


## Installation

To start working with KRE you first need to get it installed in some Kubernetes cluster. You can choose different installation targets like [local Minikube]({{< relref "docs/KRE/installation/minikube" >}}), [EKS]({{< relref "docs/KRE/installation/EKS" >}}) or [GKE]({{< relref "docs/KRE/installation/GKE" >}}). 


## Logging into KRE

Once you have KRE installed you can access the Admin UI with the user email you set during installation and then follow the detailed [login guide]({{< relref "docs/KRE/tasks/login" >}}).



## Create your first Version

You need to follow these steps in order to create a version and get it ready to be called from outside KRE.


### 1. Create Runtime

Once you are logged in as an administrator, you can create a new Runtime by simply following the [create Runtime guide]({{< relref "docs/KRE/tasks/create_runtime" >}}).


### 2. Create a KRT file

Inside a Runtime you can upload one or more versions that can later be ran and publish. You can use a pre-generated [sample greeter krt](/website/krts/greeter-v1.krt), or learn [how to create your own KRT file]({{< relref "docs/KRT/tasks" >}}).


### 3. Upload the KRT file

Next thing you need is to upload the KRT that contains your Version. Within an existing Runtime detail page you can [upload a new version]({{< relref "docs/KRE/tasks/create_version" >}}).


### 4. Start and Publish

An uploaded Version is created in "STOPPED" state, in order to use it you need to first "START" it and "PUBLISH" it before been able to call it. You can learn more about it on the [Version lifecycle guide]({{< relref "docs/kre/tasks/version_management#version-lifecycle" >}}) 


### 5. Make a call

Now you have all set to consume your Version services. You can [test it with a simple CLI]({{< relref "docs/KRE/tasks/consume_version_services#call-from-cli" >}}), or you can [create a complete gRPC client]({{< relref "docs/KRE/tasks/consume_version_services#call-from-grpc-client-on-golang" >}}) to call it. 

