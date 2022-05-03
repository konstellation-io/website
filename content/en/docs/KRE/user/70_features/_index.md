---
title: "Features"
linkTitle: "Features"
weight: 70
description: >
  Current features available to users
---

## Early Reply

Early reply allows asynchronous calling to an endpoint. So now instead of waiting for the whole
execution of the workflow to finish users can reply directly to the endpoint from anywhere within
the workflow.

The endpoint will receive an early reply message and close connection to the GRPC client, the workflow
will continue execution in the background.

{{< imgproc early_reply_example Resize "1000x" />}}

This can be done from any node by calling the function `Reply` provided in the `ctx` structure.
It is needed a proto compliant message to be returned from the `Reply` function, the message can even be empty.

Here is an example in Go:

```go
func handler(ctx *kre.HandlerContext, data *any.Any) (proto.Message, error) {
 ctx.Logger.Info("[handler invoked]")

 req := &Request{}
 res := &EtlOutput{}

 ctx.Reply(res)

 ...

 return res, nil

}
```
