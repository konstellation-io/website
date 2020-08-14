---
title: "Define a krt.yml file"
description: >
   A guide to write a KRT yaml file.
weight: 10
---

This is the example `krt.yml` file for the example project shown [here]({{< relref "docs/krt/tasks" >}}).

Here you can see how the two parts of the projects are divided in two workflows. The first workflow, called `prediction`,
has three nodes, the first input to a proper format (node `etl`), performing a prediction (node `execute-dl-model`), and
the last node convert the prediction to different format (node `create-output`).

The second workflow called `save-client-metrics` that collects metrics to match predictions with real data for later 
analysis. It only has one node called `client-metrics`.

This example Version has two environment variables called `API_KEY` and `API_SECRET` that would need to be setup once 
the version is uploaded to KRE. Same as with the file variable called `HTTPS_CERT`. 

Note that the Entrypoint `public_input.proto` should have two services defined called `MakePrediction` and 
`SaveClientMetric` to allow external call to be routed to each workflow correctly. 

Learn more about [how to create an entrypoint]({{< relref "docs/krt/tasks/create_an_entrypoint" >}}).


## Example krt.yml

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
