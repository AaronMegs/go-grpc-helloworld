# go-grpc-helloworld
go grpc example

### Generate gPRC code 
```sh
protoc --go_out=. --go_opt=paths=source_relative \
    --go-grpc_out=. --go-grpc_opt=paths=source_relative \
    helloworld/protos/*.proto
```
### Update the gRPC service
In this section you’ll update the application with an extra server method. The gRPC service is defined using protocol buffers. To learn more about how to define a service in a .proto file see Basics tutorial. For now, all you need to know is that both the server and the client stub have a SayHello() RPC method that takes a HelloRequest parameter from the client and returns a HelloReply from the server, and that the method is defined like this:

```protobuf
// The greeting service definition.
service Greeter {
// Sends a greeting
rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
string name = 1;
}

// The response message containing the greetings
message HelloReply {
string message = 1;
}
```
Open helloworld/helloworld.proto and add a new SayHelloAgain() method, with the same request and response types:

```protobuf
// The greeting service definition.
service Greeter {
// Sends a greeting
rpc SayHello (HelloRequest) returns (HelloReply) {}
// Sends another greeting
rpc SayHelloAgain (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
string name = 1;
}

// The response message containing the greetings
message HelloReply {
string message = 1;
}
```
Remember to save the file!

### Regenerate gRPC code
Before you can use the new service method, you need to recompile the updated .proto file.

While still in the examples/helloworld directory, run the following command:

```shell
$ protoc --go_out=. --go_opt=paths=source_relative \
--go-grpc_out=. --go-grpc_opt=paths=source_relative \
helloworld/helloworld.proto
```
This will regenerate the helloworld/helloworld.pb.go and helloworld/helloworld_grpc.pb.go files, which contain:
- Code for populating, serializing, and retrieving HelloRequest and HelloReply message types.
- Generated client and server code.


> from: [grpc-go](https://github.com/grpc/grpc-go)

### Compiling error, undefined: grpc.SupportPackageIsVersion

#### If you are using Go modules:

Ensure your gRPC-Go version is `require`d at the appropriate version in
the same module containing the generated `.pb.go` files.  For example,
`SupportPackageIsVersion6` needs `v1.27.0`, so in your `go.mod` file:

```go
module <your module name>

require (
    google.golang.org/grpc v1.27.0
)
```

#### If you are *not* using Go modules:

Update the `proto` package, gRPC package, and rebuild the `.proto` files:

```sh
go get -u github.com/golang/protobuf/{proto,protoc-gen-go}
go get -u google.golang.org/grpc
protoc --go_out=plugins=grpc:. *.proto
```

### How to turn on logging

The default logger is controlled by environment variables. Turn everything on
like this:

```console
$ export GRPC_GO_LOG_VERBOSITY_LEVEL=99
$ export GRPC_GO_LOG_SEVERITY_LEVEL=info
```

### The RPC failed with error `"code = Unavailable desc = transport is closing"`

This error means the connection the RPC is using was closed, and there are many
possible reasons, including:
1. mis-configured transport credentials, connection failed on handshaking
1. bytes disrupted, possibly by a proxy in between
1. server shutdown
1. Keepalive parameters caused connection shutdown, for example if you have configured
   your server to terminate connections regularly to [trigger DNS lookups](https://github.com/grpc/grpc-go/issues/3170#issuecomment-552517779).
   If this is the case, you may want to increase your [MaxConnectionAgeGrace](https://pkg.go.dev/google.golang.org/grpc/keepalive?tab=doc#ServerParameters),
   to allow longer RPC calls to finish.

It can be tricky to debug this because the error happens on the client side but
the root cause of the connection being closed is on the server side. Turn on
logging on __both client and server__, and see if there are any transport
errors.