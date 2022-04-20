---
title: "KAI Server Training"
linkTitle: "KAI Server Training"
weight: 50
description: >
  KAI Server Training Project
---


The [Kai Server Training repo](https://github.com/konstellation-io/kai-server-training) is a training exercise designed to introduce new users to the usage and development of KRT projects.  
You can use the training exercise to build and deploy your first KRT project and also to reverse engineer it and learn how to develop your own projects.

## Description

This example consists of a workflow called _Descritpor_ that will print information of a given github repository, being this info its last github and dockerhub tag released.  
The workflow is made up of 3 different nodes, each one in charge of a different process through the pipeline:

- **ETL**: This node will accept incoming requests and prepare a new info struct to be filled.
- **Github**: This node will search for the Github information of the repo then fill the info struct with it if found.
- **Dockerhub**: This node will search for the Dockerhub information of the repo then fill the info struct with it if found.

All nodes also save metrics to the InfluxDB instance as an example of metric handling.

## How to deploy

You can find information about building and deployment within the repo's `readme` file.

## How to use

Once deployed the generated KRT file, you can open an entrypoint to your local pod. However to easy up usage, we've placed some helping scripts inside a folder called `scritps`, you can find information about usage within the `readme` file placed inside this directory as well.

## Descriptor example

{{< imgproc kai-training-example Resize "1000x" />}}

### Result example

{{< imgproc kai-training-result-example Resize "1000x" />}}
