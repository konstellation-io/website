---
title: "krt.yml"
description: >
  YAML Definition of a KRT file.
weight: 30
---

This is a declarative file describing the content of the KRT. 

This file has general description of the Runtime Version, a GRPC entrypoint for the Runtime Version, nodes that are
 connected between each other to form workflows that will be access through GRPC services defined on the entrypoint.  


## Definition

Here is a description of each field divided in five main concepts that make a KRT file:
 
### Metadata

Descriptive information of the Runtime Version.

 - **version**: the unique ID of this version. 
 - **description**: brief text describing the content or functionality of this version. 
 
### Entrypoint

This section describes how to access the Runtime Version.

 - **entrypoint**:
   - **proto**: protobuffer file describing messages and services contained in this Runtime Version.
   - **image**: base image adn tag used to run the entrypoint service. It's provided by Konstellation registry.
  

### Configuration

This section describes all variables and files that are environment related and would be configured once the Runtime Version
is uploaded to a KRE instance. This includes sensible data that should not be included on the KRT file, for example passwords.

All configuration defined here is mandatory, the Runtime Version won't run if any of them is undefined. 

 - **config**:
   - **variables**: list of variable names that needs to be defined in KRE.
   - **files**: list of file names that needs to be filled in KRE.

### Nodes

This section includes a list of all existing components in the KRT file. Each component of a Runtime Version is a node.
To define each node in the krt file, you need to specify a name, a base image and its main source file. 

 - **nodes** (a list of):
   - **name**: an identifier text, must be unique in the node list.
   - **image**: a base image provided by Konstellation used to run this node. It's provided by Konstellation registry.
   - **src**: a path relative to the root of the KRT file pointing to the source file used to run this component. 
 

### Workflows

This section includes the list of workflows contained in the KRT file. Each workflow connects one or more nodes between 
each other and with a service defined in the entrypoint proto file. 

 - **workflows** (a list of):
   - **name**: an identifier text, must be unique in the workflow list.
   - **entrypoint**: the name of a serviced defined in the proto buffer file of the entrypoint. See `entrypoint.proto` above.
   - **sequential**: a list of node names that are connected sequentially as part of this workflow. All names should 
   exist on the node list defined above.


## Example file
 
This is a complete `krt.yml` example matching the folder structure shown [here]({{< relref "docs/krt/folder_structure.md#example-structure" >}})

```yaml
version: example-project
description: This is an example of a ML project.
entrypoint:
  proto: public_input.proto
  image: konstellation/kre-runtime-entrypoint:latest

config:
  variables:
    - API_KEY
    - API_SECRET
  files:
    - HTTPS_CERT

nodes:
  - name: etl
    image: konstellation/kre-py:latest
    src: src/etl/execute_etl.py

  - name: execute-dl-model
    image: konstellation/kre-py:latest
    src: src/execute_model/execute_model.py

  - name: create-output
    image: konstellation/kre-py:latest
    src: src/output/output.py

  - name: client-metrics
    image: konstellation/kre-go:latest
    src: bin/client-metrics

workflows:
  - name: prediction
    entrypoint: MakePrediction
    sequential:
      - etl
      - execute-dl-model
      - create-output
  - name: save-client-metrics
    entrypoint: SaveClientMetric
    sequential:
      - client-metrics
```
