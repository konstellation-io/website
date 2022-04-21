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

`init` function is mainly used to initialize data, for example, loading models.

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

The context provides all these functionalities:

- **In-Memory Storage**: A volatile variable storage that will be available from start until the version is stopped or restarted. You can set a value with a key name and retrieve it later by that key.

- **Logging**: Functions to register any message or error with different log levels: `debug, info, warning, error`.

- **Data Persistence**: A way to persist and/or retrieve data from a DB.

- **Measurements**: An easy way to save any amount of arbitrary measurements you need to use later on an Influx graph.

- **Metrics**: Allows you to save predicted and real data in order to feed the pre-generated charts on the "Metrics" menu for each version.

### Database

During any part of your workflows you may need to persist data or recover previously saved data from a DB.  
In those cases you can use the following functions:

```golang
    func Find(colName string, query QueryData, res interface{}) error 
    func Save(colName string, data interface{}) error
    
    // QueryData is the following type:
    type QueryData map[string]interface{} 
```

### Logger

You can use four log levels that will be shown on Admin UI page for the Version it belongs. The SDK will take care of saving the timestamp and tag each log with metadata to locate the source of the error.
These are the available levels and their variants for multiple parameters:
```go
    ctx.Logger.
    
        func Debug(msg string) 
        func Info(msg string)
        func Warn(msg string) 
        func Error(msg string) 
    
        func Debugf(format string, a ...interface{})
        func Infof(format string, a ...interface{}) 
        func Warnf(format string, a ...interface{}) 
        func Errorf(format string, a ...interface{}) 
```

### Measurements

### Predictions