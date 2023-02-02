---
title: "KRT V2 YAML specs"
description: >
  YAML Definition of a KRT file.
weight: 20
---

A KRT yaml file is a declarative file describing the content of the KRT.
It has general description of the Version, a GRPC entrypoint for the Runtime Version and
nodes that are connected between each other to form workflows that will be access through GRPC
services defined on the entrypoint.

Example below is directly taken from our krt v2 demo repo:

```yaml
version: classificator-v1
krtVersion: v2 # new tagged krt version
description: Demo email classificator for branching features.
entrypoint:
  proto: public_input.proto
  image: konstellation/kre-entrypoint:latest
config:
  variables:
    - EARLY_REPLY_ENABLED
    - WEBHOOK_URL
workflows:
  - name: classificator
    entrypoint: Classificator
    exitpoint: exitpoint-node # name must match node's
    nodes:
      - name: etl
        image: konstellation/kre-py:latest
        src: src/etl/main.py
        gpu: false
        replicas: 1
        subscriptions:
          - "entrypoint"

      - name: email-classificator
        image: konstellation/kre-py:latest
        src: src/email_classificator/main.py
        gpu: false
        replicas: 1
        subscriptions:
          - "etl"

      - name: repairs-handler
        image: konstellation/kre-go:latest
        src: bin/repairs_handler
        gpu: false
        replicas: 2
        subscriptions:
          - "email-classificator.repairs" #consume from a subtopic

      - name: stats-storer
        image: konstellation/kre-go:latest
        src: bin/stats_storer
        gpu: false
        replicas: 1
        subscriptions:
          - "email-classificator"

      - name: exitpoint-node
        image: konstellation/kre-go:latest
        src: bin/exitpoint
        gpu: false
        replicas: 1
        subscriptions:
          - "etl"
          - "stats-storer"
```

## Fields Specs

Here is a description of each field divided in five main concepts that make a KRT file:

### Metadata

Descriptive information of the Runtime Version.

- **version**: the unique ID of this version. Must be in lowercase and can only contain letters, numbers and "-".
- **krtVersion**: indicates the krt version this file is using. Only values allowed are "v1" and "v2".
- **description**: brief text describing the content or functionality of this version.

### Configuration

This section describes all variables and files that are environment related and would be configured
once the Runtime Version is uploaded to a KAI Server instance. This includes sensible data that should not
be included on the KRT file, for example passwords.

All configuration defined here is mandatory, the Runtime Version won't run if any of them is undefined.

- **config**:
  - **variables**: list of variable names that are going to be defined in KAI Server as environment variables.
    Must be in uppercase and can only contain letters, numbers and "_".
  - **files**: list of file names.

### Entrypoint

This section describes how to access the Runtime Version.

- **entrypoint**:
  - **proto**: protobuffer file describing messages and services contained in this Runtime Version.
  - **image**: base image and tag used to run the entrypoint service. It's provided by
    [Konstellation registry](https://hub.docker.com/u/konstellation).

### Workflows

This section includes the list of workflows contained in the KRT file. Each workflow has one or more
nodes defined inside, of which one of them must be assigned to be the exitpoint.

- **workflows** (a list of):
  - **name**: an identifier text, must be unique in the workflow list.
  - **entrypoint**: the name of a serviced defined in the proto buffer file of the entrypoint.
    See `entrypoint.proto` above.
  - **exitpoint**: the name of the node assigned to be the exitpoint.
  - **nodes** (a list of): a list of node specifications that form the workflow.

#### Nodes

To define each node inside the workflows, you need to specify a name, a base image, its main source
file and its subscriptions.

- **nodes**:
  - **name**: an identifier text, must be unique in the node list.
  - **image**: a base image provided by Konstellation used to run this node.
  - **src**: a path relative to the root of the KRT file pointing to the source file used to run
    this component, for the case of a python node. For a Golang node this field will point towards the generated bin file.
  - **gpu**(optional): defaults to false, if true the node will use gpu, only available for Nvidia GPU.
  - **replicas**(optional): defaults to 1, is the number of times this node will be replicated in the cluster.
  - **subscriptions** (a list of): a list of other node names to which this node will subscribe to.
    It is also possible to filter down other node's outputs by adding a dot followed by a subtopic to a node's name. Example:

```yml
    - name: repairs-handler
      image: konstellation/kre-go:latest
      src: bin/repairs_handler
      gpu: false
      replicas: 2
      subscriptions:
         - "email-classificator.repairs"
         - "other-subscription"
         - "other-subscription.subtopic"
```
