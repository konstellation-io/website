---
title: "Create a KRT"
description: >
   A guide to create a KRT file.
weight: 50
---

The end goal of Konstellation is to automate the generation of KRT files directly from KDL. This is currently on the list 
of features to add on KDL but not developed yet.

So, in order to create a KRT file, you need to create your own custom scripts to generate KRT files. Basically you need
 to create a `.tar.gz` file with the content of your version and then change its extension to `.krt`.
 
You can see a working example of this script on KRE repo: [build_krt.sh](https://github.com/konstellation-io/kre/blob/master/krt-template/build_krt.sh)


## Structure guidelines


The mandatory rules are these:

- a `krt.yml` file on  the root folder.
- (optional) a README.md file inside a `docs` folder.  
 
Other than that you are free to structure the content as you see fit.


### Example

This is how we structure or KRT files, and it's probably the way it will be exported automatically from KDL.  

_**NOTE**_: This example match with the `krt.yml` file used in [krt.yml file example]({{< relref "docs/krt/tasks/define_krt_yml#example-file" >}}) 

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
