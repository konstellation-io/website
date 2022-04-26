---
title: "Core concepts"
linkTitle: "Core concepts"
weight: 10
description: >
  Description of core concepts to understand how KAI Server works
---


KAI Server has the following core concepts:

- [Engine](#engine)
- [Projects](#projects-aka-runtimes)
- [Versions](#versions)
- [Workflows](#workflows)
- [Nodes](#nodes)

### Engine

The Engine component is the central component that works as the **operation tool** to create, manage and monitor all the resources associated with each solution that you would put in production. It includes an admin web app, an API and some other components.

Using the admin web app you will be able to manage users, permissions and runtimes.

### Projects (a.k.a. Runtimes)

Inside KAI Server you install Projects. A project is identified by a name and a description, you can upload as many versions for it as you need.

A project represents a single application, with different versions and all the tools and resources related to it, such as monitoring, audits, users, workflows, data, etc.

### Versions

A version is a collection of all things needed for your solution to work, like code for each node, models, assets... This is a key concept in KAI Server because if you make changes at any level, model or code, you then have a new version that must be uploaded and deployed. This make versions immutable entities easier to track and debug over time. To upload your versions into a project you will need a KRT file.

The initial state of a version is stopped. A stopped version doesn't consume any resource in the cluster. When you start a version, all needed resources are created, but it is not accessible from the outside. You have to publish a version if you want to call it from the outside.

### Workflows

A Workflow is a sequence of tasks that process an incoming message from the external world and return a response. Each task is designed as a node that takes an input message and generates an output. You can add as many nodes as you need to your workflow.

A workflow is composed by an entrypoint and as many nodes as you need to fulfill the desired solution. The workflow defines how the nodes are interconnected and how the information will flow from one node to another.

### Nodes

A node is an isolated process inside a workflow. They can be coded in `Python` or in `GoLang` and are designed to be scalable and asynchronous. Each node is deployed in a separated pod inside the cluster and they are connected to other nodes through an event broker.
