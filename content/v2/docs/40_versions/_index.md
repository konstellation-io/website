---
title: "Versions"
linkTitle: "Versions"
weight: 40
description: >
  Manage versions in a KAI Server runtime
---

- [Version Lifecycle](#version-lifecycle)
- [Version management](#version-management)
  - [Creating new versions](#creating-new-versions)
  - [Starting a version](#starting-a-version)
  - [Stopping a version](#stopping-a-version)
  - [Publishing a version](#publishing-a-version)

## Version Lifecycle

In the following graph you can see all possible statuses and actions for a runtime version.
The blue boxes are actions, and the green ones are statuses:

{{< imgproc version_lifecycle Resize "1000x" >}}

{{< /imgproc >}}

## Version management

In the version details screen we can manage the project's version.
There are four actions that we can perform using the left-bottom buttons: start, stop, publish and un-publish.
When we perform any of these actions the version status changes. The following table shows all possible version status:

| Status      | Description                                                                                                                                                   |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `stopping`  | Indicates the version is deleting their associated k8s resources.                                                                                             |
| `stopped`   | The version is created in KAI Server, but it is not consuming any resource because all components are not created at k8s.                                     |
| `starting`  | The version is creating their associated k8s resources.                                                                                                       |
| `started`   | The version entrypoint and nodes are running and ready in k8s. The entrypoint service is not associated with the ingress, so you cannot call it from outside. |
| `published` | The entrypoint is accessible from outside and the incoming requests are routed to it.                                                                         |

### Creating new versions

To create new versions for the project you need to prepare and upload a `KRT` file, the version name
should be unique in the runtime. This can be done by selecting the button _"Add version"_
displayed in the runtime's page.

{{< imgproc create_version Resize "1000x" >}}
Page of a runtime called "Demo" that already contains a runtime previously uploaded called "classificator-v1"
{{< /imgproc >}}

As _krt_ files depict versions, these will have different components in
[KRT v1](../40_krt/) and [KRT v2](../50_krt/). For more information about how to define your own
_krt_ file for version uploading please refer to these mentioned pages.

### Starting a version

Starting new versions is the action to start all the components for that version (defined in the `krt.yml` manifest) in the underlying kubernetes cluster. A started version is accessible from within the cluster but not from the outside.

{{< imgproc version_started_diagram Resize "400x" />}}

### Stopping a version

Stopping a version will remove all resources associated to that version from the cluster (defined in the `krt.yml` manifest).

### Publishing a version

Publishing a version will change the `Service` name attached to the version's entrypoint. The `Service` will be renamed to `active-entrypoint` so that the cluster ingress is linked to it. This means that the published version will have its entrypoint linked to a cluster ingress making it reachable from the outside.

Only one version can be public at a time, and if you try to publish a version while another version is published it will result in a change in the published versions.
