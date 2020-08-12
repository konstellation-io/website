---
title: "Folder structure"
description: >
   A description and example folder structure of a KRT file.
weight: 20
---

The KRT file contains all assets and source code needed for the given version to run. 

The mandatory rules are these:

- a `krt.yml` file on  the root folder.
- (optional) a README.md file inside a `docs` folder.  
 
Other than that you are free to structure the content as you see fit.

## Structure guidelines

This is how we structure or KRT files, and it's probably the way it will be exported automatically from KDL.  


### Example structure

_**NOTE**_: This example match with the `krt.yml` file used in [krt.yml file example]({{< relref "docs/krt/krt_yaml_file.md#example-file" >}}) 

```
<root_folder>

   - bin
     - client-metrics   # Go compiled binary for client-metric node

   - docs
     - README.md        # Documentation showed as HTML on KRE

   - models             # Model assets used on execute_model node
      model.joblib
      encoder.joblib
     
   - src                # Source code of each node
     - etl
       - execute_etl.py
     - execute_model
       - execute_model.py
     - output
       - output.py
     - client-metrics
       - main.go
       - go.mod

   - kry.yml             # KRT file definition 

   - public_input.proto  # Used to define entrypoint GRPC messages and services.
 
```
