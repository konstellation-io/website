---
title: "Create version"
linkTitle: "Create version"
description: >
  Tutorial about the runtime versions creation.
weight: 30
---

When all runtime resources are ready, you can click the created runtime in order to see the list of versions:

{{< imgproc versions_screen Resize "1000x" >}}

{{< /imgproc >}}

We are going to upload a version of the [greeting example](https://github.com/konstellation-io/kre/tree/master/krt-template/greeter). You can download the krt file [here](/website/krts/greeter-v1.krt).

{{< imgproc add_version Resize "600x" >}}

{{< /imgproc >}}

After creating the greeting version, you will see the version status screen:

{{< imgproc greeting_version_screen Resize "1000x" >}}

{{< /imgproc >}}

As you can see, the [greeting example](https://github.com/konstellation-io/kre/tree/master/krt-template/greeter) has two workflows called `greeting` and `saluting`. Both workflows have only one process called `greeter`. The version status at this moment is `stopped` so the `greeter` processes color are grey. A workflow is a sequence of connected nodes, the first and the last node are called `entrypoint`.

{{< imgproc greeting_version_screen_explained Resize "1000x" >}}

{{< /imgproc >}}
