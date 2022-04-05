---
title: "KAI Server Architecture"
linkTitle: "KAI Server Architecture"
weight: 30
description: >
  Components and architecture of a KAI Server
---

KAI Server runs inside a kubernetes cluster in a single namespace. All components for the KAI Server are deployed in the same namespace (`kre` by default).

## Components

### Admin API

The Admin API is the central component of KAI Server. It is reponsible of orchestrating all other components and to execute user actions in the cluster.

### Admin UI

This is the component that renders all the UI where the users can interact with the Runtimes.

### K8 Manager

This service exposes a gRPC service to encapsulate all Kubernetes related features and Prometheus queries to get metrics and alerts. The only service that is going to call this gRPC is the Admin API service when need to create new Kubernetes resources.

### MongoDB

This is the database where the Admin API stores all the objects needed to manage the server. Users, runtimes, versions, workflows and so on.

### NATs

NATs is the event broker that serves as the backbone of communications inside the Server. Nodes in Workflows makes use of NATs to communicate with each other.

### Monitorization

#### InfluxDB

#### Chronograf

## Helm Chart
