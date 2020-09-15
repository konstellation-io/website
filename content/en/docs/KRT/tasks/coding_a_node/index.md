---
title: "Coding a node"
description: >
  An example of how to code a node.
weight: 40
---

Here you will learn how to program a node in different languages.
Currently, KRE supports Python and Golang with an SDK that provides ways to receive, process and send messages on each
node.

With the SDK you can implement the main two handler functions that a node needs. One optional function for
initialization, and one message handler function that will be the core of your node.

Both these functions, `init` and `handler`, has an input parameter that is a shared context, and contains tools to
interact with KRE, store data, share memory values, etc.


## Python

In Python your code will be dynamically loaded into the runner. The runner will look for functions with the following
signature `init(ctx)` and `handler(ctx, data)`.  Both handlers are defined as async functions to enable the possibility
of performing async tasks.

In this example we are creating a node that make predictions with a model that is included on your KRT file. You can
use the init function to load the model, so you only load it once to memory, and then use it for each incoming message
this node would get.


### Init handler

```python
import joblib

async def init(ctx):
    ctx.logger.info("[worker init]")
    ctx.set('model', joblib.load(ctx.path('models/model.joblib')))
```

The function signature is `init(ctx)`. The context object have methods for logging and storing data that you
can later use on your message handler. It doesn't need any return value.

Note that `models/model.joblib` must be included in your KRT file with this same relative path.
Check [how to create a KRT file]({{< relref "docs/KRT/tasks/create_krt_file" >}}) for more info.



### Message handler

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

The message handler signature is `handler(ctx, data)`. The context object is the same from the init handler and is also
shared between different executions. You can `set/get` data from its internal registry to store any value.

Note that the **return value must be a JSON serializable object** for KRE to process it correctly.


### Storing data to the database

```python
import pandas as pd

MEASUREMENT = 'features'
MEASUREMENT_TAGS = { 'version': 1 }

async def handler(ctx, data):
    ctx.logger.info("storing prediction and measurements")

    # Store a prediction
    await ctx.prediction.save(data['predicted_label'], data['real_label'], data['date'])

    # Store a prediction error
    await ctx.prediction.save(error='missing_values', date=data['date'])

    # Store measurements
    ctx.measurement.save(MEASUREMENT, data['measurements'], MEASUREMENT_TAGS)

    return {}
```

Prediction data can contain a successful prediction or prediction errors:

- **Successful prediction:** both predicted data and real data are required in order to properly process prediction metrics.
- **Prediction error:** predicted data and real data are not required but error parameter must be one of the following errors: `missing_values` or `new_labels`.

> In both cases date filter is not required. Omitting it will set the date to now.

> Note that `await` must be included when storing prediction data

## Golang

KRE provides support for nodes running Golang code, but currently you need to compile the binary before generating the
KRT file. This may be also automated in future releases.

Compile your binary for the architecture you need (amd/arm) and reference the resulting binary from your `krt.yml`.
Check [how to create a KRT file]({{< relref "docs/KRT/tasks/create_krt_file" >}}) for more info.



The SDK is provided as a library that you can import in your code as `github.com/konstellation-io/kre/runners/kre-go`.


You need to ctea The runner will look for functions with the following
signature `init(ctx)` and `handler(ctx, data)`.  Both handlers are defined as async functions to enable the possibility
of performing async tasks.


### Init handler

```go
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

The function signature is `handlerInit(ctx)`. The context object have methods for logging and storing data that you
can later use on your message handler. It doesn't need any return value.

NOTE: `handlerInit` is mandatory even if you don't need to do any initialization code. The function needs to be defined
in order to invoke `kre.Start()` later.


### Message handler

```go
func handler(ctx *kre.HandlerContext, data []byte) (interface{}, error) {
	ctx.Logger.Info("message received")

	// Decode input data into your input message struct.
	input := Input{}
	err := json.Unmarshal(data, &input)
	if err != nil {
		return nil, err
	}

	// Here you can do any transformation to your data.
	greetingText := fmt.Sprintf("%s %s!", ctx.Get("greeting"), input.Name)
	ctx.Logger.Info(greetingText)

	// Prepare and return the output message.
	out := Output{}
	out.Greeting = greetingText
	return out, nil // Must be a serializable JSON.
}

// Input struct used to Unmarshal incoming []byte to the message handler.
type Input struct {
	Name string `json:"name"`
}

// Output struct used as return value in message handler.
// It will be then serialized by the SDK before sending it to next node.
type Output struct {
	Greeting string `json:"greeting"`
}

```

The message handler signature is `handler(ctx *kre.HandlerContext, data []byte) (interface{}, error) `. The context
object is the same from the init handler and is also shared between different executions. You can `set/get` data from
its internal registry to store any value.

Data input to the handler function is passed as `[]byte`. You need to define an input and output structs that will be
used to unmarshal input messages and used as response.

Note that the **return value must be a JSON serializable object** for KRE to process it correctly.


### Storing data to the database

```go
const (
	missingValues   = "missing_values"
	measurement     = "features"
	measurementTags = map[string]string{}
)

func handler(ctx *kre.HandlerContext, data []byte) (interface{}, error) {
	ctx.Logger.Info("storing prediction and measurements")

	// Decode input data into your input message struct.
	input := Input{}
	err := json.Unmarshal(data, &input)
	if err != nil {
		return nil, err
	}

	// Store a prediction
	ctx.Prediction.Save(input.Date, input.Predicted, input.Real)

	// Store a prediction error
	ctx.Prediction.SaveError(missingValues)

	// Store measurements
	ctx.Measurement.Save(measurement, input.Measurements, measurementTags)

	// Prepare and return the output message.
	out := Output{}

	return out, nil // Must be a serializable JSON.
}

// Input struct used to Unmarshal incoming []byte to the message handler.
type Input struct {
	Predicted string `json:"predicted"`
	Real string `json:"real"`
	Date time.Time `json:"date"`
	Measurements map[string]interface{} `json:"measurements"`
}

type Output struct {}
```

Prediction data can contain a successful prediction or prediction errors:

- **Successful prediction:** both predicted data and real data are required in order to properly process prediction metrics.
- **Prediction error:** predicted data and real data are not required but error parameter must be one of the following errors: `missing_values` or `new_labels`.

> In both cases date filter is not required. Omitting it will set the date to now.


### Start the runner

In your main function call the SDK `Start` method passing both handlers you defined above:

```go
func main() {
	kre.Start(handlerInit, handler)
}
```

