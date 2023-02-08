---
title: "SDK for KRT v1"
description: >
  SDK specifications for runners compatible with KRT v1
weight: 10
---

## Init function

This function receives a context object that provides methods for logging and storing data that users
can later use on the handler function. It doesn't return any value.

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
Handlers must return a proto valid structure that will be used as payload in the message sent to the
following node. It is compulsory that one request generates one answer.

### Python handler example for runner compatible with KRT V1

```python
import pandas as pd

async def handler(ctx, data):
  req = ModelOutput()
  data.Unpack(req)

  market_price, label = get_market_price_and_label(req.price_category, ctx.get('currency'))
  ctx.logger.info(f"Estimated market price[{market_price}] with label[{label}]")

  # Stores input fields and prediction to measurements
  measurement_fields = MessageToDict(req.request, preserving_proto_field_name=True,
                                       including_default_value_fields=True)
  measurement_fields['prediction'] = label
  ctx.measurement.save(MEASUREMENT, measurement_fields, MEASUREMENT_TAGS)

  res = Response()
  res.success = True
  res.message = 'Predicted market price'
  res.market_price = market_price
  res.category = label

  return res
```

### Golang handler example for runner compatible with KRT V1

```golang
func handler(ctx *kre.HandlerContext, data *any.Any) (proto.Message, error) {
  ctx.Logger.Info("[handler invoked]")

  req := &Request{}
  res := &EtlOutput{}

  err := anypb.UnmarshalTo(data, req, proto2.UnmarshalOptions{})
  if err != nil {
    return res, fmt.Errorf("invalid request: %s", err)
  }

  res.Component = req.Component

  saveEtlMetrics(ctx, req.Component)

  return res, nil
}
```

## How to provide functions to the runners

Once the `init` and `handler` functions have been declared and implemented,
they have to be served in a certain way, so runners recognize them and execute them automatically.

### For runners compatible with KRT V1

#### Python

Just by having two functions named __init__ and __handler__. Python runners will interpret what
source code is given.

```python
async def init(ctx):
  ...

async def handler(ctx, data):
  ...
```

#### Golang

Golang's nodes use the Golang runners as a package. To load the runners, you need to execute
the function `start` implemented by the runners' package passing down as its variables
the functions that serve as `init` and `handler`.  
We recommend doing this inside the `main` function in the `main.go` file. Then, implement said
`init` and `handler` functions in a different package.

```go
func main() {
  kre.Start(handlerInit, handler)
}
```
