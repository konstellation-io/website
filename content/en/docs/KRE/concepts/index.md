---
title: "Concepts"
linkTitle: "Concepts"
description: >
  Get introduced to basic concepts of KRE.
weight: 10
---

## What is KRE?

It is an engine capable of deploy, manage and monitor a full AI solution in production in seconds using [KRT files]({{< relref "docs/krt" >}}).

KRE has the following main concepts:

- [Engine](#engine)
- [Runtimes](#runtimes)
- [Versions](#versions)
- [Workflows](#workflow)

### Engine

Engine component is the central component that works as the **operation tool** to create, manage and monitor all the resources associated with each solution that you would put in production. It includes an admin web app, an API and some other components.

{{< imgproc kre_overview Resize "1000x" >}}
Engine components are deployed in a specific k8s namespace. The runtimes are deployed in other namespaces.
{{< /imgproc >}}

Using the admin web app you will able to manage users, permissions and runtimes. You can see the [tasks]({{< relref "docs/KRE/tasks" >}}) if you want more information about what things you can do.

### Runtimes

Inside a KRE you can create as many runtimes as you need. A runtime is identified by a name and a description and it is where you can upload one or more solutions that we call versions. For example, you can deploy a KRE for the ClientX and create a runtime for each ClientX project. Inside each ClientX project you will upload the versions of your AI solution.

Going into greater detail, a runtime is an environment that is **isolated** from all other runtimes and other resources. Inside a runtime we will find the resources associated to your solution versions, a messaging system used to dispatch messages between components, databases to store metrics or generic data, a s3 store and internal components like an API to process incoming messages received from the KRE admin.

{{< imgproc runtime_overview Resize "700x" >}}
All versions share databases, storage and message system but it is isolated from other runtimes.
{{< /imgproc >}}

### Versions

A version is a collection of all things needed for your AI solution to work like code, models, assets,... This is a key concept in KRE because if you make changes at any level, model or code, you then have a new version that must be uploaded. This make versions immutable entities easier to track and debug over time. To upload your versions into a runtime you will need a KRT file. Learn more about how to create a KRT file [here]({{< relref "docs/krt" >}}).

The initial state of a version is stopped. A stopped version doesn't consume any resource in the cluster. When you start a version, all needed resources are created but it is not accessible from the outside. You have to publish a version if you want to call it from the outside.

Usually, your AI solution must perform multiple actions, for example: to use a model to get a prediction or to receive information to calculate some metrics,... so a version can define multiple workflows to accomplish that actions.

{{< imgproc version_overview Resize "800x" >}}

{{< /imgproc >}}

### Workflows

A Workflow is a sequence of tasks that processes an incoming message from the external world and returns a response. Each task is a node of a graph that takes an input message and generates an output. You can add as many nodes as you need to your workflow. The following image shows a basic workflow that makes a prediction using three nodes:

{{< imgproc basic_workflow_example Resize "900x" >}}
Example of a basic workflow
{{< /imgproc >}}

In KRE the external messages comes from a gRPC client. A special component called Entrypoint (gRPC server) dispatch the messages to a determinated workflow and returns the response of the last node to the client. The workflow nodes becomes a PODs in your k8s cluster when the version is started.
