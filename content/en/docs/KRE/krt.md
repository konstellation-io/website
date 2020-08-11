---
title: "KRT"
linkTitle: "Konstellation Runtime Transport"
description: >
  The .krt files are compressed tarballs to generate new versions.
weight: 30
---

Konstellation Runtime Transport is a compressed file with the definition of a runtime version, included the code to
run and a YAML file called `kre.yml` with the desired workflows definitions.

The base structure of a `kre.yml` is as follows:

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
