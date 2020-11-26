---
title: "Create runtime"
linkTitle: "Create runtime"
description: >
  Learn how to create a Runtime.
weight: 20
---

(***NOTE**: This feature is disable if you are running in `MONORUNTIME_MODE`.*)

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
