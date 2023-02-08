---
title: "Runner features"
description: >
  Features available to runners compatible with KRT V1
weight: 20
---

## Early Reply

Early reply allows asynchronous calling to an endpoint. So now instead of waiting for the whole execution of the workflow to finish, users can reply directly to the endpoint from anywhere within the workflow.

The endpoint will receive an early reply message and close connection to the GRPC client, the workflow will continue execution in the background.

{{< imgproc early_reply_example Resize "1000x" />}}

This can be done from any node by calling the function `EarlyReply` provided in the `ctx` structure.
The reply message must be a proto message compliant with the response expected by the entrypoint, the message can also be empty.

Remember that the entrypoint can only be answered once. So if this function is called several times,
subsequent replies will be discarded by the runner as a fail-safe procedure.

Here is an example in Go:  

```go
func handler(ctx *kre.HandlerContext, data *any.Any) (proto.Message, error) {
  ctx.Logger.Info("[handler invoked]")

  req := &Request{}
  finalRes := &proto.Response{}

  ctx.EarlyReply(finalRes)

  res := &EtlOutput{}

  ...

  return res, nil
}
```

## Early Exit

The early exit feature allows users to end workflow execution when desired.

By calling the function `SetEarlyExit` provided in the `ctx` structure, we are
instructing the node to reply directly to the entrypoint instead of its following
workflow node, _only for this execution_.

{{< imgproc early_exit_example Resize "1000x" />}}

Thus, the workflow execution will be stopped without needing to throw exceptions or temper with
the workflow execution as the following nodes will not be called.

Remember that the entrypoint can only be answered once. So if this several replies are sent,
subsequent replies will be discarded by the runner as a fail-safe procedure.

Here is an example in Go:  

```go
func handler(ctx *kre.HandlerContext, data *any.Any) (proto.Message, error) {
  ctx.Logger.Info("[handler invoked]")

  req := &Request{}
  res := &EtlOutput{}

  ...

  if conditionExample {
   ctx.SetEarlyExit()
   finalRes := &proto.Response{
    Msg: "This request will not be processed"
   }
   return finalRes, nil
  }

  ...

  return res, nil
}
```
