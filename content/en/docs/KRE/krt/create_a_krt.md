---
title: "Create a KRT file"
description: >
   An example of how to create a file from source content.
weight: 50
---

The end goal of Konstellation is to automate the generation of KRT files directly from KDL. This is currently on the list 
of features to add on KDL but not developed yet.

So, in order to create a KRT file, you need to create your own custom scripts to generate KRT files. Basically you need
 to create a `.tar.gz` file with the content of your version and then change its extension to `.krt`.
 
You can see a working example of this script on KRE repo: [build_krt.sh](https://github.com/konstellation-io/kre/blob/master/krt-template/build_krt.sh)


## Using Golang 

KRE provides support for nodes running Golang code, but currently you need to compile the binary before generating the 
KRT file. This may be also automated in future releases. 

Compile your binary for the architecture you need (amd/arm) and reference the resulting binary from your `krt.yml`.  
