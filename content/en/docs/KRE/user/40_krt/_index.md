---
title: "KRT"
linkTitle: "KRT"
weight: 40
description: >
  Manage KRT to generate new versions for a project
---

- [What is a KRT?](#what-is-a-krt)
- [Entities](#entities)
- [KRT YAML File](#krt-yaml-file)
- [Entrypoint](#entrypoint)
- [Nodes](#nodes)
- [Workflows](#workflows)
- [Runners](#runners)
- [Structure of a KRT](#structure-of-a-krt)

## What is KRT?

It stands for **Konstellation Runtime Transport**. Is the file format used in Konstellation as an easy way to move between Development (KAI Lab) to Production (KAI Server) environments. 

A KRT file is a **single and self-contained file** with everything needed for a version to be deployed on the server. It is compressed in `.tar.gz` format and renamed to `.krt` extension.  
 
A KRT file defines and pack a complete Version, including: 

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

It is a declarative file describing the content of the KRT. It has general description of the Runtime Version, a gRPC entrypoint for the Version and a list of nodes that are connected between each other in different workflows. 
 
The yaml file connects all these entities together, Entrypoint, Workflow, Nodes and Runners in order to define a Version and the way it works.

Learn about the fields in the [KRT YAML specs]({{< relref "docs/KRE/user/40_krt/specs" >}}). 

```yaml
version: price-estimator-v3
description: New Room Price predictions.

entrypoint:
  proto: public_input.proto
  image: konstellation/kre-entrypoint:1.6.0

config:
  variables:
    - OUTPUT_PRICE_CURRENCY

nodes:
  - name: etl
    image: konstellation/kre-py:1.23.1
    src: src/etl/main.py
  - name: model
    image: konstellation/kre-py:1.23.1
    src: src/model/main.py
  - name: output
    image: konstellation/kre-py:1.23.1
    src: src/output/main.py
  - name: save-metric
    image: konstellation/kre-py:1.23.1
    src: src/save-prediction-metric/main.py

workflows:
  - name: ny-room-price
    entrypoint: MakePrediction
    sequential:
      - etl
      - model
      - output
  - name: save-metric
    entrypoint: SavePredictionMetric
    sequential:
      - save-metric
```

## Entrypoint

```yaml
entrypoint:
  proto: public_input.proto
  image: konstellation/kre-entrypoint:1.6.0
```

A Version defines its communication with the outside world through its entrypoint. This is a gRPC server 
defined in proto buffer format containing messages and services to interact with workflows inside the Version.

The entrypoint is defined via `proto` file. This file defines the data contract for request and responses and the services that are going to be deployed within the entrypoint.

```proto
syntax = "proto3";

package entrypoint;
option go_package = "main";

message Request { ... }

message Response { ... }

message SaveMetricRequest { ... }

message SaveMetricResponse { ... }

service Entrypoint {
  rpc MakePrediction(Request) returns (Response) {};
  rpc SavePredictionMetric(SaveMetricRequest) returns (SaveMetricResponse) {};
};
```

## Nodes

```yaml
nodes:
  - name: etl
    image: konstellation/kre-py:1.23.1
    src: src/etl/main.py
  - name: model
    image: konstellation/kre-py:1.23.1
    src: src/model/main.py
  - name: output
    image: konstellation/kre-py:1.23.1
    src: src/output/main.py
  - name: save-metric
    image: konstellation/kre-py:1.23.1
    src: src/save-prediction-metric/main.py
```

A Node is a task inside a workflow. It has two main parts, a runner image, and the source code files specific to perform its task. 
   
Nodes have a single responsibility consisting on receive an input message, perform a task and return an output message. This is achieved with handler functions defined in source code files specified in the KRT file. 

Once the node is running it will look for two handler functions, one at starting time for initialization, and a second one to process incoming messages, these functions are called init handler and message handler respectively. Init handler is optional and will be executed only once upon node starting to run. Message handler is mandatory and will be executed each time the node receives a message. 


## Workflows

A Workflow is a sequence of tasks used to process incoming messages from the outside world and returns a response. Each task is a node of a graph that takes an input message and generates an output for the next node. The last node's output is used as response to the incoming message. 

Each workflow can have one or more nodes, depending on how many steps are needed to process a given input message.

A workflow is connected to the outside world through a service defined in KRT entrypoint. This entrypoint is a gRPC server that handle incoming calls from third parties gRPC clients and deliver them as input messages to the first node in the workflow and wait for the last node output to send a response to it corresponding gRPC client call.

Once a Version is started, all nodes from all workflows are created as a POD in kubernetes.

## Runners

Runners are base docker images, provided by Konstellation, that are used for nodes to run code on different 
programming languages. Each image includes a specific framework tha provides utilities to the users and is responsible of all the piping that occurs inside a node.

## Structure of a KRT

```bash
project
└───docs   
|   README.md # Documentation shown inside KAI Server
│
└───models    # If the process needs a model to do the inference
│   │   model.joblib
│   │   encoder.joblib
│   
└───src       # Source code of each node
│   └───etl
│       │   main.py
│   └───model
│       │   main.py
│   └───output
│       │   main.py
|
|   krt.yml   # krt manifest
|   public_input.proto    # proto defining entrypoint Services and Messages.
|   internal_nodes.proto  # proto defining Messages that interconnect nodes.
```