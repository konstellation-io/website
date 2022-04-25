---
title: "KAI Server Training"
linkTitle: "KAI Server Training"
weight: 50
description: >
  KAI Server Training Project
---


The [Kai Server Training repo](https://github.com/konstellation-io/kai-server-training) is a training exercise designed 
to introduce new users to the usage and development of KRT projects. You can use the training exercise to build and
deploy your first KRT project and also to reverse engineer it and learn how to develop your own projects.

## Introduction

This example consists of a workflow called _Descriptor_ that will print information of a given GitHub repository, being
this info its last GitHub and Dockerhub tag released. The workflow is made up of 3 different nodes, each one in charge 
of a different process through the pipeline:

- **ETL**: This node will accept incoming requests and prepare a new info struct to be filled.
- **GitHub**: This node will search for the GitHub information of the repo then fill the info struct with it if found.
- **Dockerhub**: This node will search for the Dockerhub information of the repo then fill the info struct with it if found.

All nodes also save metrics to the InfluxDB instance as an example of metric handling.

## How to deploy

You can find information about building and deployment within the repo's `readme` file.

## How to use

Once deployed the generated KRT file, you can open an entrypoint to your local pod. However, to easy up usage, we've
placed some helping scripts inside a folder called `scritps`, you can find information about usage within the `readme`
file placed inside this directory as well.

## Descriptor example

{{< imgproc kai-training-example Resize "1000x" />}}

### Result example

{{< imgproc kai-training-result-example Resize "1000x" />}}

## Step-by-step guide

This step-by-step guide will aid you create a new Konstellation version for your project, which will help you understand
all the main KAI Server concepts and test KAI Server functions.


### 1. Create the version structure based on the krt-template

Start by creating an empty project with the following folders and files:


 ``` bash
    project
    └───docs   
    |   └───README.md # Documentation shown inside KAI Server
    │
    └───models    # If the process needs a model to do the inference
    │   
    └───src       # Source code of each node
    │   └───etl
    │   └───github
    │   └───dockerhub
    |
    └─── metrics
    │   └───dashboards
    |
    |   krt.yml   # krt manifest
    |   public_input.proto    # proto defining entrypoint Services and Messages.
    |   internal_nodes.proto  # proto defining Messages that interconnect nodes.
```

### 2. Fill the `krt.yml`

More information can be found [here]({{< relref "docs/KRE/user/40_krt" >}}).

```yaml
    version: descriptor-v2
    description: Version for testing.
    entrypoint:
      proto: public_input.proto
      image: konstellation/kre-entrypoint:latest

    config:
      variables:
        - GITHUB_ENABLED
        - DOCKERHUB_ENABLED

    nodes:
      - name: etl
        image: konstellation/kre-go:latest
        src: bin/etl
        # gpu is an optional value, defaults to false.
        gpu: false

      - name: github
        image: konstellation/kre-go:latest
        src: bin/github
        # gpu is an optional value, defaults to false.
        gpu: false

      - name: dockerhub
        image: konstellation/kre-go:latest
        src: bin/dockerhub
        # gpu is an optional value, defaults to false.
        gpu: false

    workflows:
      - name: go-descriptor
        entrypoint: GoDescriptor
        sequential:
          - etl
          - github
          - dockerhub
```

### 3. Fill the protobuf files

#### public_input.proto

This file will be used to define the public nodes' data contract and should contain the following info:

- `Request` message: main entrypoint entrance.
- `GithubInfo` message: Data structure for the GitHub info.
- `DockerhubInfo` message: Data structure for the Dockerhub info.
- `Response` message: Final response.
- `Entrypoint` service that routes to the first node and returns a `Response` message.

#### internal_nodes.proto

This second protobuf file defines the data contract for the internal nodes. It follows the same approach of the `public_input.proto` file to generate the following resources:

- `EtlOutput` message: Output of the ETL node.
- `GithubOutput` message: Output of the GitHub node. 


Keep in mind that in order to properly create the `GithubOutput` message, it's necessary to import the `GithubInfo` message from the `public_input.proto` file.

Nodes are defined isolated from each other in the `src` directory.

1. Create a new empty directory in `src`
```sh
mkdir node-name
```
2. Initialize the directory as Go project
```sh
go mod init node-name
```
3. Create a `main.go` as the entrypoint of the code
```sh
touch main.go
```
4. Define `init` and `handler` functions explained [here]({{< relref "docs/KRE/user/50_kais_runner_sdk" >}})

The code inside a node can be organized in many ways to keep the best clean code practices, but the root directory
should have a `main.go` file with the `init` and `handler` functions.

As this training exercise is based in creating a GitHub and DockerHub descriptor (a tool to get what are the latest
versions updated to GitHub and DockerHub of a component.), it will be necessary to create the following nodes:

- ETL: This node is in charge of processing the data passed in the entrypoint and follows the structure below:
```
  etl     
  └───go.mod
  └───go.sum
  └───main.go
  └───public_input.pb.go
```
- GitHub: This node is in charge of checking the latest version of the selected Konstellation repository.

```
  github     
  └───dockerhubservice
  |   └───mocks_service.go
  |   └───service.go
  └───go.mod
  └───go.sum
  └───main.go
  └───main_test.go
  └───public_input.pb.go
```

- Dockerhub: This node is in charge of checking the latest image uploaded to the selected Dockerhub repository.

```
  dockerhub     
  └───dockerhubservice
  |   └───mocks_service.go
  |   └───service.go
  └───go.mod
  └───go.sum
  └───main.go
  └───main_test.go
  └───public_input.pb.go
```

After creating the nodes folders, the next step is to compile the protobuf files and generate the `public_input.proto`
file inside each node folder, using the following command:

```
  protoc -I=./descriptor \
  --go_out=descriptor/src/etl \
  --go_out=descriptor/src/github \
  --go_out=descriptor/src/dockerhub \
  --go_opt=paths=source_relative descriptor/public_input.proto
```

Once the protobuf files are compiled, you can start coding the nodes.

> The full content of the nodes and protobuf files can be found [here](https://github.com/konstellation-io/kai-server-training/tree/docs/descriptor/src)

### 5. Build the .krt file

In order to upload a new version to the KAI Server, it's necessary to create a .krt file that contains all the required
files. This file can be created with the following script:

```bash
#!/bin/bash

# shellcheck disable=SC2086

## USAGE:
#    ./build_krt.sh <new-version-name>

set -eu

VERSION_DIR="descriptor"

# NOTE: if yq commands fails it due to the awesome Snap installation that is confined (heavily restricted).
# Please install yq binary from https://github.com/mikefarah/yq/releases and think twice before using Snap next time.

echo -e "Reading current version: \c"
CURRENT_VERSION=$(yq e .version ${VERSION_DIR}/krt.yml)

echo "${CURRENT_VERSION}"

VERSION=${VERSION_DIR}-${1:-${CURRENT_VERSION#$VERSION_DIR-}}

if [ -z "$VERSION" ]; then
  echo "error setting KRT version"
  exit 1;
fi

echo "Building ETL node Golang binary..."
cd descriptor/src/etl
go build -o ../../bin/etl .

echo "Building Github node Golang binary..."
cd ../github
go build -o ../../bin/github .

echo "Building Dockerhub node Golang binary..."
cd ../dockerhub
go build -o ../../bin/dockerhub .
cd ../../..


echo "Generating $VERSION.krt..."

mkdir -p build/${VERSION_DIR}
rm ./build/${VERSION_DIR}/{docs,src,assets,models,*.proto,*.yml} -rf

cd build/${VERSION_DIR}

cp  -r ../../${VERSION_DIR}/* .

yq eval --inplace -- ".version = \"${VERSION}\"" ./krt.yml

tar -zcf ../${VERSION}.krt  --exclude=*.krt --exclude=*.tar.gz *
cd ../../
rm  build/${VERSION_DIR} -rf

echo "Done"
```

### 6. Deploy and test the new version

Once the .krt file is created, it can be uploaded to KAI Server.

{{< figure src="/docs/static/upload_version.png" width="1000px" >}}

Finally, in order to test the new version, you can use the [run_test.sh]() script as follows:

```bash
run_test.sh <NAMESPACE> <VERSION_NAME> <WORKFLOW> <NUM_MSGS> <REQUESTED_COMPONENT>
```

**Example**: For our namespace `kre` and a krt version `descriptor-v1`, you can try out these commands:

  
```bash
./scripts/run_test.sh kre descriptor-v1 GoDescriptor 1 konstellation-io/kre
./scripts/run_test.sh kre descriptor-v1 GoDescriptor 1 konstellation-io/kdl-server
```