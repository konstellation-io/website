---
title: "Getting started"
linkTitle: "Getting started"
description: >
  We are going to learn how to create runtimes and versions.
weight: 40
---

## Prerequisites

First of all, you will need a KRE environment installed. Read more info about the installation [here](../installing-kre).

## Login

KRE login is a passwordless process so you only need to provide your email:

{{< imgproc login_screen Resize "600x" >}}

{{< /imgproc >}}

After submitting your email address you will see the following screen:

{{< imgproc login_passwordless Resize "600x" >}}

{{< /imgproc >}}

You will receive an email with a login link:

{{< imgproc login_link Resize "600x" >}}

{{< /imgproc >}}

Clicking the login link you will navigate to the runtime list screen.

{{< imgproc user_viewer Resize "1000x" >}}

{{< /imgproc >}}

If you are a new user, your role will be `viewer` so you should ask to the admin user for privileges for edition and creation (`manager` role).
An `admin` user can change the user role using the user administration interface (`/settings/users`):

{{< imgproc user_admin Resize "1000x" >}}

{{< /imgproc >}}

### User roles

There are three roles for users in KRE. The following table shows the available actions for each role/resource:

| Role    | Description                                                 | Metrics | Runtimes  | Versions  | Audits | Logs | Settings  | Users     |
| ------- | ----------------------------------------------------------- | ------- | --------- | --------- | ------ | ---- | --------- | --------- |
| admin   | Full control                                                | view    | view/edit | view/edit | view   | view | view/edit | view/edit |
| manager | Can do anything but management of settings and users        | view    | view/edit | view/edit | view   | view | -         | -         |
| viewer  | Only can see the metrics, created runtimes and its versions | view    | view      | view      | -      | -    | -         | -         |

## Create a new Runtime

If you have `manager` role, you will see the "ADD RUNTIME" button in the runtimes screen:

{{< imgproc runtimes_screen Resize "1000x" >}}

{{< /imgproc >}}

We are going to create a new runtime called `test`:

{{< imgproc add_runtime Resize "600x" >}}

{{< /imgproc >}}

After clicking the `save` button, we will see the `test` runtime in the runtimes screen with status `creating`.
At this moment KRE is creating all necessary resources inside Kubernetes and it can take a time.

{{< imgproc runtime_creating Resize "1000x" >}}

{{< /imgproc >}}

After a while, the runtime status should be `started` or if there was a problem with the initialization the status should be `error`.

{{< imgproc runtime_started Resize "400x" >}}

{{< /imgproc >}}

## Upload a new Version

When all runtime resources are ready, you can click the created runtime in order to see the list of versions:

{{< imgproc versions_screen Resize "1000x" >}}

{{< /imgproc >}}

We are going to upload a version of the [greeting example](https://github.com/konstellation-io/kre/tree/master/krt-template/greeter). You can download the krt file [here](./greeter-v1.krt).

{{< imgproc add_version Resize "600x" >}}

{{< /imgproc >}}

After creating the greeting version, you will see the version status screen:

{{< imgproc greeting_version_screen Resize "1000x" >}}

{{< /imgproc >}}

As you can see, the [greeting example](https://github.com/konstellation-io/kre/tree/master/krt-template/greeter) has two workflows called `greeting` and `saluting`. Both workflows have only one process called `greeter`. The version status at this moment is `stopped` so the `greeter` processes color are grey. A workflow is a sequence of connected nodes, the first and the last node are called `entrypoint`.

{{< imgproc greeting_version_screen_explained Resize "1000x" >}}

{{< /imgproc >}}

### Version lifecycle

In the following grahp you can see all posible statuses and actions for a runtime version. The blue boxes are actions and the green ones are statuses:

{{< imgproc version_lifecycle Resize "1000x" >}}

{{< /imgproc >}}

## Version management

In the version details screen we can manage the opened version. There are four actions that we can perform using the left-bottom buttons: start, stop, publish and unpublish. When we perform some of these actions the version status change. The following table shows all possible version status:

| Status      | Description                                                                                                                                  |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| `stopping`  | Indicates the version is deleting their associated k8s resources.                                                                            |
| `stopped`   | The version is created in KRE but it is not consuming any resource because all components are not created at k8s.                            |
| `starting`  | The version is creating their associated k8s resources.                                                                                      |
| `started`   | The version entrypoint and processes are running and ready in k8s. The entrypoint ingress is not created so you cannot call it from outside. |
| `published` | The entrypoint is accesible from outside and the incoming request can be managed by the processes.                                           |

All of these actions are important so you must provide a reason text and they are registered in the user audit list (`/audit`). Only `manager` or `admin` users can view the user audit list.

### Start a version

Clicking the `Start` button and providing the reason text we will see that the version goes to the `starting` status and after a time changes to `started`.

{{< imgproc version_started Resize "1000x" >}}

{{< /imgproc >}}

The processes color should be green, this means that the processes are ready and listening for incoming messages.

### Stop a version

Using the actions buttons you can press the `Stop` button when the version is `started` or `starting`.
Use this action when you want to delete all associated resources in kubernetes for this version.
The collected logs and metrics will be persisted after stopping.

### Publish a version

A started version with all nodes on green means that the version is ready to receive messages but it is not .
You can use this state to be sure that all pieces of the workflow are working correctly before publishing.
