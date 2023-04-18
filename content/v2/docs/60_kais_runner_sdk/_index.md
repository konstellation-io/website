---
title: "KAI Server Runner SDK"
linkTitle: "KAI Server Runner SDK"
weight: 60
description: >
  Code your nodes on top of the KAI Server Runner SDK
---

## The Runner SDK

KAI Server provides an SDK that allows users to receive, process and send messages between nodes.
As well as store metrics, predictions or other kinds of data desired by the user.

Currently, KAI Server only supports _Python_ and _Golang_ code.

Thanks to the SDK, users can implement `handler` and `init` functions, needed for the correct
functionality of nodes. These functions are implemented in different manners depending on the
_KRT version_ being used. Refer to [KRT v1](../40_krt) and [KRT v2](../40_krt_v2) for more information.

_KAI Server version_, _KRT version_ and _runner version_ are dependent of each other but only
compatible in certain manners.  
Runner features and implementation differ between the KRT version you are aiming for.  
Please refer to [SDK KRT v1](./sdk_krt_v1) and [SDK KRT v2](./sdk_krt_v2) for more information.

However, the _context_ object is common to all versions and will be explained here.

Here is a table depicting compatibility between versions:

## Compatibility table

|KRT Version|KRE version|Entrypoint version|Golang runner version|Python runner version|
|-----------|-----------|------------------|---------------------|---------------------|
|V1         |v1 to v7   |1.x.x and 2.x.x   |1.x.x and 2.x.x      |1.x.x and 2.x.x      |
|V2         |v8         |3.x.x             |3.x.x and 4.x.x      |3.x.x                |

It is recommended to use the latest available version when developing your codebase.
You can find published versions at the [Konstellation registry](https://hub.docker.com/u/konstellation).

## Context

The context object provides a set of utilities that can be used for different purposes on `init` and `handler` functions:

- __In-Memory Storage__: A volatile variable storage that will be available from start until the
  version is stopped or restarted. You can set a value with a key name and retrieve it later by that key.

- __Logging__: Functions to register any message or error with different log levels: `debug, info, warning, error`.

- __Data Persistence__: A way to persist and/or retrieve data from a DB.

- __Measurements__: An easy way to save any amount of arbitrary measurements you need to use later on an Influx graph.

- __Metrics__: Allows you to save predicted and real data in order to feed the pre-generated charts on the "Metrics" menu for each version.

- __Object Store__: A way to share data between nodes.

- __Configuration__: A centralized configuration repository.

### Database

During any part of your workflows you may need to persist data or recover previously saved data from a database.
In those cases you can use the following functions of the context's DB attribute:

```golang
    func Find(colName string, query QueryData, res interface{}) error 
    func Save(colName string, data interface{}) error
    
    // QueryData is the following type:
    type QueryData map[string]interface{} 
```

This functions will store on query data on a MongoDB database, so the data inserted should have a
BSON compatible structure.
Each runtime has its respective database whose name is `<runtime-id>-data`.

### Logger

You can use four log levels that will be shown on the Admin UI page for the Version it belongs.
The SDK will take care of saving the timestamp and tag each log with metadata to help locate the
source of the error.  

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
In order to do so you can use the function `save` from the `Measurement`. To use this function you
will have to declare previously tags and fields for your desired point, as if you were creating an influx point.  
The `save` function will take as parameters first the collection name you desire to write to,
then a dictionary of fields and a dictionary of tags.

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

To help to analyze and visualize the reliability of a model, prediction's data can be stored through
the `save` function given by the _prediction_ attribute of the context. This function receives a
timestamp, the predicted value as string, and the real value as string. This data is stored in a
MongoDB database named `<runtime-id>-data`.

```go
    ctx.Prediction.Save(time.Now(), "test-predicted-value", "test-true-value")
```

### Object Store

The object store allows users to upload large files, and it can be defined with
project or workflow scope. __Project scope__ makes the object store accessible from
all workflows, while __workflow scope__ makes it accessible only from the workflow
nodes.

The creation of object stores is managed from the _krt.yaml_ file. Through the
context, users will be able to save, get and delete objects. The object store is
ephemeral and, therefore, the persistence of the objects is ensured only during
the execution in which they were created.

Through the context, users will be able to save, get and delete objects.

```python
# Save object
await ctx.object_store.save(key, bytes(large_file, "utf-8"))

# Get object
object = await ctx.object_store.get(key)

# Delete object
await ctx.object_store.delete(key)
```

### Configuration

Through configuration, users can store parameters in a key-value format for use
elsewhere in the code. There are three configuration scopes, the __project scope__,
which is shared by all workflows in the project; the __workflow scope__, which is
shared by all workflow nodes; and the __node scope__, which is only accessible
from the node itself.

Through the context, users will be able to set, get and delete configuration.

```python
# Set configuration without a scope, by default the scope will be set to "node"
await ctx.configuration.save(key, value)

# Set configuration from scope (scope should be "project", "workflow" or "node")
await ctx.configuration.save(key, value, scope)

# Get configuration without scope. 
# By default, it reads from the most specific scope to the least specific:
# "node" scope, then "workflow" scope, and finally "project" scope.
# If the configuration exists in multiple scopes, it will be overwritten
# in the order mentioned above.
object = await ctx.configuration.get(key)

# Get configuration from scope (scope should be "project", "workflow" or "node")
object = await ctx.configuration.get(key, scope)

# Delete configuration without scope, by default the scope will be set to "node"
await ctx.configuration.delete(key)

# Get configuration from scope (scope should be "project", "workflow" or "node")
await ctx.configuration.delete(key, scope)
```
