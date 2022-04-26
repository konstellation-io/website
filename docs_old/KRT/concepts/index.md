---
title: "Concepts"
linkTitle: "Concepts"
description: >
  A general description and purpose of KRT files.
weight: 10
---

## What is KRT?

It stands for **Konstellation Runtime Transport**. Is the file format used in Konstellation as an easy way to move
 between Development (KDL) to Production (KRE) environments. 

A KRT file is a **single and self-contained file** with everything needed for a solution to be deployed on KRE. Is compressed 
 in `.tar.gz` format and renamed to `.krt` extension, it contains definition and all components of Runtime Version.  
 
A KRT file defines and pack a complete Runtime Version, including: 

  - a definition YAML file named `krt.yml`.
  - source code of all components.
  - assets needed by each component.


## Entities

These entities are used to in a KRT file to define how Version's components interoperate:

- [KRT YAML file](#krt-yaml-file)
- [Entrypoint](#entrypoint)
- [Workflows](#workflows)
- [Nodes](#nodes)
- [Runners](#runners)



## KRT YAML file

It is a declarative file describing the content of the KRT. It has general description of the Runtime Version, 
 a gRPC entrypoint for the Version, a list of nodes that are connected between each other in different workflows. 
 
The yaml file connects all these entities together, Entrypoint, Workflow, Nodes and Runners in order to define a Version
and the way it works.

Learn about the fields in the [KRT YAML specs]({{< relref "docs/KRT/specs" >}}) and see a working 
[krt.yml example]({{< relref "docs/KRT/tasks/define_krt_yml" >}}) 


## Entrypoint

A Version defines its communication with the outside world through its entrypoint. This is a gRPC server 
defined in proto buffer format containing messages and services to interact with workflows inside the Version.

{{< imgproc entrypoint_example Resize "900x" >}}
Entrypoint example
{{< /imgproc >}}
 
Each service defined in the proto file is connected to a workflow in the `krt.yml` file definition. 


## Workflows

A Workflow is a sequence of tasks used to process incoming messages from the outside world and returns a response.
Each task is a node of a graph that takes an input message and generates an output for the next node. The last node's 
output is used as response to the incoming message. 

{{< imgproc basic_workflow_example Resize "900x" >}}
Example of a basic workflow that makes a prediction using three nodes
{{< /imgproc >}}

Each workflow can have one or more nodes, depending on how many steps are needed to process a given input message.

A workflow is connected to the outside world through a service defined in KRT entrypoint. This entrypoint is a gRPC 
server that handle incoming calls from third parties gRPC clients and deliver them as input messages to the first
node in the workflow and wait for the last node output to send a response to it corresponding gRPC client call.

Once a Version is started, all nodes from all workflow are created as a POD in kubernetes.
  

## Nodes

A Node is a task inside a workflow. It has two main parts, a runner image, and the source code files specific to perform
its task. 
   
Nodes have a single responsibility consisting on receive an input message, perform a task and return an output message.
This is achieved with handler functions defined in source code files specified in the KRT file. 

Once the node is running it will look for two handler functions, one at starting time for initialization, and a second
one to process incoming messages, these functions are called init handler and message handler respectively. Init handler
is optional and will be executed only once upon node starting to run. Message handler is mandatory and will
be executed each time the node receives a message. 


## Runners

Runners are base docker images, provided by Konstellation team, that can be used for nodes to run code on different 
programming languages. Each image includes language specific tools to integrate itself as part of a workflow. 
