---
title: "Getting started"
description: A quick guide to start working with KRE.
weight: 5
---

## Installation

The best way to start working with KRE, to test it and understand how it works, is to deploy it on your local machine and follow basic instructions to create an example project.

Follow the instrcutions of install KRE on your [local environment]({{< relref "docs/KRE/installation/local" >}})


## Logging into KRE

As mention in the Install local environment section, to login to KRE just run the following script that you can find in the KRE repository.

```bash
./krectl.sh login 
```

## Create your first Version

You need to follow these steps in order to create a version and get it ready to be called from outside KRE.


### 1. Create Runtime

Once you are logged in as an administrator, you can create a new Runtime by simply following the [create Runtime guide]({{< relref "docs/KRE/tasks/create_runtime" >}}).


### 2. Download a sample KRT file

Inside a Runtime you can upload one or more versions that can later be ran and publish. You can use a pre-generated [sample greeter krt](/website/krts/greeter-v1.krt). You can find a more detailed explanation on how to create your own KRT file [here]({{< relref "docs/KRT/tasks" >}}).


### 3. Upload the KRT file

Next thing you need is to upload the KRT that contains your Version. Within an existing Runtime detail page you can [upload a new version]({{< relref "docs/KRE/tasks/create_version" >}}).


### 4. Start and Publish

An uploaded Version is created in "STOPPED" state, in order to use it you need to first "START" it and "PUBLISH" it before been able to call it. You can learn more about it on the [Version lifecycle guide]({{< relref "docs/kre/tasks/version_management#version-lifecycle" >}}) 


### 5. Make a call

Now you have all set to consume your Version services. You can [test it with a simple CLI]({{< relref "docs/KRE/tasks/consume_version_services#call-from-cli" >}}), or you can [create a complete gRPC client]({{< relref "docs/KRE/tasks/consume_version_services#call-from-grpc-client-on-golang" >}}) to call it. 
