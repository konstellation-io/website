---
title: "Call Version Services"
description: >
  Learn how to call a runtime version from an outside client or command line.
weight: 50
---

Once you have a published version you can call it from outside through the entrypoint gRPC services.

For this example the pre requisites are:

- An existing Runtime called `Test`.
- A **published version** containing a workflow pointing to a `Greeting` service in the entrypoint. 

To learn how to get this requisites you can check [KRT tasks]({{< relref "docs/KRT/tasks" >}}) and [KRE version management]({{< relref "docs/KRE/tasks/version_management" >}}). 


## Call from CLI
 
The fastest way to interact with the gRPC server is with [grpcurl](https://github.com/fullstorydev/grpcurl). This tool is similar to `curl` but for gRPC servers.

Just run this:
```bash
grpcurl -insecure -d '{"name": "John"}' entrypoint.kre-test.local:443 entrypoint.Entrypoint/Greet

# Outputs
{
  "greeting": "Hello John!"
}
```


## Call from gRPC client on Golang


### Get proto file from the Entrypoint
  
To create a gRPC client you need the proto file to generate code with `protoc` tool. Each entrypoint has its proto file published in KRE, for this example you can access it at `http://proto.kre-test.local/` (assuming your Runtime is called "Test"). You will see a file named `public_input.proto` that you have to download.


### Generate code with protoc

Inside the folder you are creating the client add a folder named `entrypoint` and put `public_input.proto` file inside. 

Now you can run `protoc` to generate the gRPC client code:

```bash 
protoc --go_out=plugins=grpc:. public_input.proto
``` 

After running this command you will see a new file called `public_input.pb.go` in the same folder.


### Code the gRPC client

Create two files, one called `go.mod` for Go to manage dependencies, and another file named `main.go`.

 
#### go.mod file 
```
module greeter-client

go 1.15

require (
	github.com/golang/protobuf v1.4.2
	google.golang.org/grpc v1.31.0
)
```

#### main.go file 
```go

package main

import (
	"context"
	"crypto/tls"
	"log"

	"google.golang.org/grpc"
	"google.golang.org/grpc/credentials"

	"greeter-client/entrypoint"
)

func main() {
	log.Println("Client started")

	address := "entrypoint.kre-test.local:443"

	var tlsConf tls.Config
	tlsConf.InsecureSkipVerify = true

    grpcOptions := grpc.WithTransportCredentials(credentials.NewTLS(&tlsConf))

	cc, err := grpc.Dial(address, grpcOptions)
	if err != nil {
		log.Fatalf("Could not connect: %v", err)
	}
	defer cc.Close()

	c := entrypoint.NewEntrypointClient(cc)

	req := entrypoint.Request{
		Name: "Go GRPC Client",
	}

	res, err := c.Greet(context.Background(), &req)
	if err != nil {
		log.Fatalf("Error calling RuntimeRPC: %v", err)
	}

	log.Printf("Response from server: %v \n %#v", res.GetGreeting(), res.String())

}

```

### Run your client

Now you can run your Go client with this command:

```bash
go run .

# Outputs
...
Response from server: Hello Go GRPC Client!

```

