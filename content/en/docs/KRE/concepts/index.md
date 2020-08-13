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

A version is a collection of all things needed for your AI solution to work like code, models, assets,... This is a key concept in KRE because if you make changes at any level, model or code, you then have a new version that must be uploaded. This make versions immutable entities easier to track and debug over time.

To upload your versions into a runtime you will need a KRT file. To learn how to create a [KRT file]({{< relref "docs/krt" >}}) for more information.

### Workflows
