---
title: "KRT YAML specs"
description: >
  YAML Definition of a KRT file.
weight: 20
---

A KRT yaml file is a declarative file describing the content of the KRT.
It has general description of the Version, a GRPC entrypoint for the Runtime Version and
nodes that are connected between each other to form workflows that will be access through GRPC services defined on the entrypoint.  

## Fields Specs

Here is a description of each field divided in five main concepts that make a KRT file:

### Metadata

Descriptive information of the Runtime Version.

- **version**: the unique ID of this version.
- **description**: brief text describing the content or functionality of this version.

### Entrypoint

This section describes how to access the Runtime Version.

- **entrypoint**:
  - **proto**: protobuffer file describing messages and services contained in this Runtime Version.
  - **image**: base image and tag used to run the entrypoint service. It's provided by Konstellation registry.

### Configuration

This section describes all variables and files that are environment related and would be configured
once the Runtime Version is uploaded to a KAI Server instance. This includes sensible data that should not
be included on the KRT file, for example passwords.

All configuration defined here is mandatory, the Runtime Version won't run if any of them is undefined.

- **config**:
  - **variables**: list of variable names that are going to be defined in KAI Server as environment variables.
  - **files**: list of file names.

### Nodes

This section includes a list of all existing components in the KRT file. Each component of a Runtime Version is a node.
To define each node in the krt file, you need to specify a name, a base image and its main source file.

- **nodes** (a list of):
  - **name**: an identifier text, must be unique in the node list.
  - **image**: a base image provided by Konstellation used to run this node.
  - **src**: a path relative to the root of the KRT file pointing to the source file used to run this component.
  - **gpu**(optional): defaults to false, if true the node will use gpu, only available for Nvidia GPU.

### Workflows

This section includes the list of workflows contained in the KRT file. Each workflow connects one or more nodes between
each other and with a service defined in the entrypoint proto file.

- **workflows** (a list of):
  - **name**: an identifier text, must be unique in the workflow list.
  - **entrypoint**: the name of a serviced defined in the proto buffer file of the entrypoint. See `entrypoint.proto` above.
  - **sequential**: a list of node names that are connected sequentially as part of this workflow. All names should
   exist on the node list defined above.
