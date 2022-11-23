---
title: "KAI Server Runner SDK"
linkTitle: "KAI Server Runner SDK"
weight: 60
description: >
  Code your nodes on top of the KAI Server Runner SDK
---


## The Runner SDK

KAI Server supports Python and Golang providing an SDK that allows users to receive,
process and send messages between nodes.

Thanks to the SDK, users can implement `handler` and `init` functions.

## Init function

This function receives a context that provides methods for logging and storing data that users
can later use on the handler function. It doesn't return any value.

`init` function is mainly used to initialize data, for example, loading models.

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

This function receives a context, the same as the `init` function does, and the data, which is the incoming payload to process sent from this node's subscription. The context object is shared between different executions.

`Handler` functions are used to process data, users can implement their service logic here.

### Python handler example for runner compatible with KRT V1

```python
import pandas as pd

async def handler(ctx, data):
    ctx.logger.info("message received")

    # Create a dataframe from input dict
    df = pd.DataFrame.from_dict(data)

    # Access preloaded data from context loaded on init() function 
    model = ctx.get('model')

    # Run a prediction
    ctx.logger.info('running prediction')
    prediction = model.predict(df)

    # Return prediction value as a JSON serializable object
    return {'price_category': prediction.item()}
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

## How to provide node functions to kre

Once the `init` and `handler` functions have been declared and implemented, they have to be served in a certain way, so runners recognize them.

### For runners compatible with KRT V1

#### Python (images v1 and v2)

Just by having two functions named __init__ and __handler__. Python runners interpret what source code is given.

#### Golang (images v1 and v2)

Golang nodes use the Golang runners as a library. To load them up, inside the `main` function in the `main.go` file the runner must be run passing down the functions that serve as `init` and `handler`.

```go
func main() {
  kre.Start(handlerInit, handler)
}
```

### For runners compatible with KRT V2

In this runner version you can decide which function implemented by the node will act as the `handler` depending on the source of the incoming payload.

#### Python (image v3)

The function named _default_handler_ will act as the default handler. Custom handlers will be declared inside a dictionary named _custom_handlers_, the dictionary will have as key the source node name and as value the function to execute.

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

#### Golang (image v4)

To load default and custom handlers, they have to be used in the function `Start` from the runner's library as their arguments. The first argument will be the _init_ function, the second the _default handler_ and the third a map of key the source node's name and value the function to execute.

```go
func main() {
  handlers := map[string]kre.Handler{
    "entrypoint": entrypointHandler,
    "nodeA": nodeAHandler,
  }

  kre.Start(handlerInit, defaultHandler, handlers)
}
```

## Secrets & Environment

## Config

### Variables

### Files

## Context

The context object provides a set of utilities that can be used  for different purposes on `init` and `handler` functions:

- __In-Memory Storage__: A volatile variable storage that will be available from start until the version is stopped or restarted. You can set a value with a key name and retrieve it later by that key.

- __Logging__: Functions to register any message or error with different log levels: `debug, info, warning, error`.

- __Data Persistence__: A way to persist and/or retrieve data from a DB.

- __Measurements__: An easy way to save any amount of arbitrary measurements you need to use later on an Influx graph.

- __Metrics__: Allows you to save predicted and real data in order to feed the pre-generated charts on the "Metrics" menu for each version.

### Database

During any part of your workflows you may need to persist data or recover previously saved data from a database.
In those cases you can use the following functions of the context's DB attribute:

```golang
    func Find(colName string, query QueryData, res interface{}) error 
    func Save(colName string, data interface{}) error
    
    // QueryData is the following type:
    type QueryData map[string]interface{} 
```

This functions will store on query data on a MongoDB database, so the data inserted should have a BSON compatible structure.
Each runtime has its respective database whose name is `<runtime-id>-data`.

### Logger

You can use four log levels that will be shown on the Admin UI page for the Version it belongs. The SDK will take care
of saving the timestamp and tag each log with metadata to help locate the source of the error.
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

Metrics can be stored in the runtime's influx bucket anytime.  
In order to do so you can use the function `save` from the `Measurement`. To use this function you will have to declare previously tags and fields for your desired point, as if you were creating an influx point.  
The `save` function will take as parameters first the collection name you desire to write to, then a dictionary of fields and a dictionary of tags.

By default, this function will attach to the saved tags to every metric the following tags:

- "version": the version name to which this node belongs
- "workflow": the workflow name to which this node belongs
- "node": the node's name

It is advised to document yourself about influxDB metrics before using this function.

Here is an example in Go:

```go
func saveMetrics(ctx *kre.HandlerContext, component string) {
  tags := map[string]string{}

  fields := map[string]interface{}{
    "requested_component": component,
  }

  // influx is accessible via the ctx.Measurement variable
  ctx.Measurement.Save("requests", fields, tags)

  fields = map[string]interface{}{
    "called_node": "etl",
  }

  ctx.Measurement.Save("number_of_calls", fields, tags)
}
```

### Predictions

To help to analyze and visualize the reliability of a model, prediction's data could de stored through the `save` function
of the Prediction attribute of the context. This function receives a timestamp, the predicted value as string, and the
real value as string. This data is stored in a MongoDB database named `<runtime-id>-data`.

```go
    ctx.Prediction.Save(time.Now(), "test-predicted-value", "test-true-value")
```
