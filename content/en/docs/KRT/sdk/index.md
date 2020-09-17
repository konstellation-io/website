---
title: "SDK docs"
description: >
  SDK documentation
weight: 30
---


## Overview
The SDK created for KRE is a language-specific set of features designed to be used in message and initialization handlers. You can access these features from the context passed to your process functions.

The context provides all these functionalities:

- **In-Memory Registry**: A volatile registry that will be available from start until the version is stopped or restarted. You can set a value with a key name and retrieve it later by that key.

- **Logging**: Functions to register any message or error with different log levels: `debug, info, warning, error`. 

- **Data Persistence**: A way to persist and/or retrieve data from a DB. 

- **Measurements**: An easy way to save any amount of arbitrary measurements you need to use later on an Influx graph.

- **Metrics**: Allows you to save predicted and real data in order to feed the pre-generated charts on the "Metrics" menu for each version. 


## Golang SDK


### Context

The context provided to your handlers allows you to add or recover data from the in-memory registry. This feature is useful when you need to set some data during initialization and re-use it when processing messages. 

The methods available in the context are these:

```go
    ctx.

        func Path(relativePath string) string
        func Set(key string, value interface{})
        func Get(key string) interface{} 
        func GetString(key string) string 
        func GetInt(key string) int 
        func GetFloat(key string) float64 
```


#### 1. Reading data set in the init handler

{{<highlight go "linenos=table,hl_lines=6-7 11-12">}}
package main

import "github.com/konstellation-io/kre/runners/kre-go"

func handlerInit(ctx *kre.HandlerContext) {
    // Saves a value in the context internal registry
	ctx.Set("greeting", "Hello")   
}

func handler(ctx *kre.HandlerContext, data []byte) (interface{}, error) {
    // Recover value from the registry
    prefix := ctx.GetString("greeting")  

    // Here you can do any transformation to your data.
	greetingText := fmt.Sprintf("%s %s!", prefix, input.Name)

	return Output{ Greeting: greetingText }, nil
}
{{</highlight>}}

#### 2. Loading data into memory from an asset included in KRT

In this example you can see (line 7) how to get the path of any file relative to the root of your KRT file and then save it on the registry to use it later (line 11).

{{<highlight go "linenos=table,hl_lines=6-7 11">}}
package main

import "github.com/konstellation-io/kre/runners/kre-go"

func handlerInit(ctx *kre.HandlerContext) {
    // get the path for labels file included in the KRT
	labelPath := ctx.Path("assets/labels.json")
	
	content, err := ioutil.ReadFile(labelPath)
	data := /* unmarshal json content */  
	ctx.Set("labels", data)
}
{{</highlight>}}



### Logging

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

#### 1. Basic Logging

{{<highlight go "linenos=table,hl_lines=7 11 14 17">}}
package main

import "github.com/konstellation-io/kre/runners/kre-go"

func handlerInit(ctx *kre.HandlerContext) {
	ctx.Set("greeting", "Hello")
	ctx.Logger.Info("Process started")
}

func handler(ctx *kre.HandlerContext, data []byte) (interface{}, error) {
	ctx.Logger.Info("Performing some transformation")
    v, err := someFunction()
    if err != nil {
    	ctx.Logger.Errorf("someFunction error: %s", err)
        return
    }
	ctx.Logger.Debugf("someFunction result: %d", v)

	// ... more code... 
	return Output{ Greeting: greetingText }, nil
}

{{</highlight>}}


### Data Persistence

During any part of your workflows you may need to persist data or recover previously saved data from a DB.  

In those cases you can use the following functions:

```go
    ctx.DB.
        func Find(colName string, query QueryData, res interface{}) error 
        func Save(colName string, data interface{}) error
    
    // QueryData is the following type:
    type QueryData map[string]interface{} 
```

#### 1. Save data received in a workflow for later model retraining

In this example you can see the `Input` type which define the data received on this node. After you unmarshal raw data into your input type (line 11), you can save it in a DB (line 16) for later use. 

On this example, a Data Scientist can access all `ny-data` and use it to retrain the model to get better performance.

{{<highlight go "linenos=table,hl_lines=11 16">}}
// Input struct used to Unmarshal incoming []byte to the message handler.
type Input struct {
	Name     string `json:"name"`
	Bedrooms int `json:"bedrooms"`
	TV       bool `json:"tv""` 
}

func handler(ctx *kre.HandlerContext, rawData []byte) (interface{}, error) {
    // Decode input data into your input message struct.
	flatInfo := Input{}
	err := json.Unmarshal(data, &flatInfo)
	if err != nil {
		return nil, err
	}

	ctx.DB.Save("ny-data", flatInfo)
	// ... more code... 
}
{{</highlight>}}


### Measurements

You can add arbitrary measurements to your code that will be saved as InfluxDB data for later use.  The context provides easy access to it with this function:

```go
    ctx.Measurements.

        func Save(measurement string, fields map[string]interface{}, tags map[string]string)
```

#### 1. Saving features and prediction label as measurements

{{<highlight go "linenos=table,hl_lines=5-9">}}
func handler(ctx *kre.HandlerContext, rawData []byte) (interface{}, error) {
	// ... prediction code...
	 
	ctx.Measurements.Save("features", map[string]interface{}{
	    "bedrooms": 3,
	    "beds": 5,
	    "label": "low",
	}, map[string]string{})
	
	return ...
}
{{</highlight>}}


### Metrics

KRE has built-in graphs for classification problems. To populate the "Predictions" page of a Version, you need to save prediction values along with real values. 

Metrics context also provides an easy way to register common errors during a classification prediction: 

```go
    ctx.Metrics.
        func Save(date time.Time, predictedValue, trueValue string) 
        func SaveError(saveMetricErr SaveMetricErr)

    type SaveMetricErr string
    const ErrMissingValues SaveMetricErr = "missing_values"
    const ErrNewLabels SaveMetricErr = "new_labels"
```


#### 1. Saving real value to an existing prediction

 
{{<highlight go "linenos=table,hl_lines=2-4 6">}}
func handler(ctx *kre.HandlerContext, rawData []byte) (interface{}, error) {
	date := time.Now()
	predictedValue := "low"
	realValue := "high"
	
	ctx.Metrics.Save(date, predictedValue, realValue)
}
{{</highlight>}}


## Python SDK

