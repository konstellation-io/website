---
title: "Create an Entrypoint"
description: >
   A guide to create a gRPC entrypoint definition.
weight: 20
---

To connect workflows with third party clients you need a gRPC server called Entrypoint in your KRT file. 

But you don't need to implement any code, just define its messages and services.

In the [example project]({{< relref "docs/krt/tasks" >}}) there are defined two workflows that need to be exposed
through the entrypoint. So in the entrypoint proto file you need to create two services and messages as follows:


### Example public_input.proto

``` 
syntax = "proto3";

package entrypoint;

service Entrypoint {
  rpc Prediction(PredictionRequest) returns (PredictionResponse) {};
  rpc SaveMetrics(MetricsRequest) returns (MetricsResponse) {};
};

message PredictionRequest {
  string name = 1;
  ... add more fields here ...
}

message PredictionResponse {
  string prediction = 1;
  ... add more fields here ...
}
message MetricsRequest {
  ... add fields here ...
}

message MetricsResponse {
  ... add fields here ...
}

```