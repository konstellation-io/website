---
title: "KAI Server Architecture"
linkTitle: "KAI Server Architecture"
weight: 20
description: >
  Components and architecture of a KAI Server
---

KAI Server runs inside a kubernetes cluster in a single namespace. All components for the KAI Server are deployed in the same namespace (`kre` by default).

## Components

{{< imgproc kais_architecture_img Resize "1200x" />}}

The schema above shows the architecture inside KAI Server. It is a scalable and asynchronous architecture designed to seamless deploy process automation projects into production.

### Admin API

The Admin API is the central component of KAI Server. It is responsible for orchestrating all other components and executing user actions in the cluster.

### Admin UI

This is the component that renders all the UI, where the users can interact with the Runtimes.

### K8 Manager

This service exposes a gRPC service to encapsulate all Kubernetes related features and Prometheus queries to get metrics and alerts. The only service that is going to call this gRPC is the Admin API service when it's needed to create new Kubernetes resources.

### MongoDB

This is the database where the Admin API stores all the objects needed to manage the server. Users, runtimes, versions, workflows and so on.

### NATs

NATs is the event broker that serves as the backbone of communications inside the Server. Nodes in Workflows make use of NATs to communicate with each other.

#### InfluxDB

KAI Server also stores metrics, for this it is used an InfluxDB instance. All nodes can write any kind of metric anytime desired. By giving a measurement name, tags, fields and data as usage of Influx requires.

#### Chronograf

Chronograf will display stored metrics in any form desired. For this _Flux_ queries are used within a given display format. These will display cells inside dashboards giving metrics meaning and context, valuable for analytics and supervision of projects.
