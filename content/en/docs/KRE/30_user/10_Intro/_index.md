---
title: "Intro"
linkTitle: "Intro"
weight: 10
description: >
  Architecture of versions
---


When dealing with solutions at a user's level our scope changes and narrows down to another set of issues.  
Our solutions will be coded into nodes, these nodes alongside an entrypoint will form a workflow,
one or more workflows will form a version of a project.
Projects can be versioned and several can be uploaded at the same time to your KAI Server.

Projects will then be run by a runner inside KAI Server, it looks something like this:

## Versions architecture

{{< imgproc versions_architecture Resize "1000x" />}}

NATS acts as an event broker, when building our projects we must provide the order and ID of the nodes
compromising our workflow, so remember, nodes will be executed in a specific order.  
Also, an entrypoint must be declared, so we can call our project. So, for the moment being we can
make some calls to a started version from within the cluster.  
But what happens when we want to call a version from outside the cluster? We must then publish a version.

{{< imgproc versions_architecture_published Resize "1000x" />}}

A published version will expose an entrypoint through GRPC and an ingress, it will be hosted in an
IP address. So external users can make use of this endpoint to send requests.  
Only one version can be published at a time in a KAI Server cluster.
