---
title: "Version Management"
linkTitle: "Version Management"
description: >
  We are going to learn how to manage a runtime version.
weight: 40
---

### Version lifecycle

In the following graph you can see all possible statuses and actions for a runtime version. The blue boxes are actions, and the green ones are statuses:

{{< imgproc version_lifecycle Resize "1000x" >}}

{{< /imgproc >}}


## Version management

In the version details screen we can manage the opened version. There are four actions that we can perform using the left-bottom buttons: start, stop, publish and un-publish. When we perform some of these actions the version status change. The following table shows all possible version status:

| Status      | Description                                                                                                                                  |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| `stopping`  | Indicates the version is deleting their associated k8s resources.                                                                            |
| `stopped`   | The version is created in KRE but it is not consuming any resource because all components are not created at k8s.                            |
| `starting`  | The version is creating their associated k8s resources.                                                                                      |
| `started`   | The version entrypoint and processes are running and ready in k8s. The entrypoint ingress is not created so you cannot call it from outside. |
| `published` | The entrypoint is accessible from outside and the incoming request can be managed by the processes.                                           |

All of these actions ask the user to input a reason that is registered as an audit log. An `admin` or `manager` user can see all users activity on the audit list (`/audit`). 


### Start a version

Clicking the `Start` button and providing the reason text we will see that the version goes to the `starting` status and after a time changes to `started`.

{{< imgproc version_started Resize "1000x" >}}

{{< /imgproc >}}

The processes color should be green, this means that the processes are ready and listening for incoming messages.


### Stop a version

Using the action buttons you can press the `Stop` button when the version is `started` or `starting`.
Use this action when you want to delete all associated resources in kubernetes for this version.
The collected logs and metrics will be persisted after stopping.


### Publish a version

A started version with all nodes on green means that the version is ready to receive messages, but it is not exposed to the outside world until you decide to publish it.
You can use this state to be sure all pieces of the workflow are working correctly before publishing.
