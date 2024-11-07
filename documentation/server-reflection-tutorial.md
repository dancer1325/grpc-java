# gRPC Server Reflection Tutorial

* gRPC Server Reflection
  * provides 
    * information about publicly-accessible gRPC services | server
  * assists clients | runtime -- via, WITHOUT precompiled service information, -- constructing 
    * RPC requests
    * RPC responses 
  * used by
    * gRPC CLI, to
      * introspect server protos
      * send/receive test RPCs
  * ONLY supported for proto-based services 

## Enable Server Reflection

* steps
  * include `io.grpc:grpc-services`
  * add `io.grpc.protobuf.services.ProtoReflectionService` | your gRPC server
* _Example:_ changes to enable server reflection 

    ```diff
    --- a/examples/build.gradle
    +++ b/examples/build.gradle
    @@ -27,6 +27,7 @@
     dependencies {
       compile "io.grpc:grpc-netty-shaded:${grpcVersion}"
       compile "io.grpc:grpc-protobuf:${grpcVersion}"
    +  compile "io.grpc:grpc-services:${grpcVersion}"
       compile "io.grpc:grpc-stub:${grpcVersion}"
     
       testCompile "junit:junit:4.12"
    --- a/examples/src/main/java/io/grpc/examples/helloworld/HelloWorldServer.java
    +++ b/examples/src/main/java/io/grpc/examples/helloworld/HelloWorldServer.java
    @@ -33,6 +33,7 @@ package io.grpc.examples.helloworld;
     
     import io.grpc.Server;
     import io.grpc.ServerBuilder;
    +import io.grpc.protobuf.services.ProtoReflectionService;
     import io.grpc.stub.StreamObserver;
     import java.io.IOException;
     import java.util.logging.Logger;
    @@ -50,6 +51,7 @@ public class HelloWorldServer {
         int port = 50051;
         server = ServerBuilder.forPort(port)
             .addService(new GreeterImpl())
    +        .addService(ProtoReflectionService.newInstance())
             .build()
             .start();
         logger.info("Server started, listening on " + port);
    ```

## gRPC CLI

* see [gRPC CLI](https://github.com/grpc/grpc/blob/master/doc/command_line_tool.md)

## how to run ?

* build & start the `hello-world-server`

    ```sh
    $ cd examples/
    $ ./gradlew installDist
    $ build/install/examples/bin/hello-world-server
    ```

* | `grpc_cli` binary directory

    ```sh
    $ cd <grpc-cpp-directory>/bins/opt
    ```

### List services & methods

* `grpc_cli ls port`
  * lists services & methods / exposed | given port

      ```sh
      $ ./grpc_cli ls localhost:50051
      
      # Output  
      helloworld.Greeter
      grpc.reflection.v1alpha.ServerReflection
      ```

  * `serviceName -l` 
    * list more details about a service

      ```sh
      $ ./grpc_cli ls localhost:50051 helloworld.Greeter -l
    

      # Output:
      filename: helloworld.proto
      package: helloworld;
      service Greeter {
        rpc SayHello(helloworld.HelloRequest) returns (helloworld.HelloReply) {}
      }
      ```

  * `serviceName.methodName -l`
    * list more details about a method

      ```sh
      $ ./grpc_cli ls localhost:50051 helloworld.Greeter.SayHello -l
    
      # output:
      rpc SayHello(helloworld.HelloRequest) returns (helloworld.HelloReply) {}
      ```

### Inspect message types

* `grpc_cli type port serviceName`
  * inspect request/response types

      ```sh
      $ ./grpc_cli type localhost:50051 helloworld.HelloRequest

      #output
      message HelloRequest {
        optional string name = 1[json_name = "name"];
      }
    ```

### Call a remote method

* `grpc_cli call port serviceName requestMessage`
  * send RPCs -- to a -- server

      ```sh
      $ ./grpc_cli call localhost:50051 SayHello "name: 'gRPC CLI'"
      
      # output
      message: "Hello gRPC CLI"
      ```
