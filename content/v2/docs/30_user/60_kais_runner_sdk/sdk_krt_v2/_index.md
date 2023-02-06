---
title: "SDK for KRT v2"
description: >
  SDK specifications for runners compatible with KRT v2
weight: 20
---

## Init function

This function receives a context object that provides methods for logging and storing data that
users can later use on the handler function. It doesn't return any value.

`init` function is mainly used to initialize data, for example, loading models. Thus, the `init`
function will only be called once upon node startup.

### Python init example

```python
import joblib

async def init(ctx):
  ctx.logger.info("[worker init]")
  ctx.set('model', joblib.load(ctx.path('models/model.joblib')))
```

### Golang init example

```golang
package main

import (
  "github.com/konstellation-io/kre/runners/kre-go"
)

func handlerInit(ctx *kre.HandlerContext) {
  ctx.Logger.Info("init handler")
    
  // Saves a value in the context internal registry
  ctx.Set("greeting", "Hello")   
}
```

## Handler function

The `handler` function receives a context (the same as the `init` function does), and an incoming
payload as raw data.  
The context object is shared between different executions.

`handler` functions are used to process data, users can implement their service logic here.
Handlers in this version don't need to reply nor generate any message per request and can generate
several responses to different subjects if desired as well.  
To send a response, users need to make use of the function `send_output` that comes within the
_context_ object. The output sent must be a proto valid structure as it will be used as the
payload in the message sent to the following node.

Also, in this runner version users can decide which function implemented by the node will act as
the `handler` depending on the source of the incoming payload.

### Python handler example for runner compatible with KRT V2

```python
async def default_handler(ctx, req):
    if ctx.is_message_early_reply():
        return

    etl_output = EtlOutput()
    req.Unpack(etl_output)
    email = etl_output.email
    category = classify_email(email)

    res = ClassificatorOutput()
    res.email.CopyFrom(email)
    res.category = category
    if category == CATEGORY_REPARATIONS:
        await ctx.send_output(res, "repairs")
    await ctx.send_output(res)
```

### Golang handler example for runner compatible with KRT V2

```golang
func handler(ctx *kre.HandlerContext, data *anypb.Any) error {
 ctx.Logger.Info("[handler invoked]")

 req := &proto.ClassificatorOutput{}
 err := anypb.UnmarshalTo(data, req, protobuf.UnmarshalOptions{})
 if err != nil {
  return fmt.Errorf("invalid request: %s", err)
 }

 err = storeEmail(ctx, req.Email)
 if err != nil {
  ctx.Logger.Errorf("error storing email: %w", err)
 }

 return nil
}
```

## How to provide functions to the runners

Once the `init` and `handler` functions have been declared and implemented,
they have to be served in a certain way, so runners recognize them and execute them automatically.

### For runners compatible with KRT V2

#### Python

The `init` function will be a function named _init_.

```python
async def init(ctx):
  ...
```

As for the handlers, the function named _default_handler_ will act as the default handler.
Custom handlers can be declared inside a dictionary named _custom_handlers_, the dictionary will
have as key the expected source node name and value the function to execute.

```python
import pandas as pd

async def default_handler(ctx, data):
    ctx.logger.info("message received from non recognized node, executing default handler")
    ...

async def repairs_handler(ctx, data):
    ctx.logger.info("message received from node repairs, executing repairs handler")
    ...

async def spam_handler(ctx, data):
    ctx.logger.info("message received from node spam, executing spam handler")
    ...

custom_handlers = {
    "repairs": repairs_handler,
    "spam": spam_handler,
}
```

#### Golang

Golang's nodes use the Golang runners as a package. To load the runners, you need to execute
the function `start` implemented by the runners' package passing down as its variables
the functions that serve as `init` and `handler`.  
We recommend doing this inside the `main` function in the `main.go` file. Then, implement said
`init` and `handler` functions in a different package.

To load default and custom handlers, they have to be used in the function `start` from the runner's
package as their arguments. The first argument will be the _init_ function, the second the
_default handler_ and the third a map that will have as key the expected source node name and value
the function to execute.

```go
func main() {
  handlers := map[string]kre.Handler{
    "entrypoint": entrypointHandler,
    "nodeA": nodeAHandler,
  }

  kre.Start(handlerInit, defaultHandler, handlers)
}
```
