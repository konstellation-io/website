---
title: "Versions in KRT V1"
weight: 10
description: >
  How versions are hanlded in KRT V1
---

- [Version Components](#version-components)
  - [Entrypoint](#entrypoint)
  - [Node](#node)
  - [Workflow](#workflow)

## Version Components

Each version is composed by at least one `entrypoint`, one `node` and one `workflow`.

### Entrypoint

This is the starting point of the application, and it is the node that receives requests from
external actors and provides responses to them.
The entrypoint is created by the `KAI Server` and the users do not have control over it.
This is done automatically by the underlying `KAI Server Runner SDK`.

An `entrypoint` has a kubernetes `Service` attached to it, so it can be reached and queried.
The `Service` attached to the `Entrypoint` has the following spec:

```yml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: "2022-03-28T13:23:40Z"
  labels:
    type: entrypoint
    version-name: #VERSION_NAME#-entrypoint
  name: #SERVICE_NAME#
  namespace: kre
  resourceVersion: "11275" 
  uid: 747e7d01-2278-4ee7-8fc7-d158293c1697 
spec:
  clusterIP: 10.108.63.40
  ports:
  - name: grpc
    port: 9000
    protocol: TCP
    targetPort: 9000
  - name: web
    port: 80
    protocol: TCP
    targetPort: 8080
  selector:
    type: entrypoint
    version-name: #VERSION_NAME#-entrypoint
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}   
```

Where:

- `#VERSION_NAME#`: The version name defined in the `krt.yml` manifest (e.g. `v1`).
- `#SERVICE_NAME#`: Depending on the version status:
  - `Started` version: is the same as the `#VERSION_NAME#`
  - `Published` version: `active-entrypoint`

### Node

A node is a process defined and programmed by the user. It can be coded in Python or GoLang,
and it uses the `KAI Server Runner SDK`.
A user can define one or more nodes inside a version.

Every node must receive data and return data. The data received/returned must be specified in a
`.proto` file and is defined by the user.

The main components of a node are:

- The data contract defined in `.proto` files. Specifies the input and output of a node.
- The base image for the node. `KAI Server` provides several flavors for the base image
  (both in Python and GoLang).
- The code that runs on top the base image. The code defines the behavior and responsibilities of
  the node. Nodes are coded on top of the `KAI Server Runner SDK`

Nodes are defined in the `krt.yml` manifest with the following structure:

```yml
- name: py-greeter
  image: konstellation/kre-py:1.23.0
  src: src/py-greeter/main.py
  gpu: false # gpu is an optional value, defaults to false.
```

A node is deployed within a `KAI Server Runner Image` that is responsible for executing the
code defined for the node.
In this way, KAI Server can provide utilities (such as measurements, logs, observability...)
that make coding a node focused solely on worrying about business logic.

{{< imgproc version_architecture Resize "800x" />}}

You can get more detailed info about nodes in [KRT V1 guide]({{< relref "docs/50_krt" >}})
and in the [KAI Server Runner SDK for KRT V1 Guide]({{< relref "docs/60_kais_runner_sdk/10_sdk" >}})

### Workflow

A workflow is the definition of how the nodes are connected between them.
A user can define one or multiple workflows inside a version.

The main components of a workflow are:

- The workflow entrypoint which represents the gRPC service that is tied to the workflow.
  This is useful when you define more than one workflow.
- The flow shape that links the nodes. The only supported flow is `sequential`.
- An ordered node list that defines the sequential order for the nodes in order of execution.

```yml
- name: py-greeting
  entrypoint: PyGreet
  sequential:
    - etl
    - inference
    - output
```

{{< imgproc version_screenshot Resize "1000x" >}}
Published version with 2 workflows (ny-room-price and save-metrics) and a total of 4 nodes
(etl, model, output and save-metric).
{{< /imgproc >}}
