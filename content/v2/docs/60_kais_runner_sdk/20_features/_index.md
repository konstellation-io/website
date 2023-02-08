---
title: "Runner features"
description: >
  Features available to runners compatible with KRT V2
weight: 20
---

## Send Output

The _'SendOutput'_ functionality is the bread and butter of all node. Nodes are able to publish from 0 to n messages to their own NATS subject. In order to do so, the _'SendOutput'_ function is called using a protobuf structure as argument.  
This function is used when users want to publish a _known_ protobuf structure as the payload.

Here is an example in Python:

```python
async def default_handler(ctx, _):
    ctx.logger.info("[executing default handler]")
    emails = ctx.get("emails")

    res = Response()
    res.message = f"Processing of {len(emails)} emails in progress"
    await ctx.send_early_reply(res)

    for email in emails:
        etl_output = EtlOutput()
        etl_output.email.CopyFrom(email)
        await ctx.send_output(etl_output)
    return
```

## Send Any

The _'SendAny'_ functionality is very similar to the _'SendOut'_. Nodes are able to publish from 0 to n messages to their own NATS subject using this function as well. What differs is that the protobuf structure used as payload has not been declared previously by the handler.  
This function is used when users want to publish an _unknown_ protobuf structure as the payload. As for example _redirecting_ incoming messages.

Here is an example in Python:

```python
async def default_handler(ctx, data):
  ctx.logger.info("[executing default handler]")

  if os.environ["REDIRECT_MESSAGES"] == "true":
    ctx.send_any(data) # we don't know which proto is data
    return
  
  ...

```

## Message Types

When publishing a payload to a subject the payload is encapsulated in a kre message structure, this message has a type assigned to it. There are 4 different message types:

- OK: Message has been processed correctly.
- Error: This message encountered errors during its process, there is no payload attached.
- Early reply: For early reply operations (see description below).
- Early exit: For early exit operations (see description below).

_'SendOutput'_ and _'SendAny'_ functions will send _OK_ typed messages. _Error_ typed messages will be sent by the runners if an error occurred.  
The context implements a function to check for each message type:

- _'IsMessageOK'_
- _'IsMessageError'_
- _'IsMessageEarlyReply'_
- _'IsMessageEarlyExit'_

## Early Reply

_'SendEarlyReply'_ function will publish any desired payload with an _EarlyReply_ type attached to it. Users can later on check on this by using the function _'IsMessageEarlyReply'_.

Users may use this functionality to quickly reply synchronous GRPC requests then handle the execution on a second plane.

It is advised that the exitpoint is subscribed to nodes capable of emitting early reply messages, then handle the early reply and reply to the entrypoint.  
Nodes subscribed to early reply emitting nodes should be capable of dealing with this type of message as well.  
Also, to send a payload expected by the entrypoint. This can be done by using the _SendAny_ function in the exitpoint and just redirect the payload to the entrypoint.

Here is an example in Go:  

```go
func firstNodeHandler(ctx *kre.HandlerContext, data *any.Any) error {
  ctx.Logger.Info("[first node handler invoked]")

  // don't keep the entrypoint waiting
  finalRes := &proto.Response{}
  finalRes.result = "processing messages..."
  ctx.SendEarlyReply(finalRes)

  req := &Request{}
  res := &Output{}

  ...

  return ctx.SendOutput(res)
}

func secondNodeHandler(ctx *kre.HandlerContext, data *any.Any) error {
  ctx.Logger.Info("[second node handler invoked]")

  // early replies are to be ignored
  if ctx.IsEarlyReply(){
    return
  }

  ...

}

func exitpointHandler(ctx *kre.HandlerContext, data *any.Any) error {
  ctx.Logger.Info("[exitpoint handler invoked]")

  // early replies are to be reported to the entrypoint
  if ctx.IsEarlyReply(){
    ctx.SendAny(data)
    return
  }

  req := &ExitpointRequest{}

  // process result if not an early reply
  err := anypb.UnmarshalTo(data, req, protobuf.UnmarshalOptions{})
  if err != nil {
    return fmt.Errorf("invalid request: %s", err)
  }
  
  ctx.DB.Save("results", req)

  ...

}
```

## Early Exit

_'SendEarlyExit'_ function will publish any desired payload with an _EarlyExit_ type attached to it. Users can later on check on this by using the function _'IsMessageEarlyExit'_.

Users may use this functionality to halt execution of a workflow without necessarily throwing an exception or an error.

It is advised that the exitpoint is subscribed to nodes capable of emitting early exit messages, then handle the early exit and reply to the entrypoint.  
Nodes subscribed to early reply emitting nodes should be capable of dealing with this type of messages as well.  
Also, to send a payload expected by the entrypoint. This can be done by using the _SendAny_ function in the exitpoint and just redirect the payload to the entrypoint.

Here is an example in Go:  

```go
func firstNodeHandler(ctx *kre.HandlerContext, data *any.Any) error {
  ctx.Logger.Info("[handler invoked]")

  req := &Request{}
  err := anypb.UnmarshalTo(data, req, protobuf.UnmarshalOptions{})
  if err != nil {
    return fmt.Errorf("invalid request: %s", err)
  }

  // we are not processing tests samples minor than 10 emails
  if len(req.emails) < 10 {
    finalRes := &FinalRes{}
    finalRes.result = "email length minor than 10, stopping execution"
    ctx.SendEarlyReply(finalRes)
    return
  }

  res := &Output{}

  ...

  return ctx.SendOutput(res)
}

func secondNodeHandler(ctx *kre.HandlerContext, data *any.Any) error {
  ctx.Logger.Info("[second node handler invoked]")

  // early exits are to be ignored
  if ctx.IsEarlyExit(){
    return
  }

  ...

}

func exitpointHandler(ctx *kre.HandlerContext, data *any.Any) error {
  ctx.Logger.Info("[exitpoint handler invoked]")

  // early exits are to be reported to the entrypoint
  if ctx.IsEarlyExit(){
    ctx.SendAny(data)
    return
  }

  ...

}
```
