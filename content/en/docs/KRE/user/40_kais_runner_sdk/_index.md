---
title: "KAI Server Runner SDK"
linkTitle: "KAI Server Runner SDK"
weight: 50
description: >
  Code your nodes on top of the KAI Server Runner SDK
---


## The Runner SDK

KAI Server supports Python and Golang providing an SDK that allow users to receive,
process and send messages between nodes.

Thanks to the SDK, users can implement `handler` and `init` functions, which are explained below.

## Init function

This function receives a context that provides methods for logging and storing data that users
can later use on the handler function. It doesn't return any value.

[//]: # (TODO: Verify next paragraph)
`init` function is mainly used to initialize data, for example, loading models, because its only executed once,
when the node is started.

### Python example

```python
async def handler(ctx, data):
    ctx.logger.info("message received")

    # Create a dataframe from input dict
    df = pd.DataFrame.from_dict(data)

    # Access pre-loaded data from context loaded on init() function 
    model = ctx.get('model')

    # Run a prediction
    ctx.logger.info('running prediction')
    prediction = model.predict(df)

    # Return prediction value as a JSON serializable object
    return {'price_category': prediction.item()}
```

### Golang example

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

This function receives a context, which is the same from the `init` function, and the data, which is the message
sent from the previous node or from the client. The context object is shared between different executions.

### Python example

```python
import pandas as pd

async def handler(ctx, data):
    ctx.logger.info("message received")

    # Create a dataframe from input dict
    df = pd.DataFrame.from_dict(data)

    # Access pre-loaded data from context loaded on init() function 
    model = ctx.get('model')

    # Run a prediction
    ctx.logger.info('running prediction')
    prediction = model.predict(df)

    # Return prediction value as a JSON serializable object
    return {'price_category': prediction.item()}
```

### Golang example

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

## Secrets & Environment

## Config

### Variables

### Files

## Context

The context object provides a set of utilities as database client or logger that can be used
for different purposes on `init` or `handler` functions.

### Database

### Logger

The logger object provides a way to handle logs that should be visible through the KAI Server UI.

```golang
ctx.Logger.Info("sample log")
```

### Measurements

### Predictions