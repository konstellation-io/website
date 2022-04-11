---
title: "Versions"
linkTitle: "Project Versions"
weight: 30
description: >
  Manage versions in a KAI Server project
---

- [Version Components](#version-components)
    - [Entrypoint](#entrypoint)
    - [Node](#node)
    - [Workflow](#workflow)
- [Version Lifecycle](#version-lifecycle)
- [Version management](#version-management)
    - [Creating new versions](#creating-new-versions)
    - [Starting a version](#starting-a-version)
    - [Stopping a version](#stopping-a-version)
    - [Publishing a version](#publishing-a-version)



{{< imgproc version_screenshot Resize "1000x" >}}
Published version with 2 workflows (ny-room-price and save-metrics) and a total of 4 nodes (etl, model, output and save-metric).
{{< /imgproc >}}

## Version Components

Each version is composed by at least one `entrypoint`, one `node` and one `workflow`.

### Entrypoint

This is the starting point of the application and is the node that receive requests from external actors and provide responses to them. The entrypoint is created by the `KAI Server` and the users do not have control over it. This is done automatically by the underlying `KAI Server Runner SDK`.

An `entrypoint` has a kubernetes `Service` attached to it, so it can be reached and queried. The `Service` attached to the `Entrypoint` has the following spec:

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

- `#VERSION_NAME#`: The version name defined in the `krt.yml` manifest (eg. `v1`).
- `#SERVICE_NAME#`: Depending on the version status:
    - `Started` version: is the same as the `#VERSION_NAME#`
    - `Published` version: `active-entrypoint`

### Node

A node is a process defined and programmed by the user. It can be coded in python or goLang and it uses the `KAI Server Runner SDK`. A user can define one or more nodes inside a version.

Every node must receive data and return data. The data received/returned must be specified in a `.proto` file and is defined by the user.

The main components of a node are:

- The data contract defined in `.proto` files. Specifies the input and output of a node.
- The base image for the node. `KAI Server` provides several flavors for the base image (both in python and goLang).
- The code that runs on top the base image. The code defines the behaviour and responsibilities of the node. Nodes are coded on top of the `KAI Server Runner SDK`

Nodes are defined in the `krt.yml` manifest with the following structure:

```yml
- name: py-greeter
  image: konstellation/kre-py:1.23.0
  src: src/py-greeter/main.py
  gpu: false # gpu is an optional value, defaults to false.
```

A node is deployed within a `KAI Server Runner Image` that is responsible for executing the code defined for the node. In this way, KAI Server can provide utilities (such as measurements, logs, observability...) that make coding a node focused solely on worrying about business logic.

{{< imgproc version_architecture Resize "800x" />}}

You can get more detailed info about nodes in [KRT guide]({{< relref "docs/KRE/user/40_krt" >}}) and in the [KAI Server Runner SDK Guide]({{< relref "docs/KRE/user/50_kais_runner_sdk" >}})

### Workflow

A workflow is the definition of how the nodes are connected between them. A user can define one or multiple workflows inside a version.

The main components of a workflow are:

- The workflow entrypoint which represent the gRPC service that is tied to the workflow. This is usefull when you define more than one workflow.
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

## Version Lifecycle

In the following graph you can see all possible statuses and actions for a runtime version. The blue boxes are actions, and the green ones are statuses:

{{< imgproc version_lifecycle Resize "1000x" >}}

{{< /imgproc >}}

## Version management

In the version details screen we can manage the project version. There are four actions that we can perform using the left-bottom buttons: start, stop, publish and un-publish. When we perform any of these actions the version status changes. The following table shows all possible version status:

| Status      | Description                                                                                                                                  |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| `stopping`  | Indicates the version is deleting their associated k8s resources.                                                                            |
| `stopped`   | The version is created in KRE but it is not consuming any resource because all components are not created at k8s.                            |
| `starting`  | The version is creating their associated k8s resources.                                                                                      |
| `started`   | The version entrypoint and nodes are running and ready in k8s. The entrypoint service is not not associated with the ingress so you cannot call it from outside. |
| `published` | The entrypoint is accessible from outside and the incoming request are routed to it.                                                         |

### Creating new versions

To create new versions for the project you need to prepare and upload a `KRT` file. You can find detailed info about `KRT` files in the [KRT guide]({{< relref "docs/KRE/user/40_krt" >}}).

### Starting a version

Starting new versions is the action to start all the components for that version (defined in the `krt.yml` manifest) in the underlying kubernetes cluster. A started version is accesible from within the cluster but not from the outside.

{{< imgproc version_started_diagram Resize "400x" />}}


### Stopping a version

Stopping a version will remove all resources associated to that version from the cluster (defined in the `krt.yml` manifest).

### Publishing a version

Publishing a version will change the `Service` name attached to the version's entrypoint. The `Service` will be renamed to `active-entrypoint` so that the cluster ingress is linked to it. This means that the published version will have its entrypoint linked to a cluster ingress making it reachable from the outside.

Only one version can be public at a time, and if you try to publish a version while another version is published it will result in a change in the published versions.
