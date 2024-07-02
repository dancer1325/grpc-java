gRPC Examples
==============================================

## Prerequisites
* [JDK7+](https://jdk.java.net/)
* `grpc-java` 
  * ways
    * check out a git release tag -- which contains a build of gRPC --
    * follow [COMPILING](../COMPILING.md)


## <a name="to-build-the-examples"></a> Ways to build & run the examples

### Via Gradle
* From grpc-java/examples directory
  * ` $ ./gradlew installDist`
    * Creates the scripts `hello-world-server`, `hello-world-client`, `route-guide-server`, `route-guide-client`, etc. in the `build/install/examples/bin/` directory that run the examples.
  * `$ ./build/install/examples/bin/*-server` -- _Example:_ `$ ./build/install/examples/bin/hello-world-server` 
    * server -- must running before -- the client
  *  (in a different terminal) `$ ./build/install/examples/bin/*-client` -- _Example:_ `$ ./build/install/examples/bin/hello-world-client` --
    * run the client
    * check the log message, confirming the response from the server!!!



### Via Maven
* From grpc-java/examples directory
  * `$ mvn verify`
  * `$ mvn exec:java -Dexec.mainClass=io.grpc.examples.*.*Server` -- _Example:_ `$ mvn exec:java -Dexec.mainClass=io.grpc.examples.helloworld.HelloWorldServer`
    * run the server
  * (In another terminal) `$ mvn exec:java -Dexec.mainClass=io.grpc.examples.*.*Client` -- _Example:_ `$ mvn exec:java -Dexec.mainClass=io.grpc.examples.helloworld.HelloWorldClient`
    * run the client
    * check the log message, confirming the response from the server!!!


### Via Bazel
* [Install bazel](https://bazel.build/install)
* From grpc-java/examples directory
  * `$ bazel build :*-server :*-client` -- _Example:_ `$ bazel build :hello-world-server :hello-world-client`
    * creates 'bazel-bin/', 'bazel-examples/', 'bazel-out/' 
  * `$ bazel-bin/*-server`  _Example:_ `$ bazel-bin/hello-world-server`
    * run the server
  * (In another terminal) `$ bazel-bin/*-client` -- _Example:_ `$ bazel-bin/hello-world-client`
    * run the client
    * check the log message, confirming the response from the server!!!


## Basic examples

- [Hello world](src/main/java/io/grpc/examples/helloworld)

- [Route guide](src/main/java/io/grpc/examples/routeguide)

- [Metadata](src/main/java/io/grpc/examples/header)

- [Error handling](src/main/java/io/grpc/examples/errorhandling)

- [Compression](src/main/java/io/grpc/examples/experimental)

- [Flow control](src/main/java/io/grpc/examples/manualflowcontrol)

- [Wait For Ready](src/main/java/io/grpc/examples/waitforready)

- [Json serialization](src/main/java/io/grpc/examples/advanced)

- <details>
  <summary>Hedging</summary>

  The [hedging example](src/main/java/io/grpc/examples/hedging) demonstrates that enabling hedging
  can reduce tail latency. (Users should note that enabling hedging may introduce other overhead;
  and in some scenarios, such as when some server resource gets exhausted for a period of time and
  almost every RPC during that time has high latency or fails, hedging may make things worse.
  Setting a throttle in the service config is recommended to protect the server from too many
  inappropriate retry or hedging requests.)

  The server and the client in the example are basically the same as those in the
  [hello world](src/main/java/io/grpc/examples/helloworld) example, except that the server mimics a
  long tail of latency, and the client sends 2000 requests and can turn on and off hedging.

  To mimic the latency, the server randomly delays the RPC handling by 2 seconds at 10% chance, 5
  seconds at 5% chance, and 10 seconds at 1% chance.

  When running the client enabling the following hedging policy

  ```json
        "hedgingPolicy": {
          "maxAttempts": 3,
          "hedgingDelay": "1s"
        }
  ```
  Then the latency summary in the client log is like the following

  ```text
  Total RPCs sent: 2,000. Total RPCs failed: 0
  [Hedging enabled]
  ========================
  50% latency: 0ms
  90% latency: 6ms
  95% latency: 1,003ms
  99% latency: 2,002ms
  99.9% latency: 2,011ms
  Max latency: 5,272ms
  ========================
  ```

  See [the section below](#to-build-the-examples) for how to build and run the example. The
  executables for the server and the client are `hedging-hello-world-server` and
  `hedging-hello-world-client`.

  To disable hedging, set environment variable `DISABLE_HEDGING_IN_HEDGING_EXAMPLE=true` before
  running the client. That produces a latency summary in the client log like the following

  ```text
  Total RPCs sent: 2,000. Total RPCs failed: 0
  [Hedging disabled]
  ========================
  50% latency: 0ms
  90% latency: 2,002ms
  95% latency: 5,002ms
  99% latency: 10,004ms
  99.9% latency: 10,007ms
  Max latency: 10,007ms
  ========================
  ```

</details>

- <details>
  <summary>Retrying</summary>

  The [retrying example](src/main/java/io/grpc/examples/retrying) provides a HelloWorld gRPC client &
  server which demos the effect of client retry policy configured on the [ManagedChannel](
  ../api/src/main/java/io/grpc/ManagedChannel.java) via [gRPC ServiceConfig](
  https://github.com/grpc/grpc/blob/master/doc/service_config.md). Retry policy implementation &
  configuration details are outlined in the [proposal](https://github.com/grpc/proposal/blob/master/A6-client-retries.md).

  This retrying example is very similar to the [hedging example](src/main/java/io/grpc/examples/hedging) in its setup.
  The [RetryingHelloWorldServer](src/main/java/io/grpc/examples/retrying/RetryingHelloWorldServer.java) responds with
  a status UNAVAILABLE error response to a specified percentage of requests to simulate server resource exhaustion and
  general flakiness. The [RetryingHelloWorldClient](src/main/java/io/grpc/examples/retrying/RetryingHelloWorldClient.java) makes
  a number of sequential requests to the server, several of which will be retried depending on the configured policy in
  [retrying_service_config.json](src/main/resources/io/grpc/examples/retrying/retrying_service_config.json). Although
  the requests are blocking unary calls for simplicity, these could easily be changed to future unary calls in order to
  test the result of request concurrency with retry policy enabled.

  One can experiment with the [RetryingHelloWorldServer](src/main/java/io/grpc/examples/retrying/RetryingHelloWorldServer.java)
  failure conditions to simulate server throttling, as well as alter policy values in the [retrying_service_config.json](
  src/main/resources/io/grpc/examples/retrying/retrying_service_config.json) to see their effects. To disable retrying
  entirely, set environment variable `DISABLE_RETRYING_IN_RETRYING_EXAMPLE=true` before running the client.
  Disabling the retry policy should produce many more failed gRPC calls as seen in the output log.

  See [the section below](#to-build-the-examples) for how to build and run the example. The
  executables for the server and the client are `retrying-hello-world-server` and
  `retrying-hello-world-client`.

</details>

- <details>
  <summary>Health Service</summary>

  The [health service example](src/main/java/io/grpc/examples/healthservice)
  provides a HelloWorld gRPC server that doesn't like short names along with a
  health service.  It also provides a client application which makes HelloWorld
  calls and checks the health status.

  The client application also shows how the round robin load balancer can
  utilize the health status to avoid making calls to a service that is
  not actively serving.
</details>


- [Keep Alive](src/main/java/io/grpc/examples/keepalive)

## Other examples

- [Android examples](android)

- Secure channel examples

  + [TLS examples](example-tls)

  + [ALTS examples](example-alts)

- [Google Authentication](example-gauth)

- [JWT-based Authentication](example-jwt-auth)

- [OAuth2-based Authentication](example-oauth)

- [Pre-serialized messages](src/main/java/io/grpc/examples/preserialized)

## Unit test examples
* Check [examples/src/test](src/test)
* 👁️Recommendations 👁️
  * NOT allow overriding the client stub
  * NOT support mocking (nor stubs / responses, ...) in gRPC-Java library
    * == NOT use tools like PowerMock or [mockito-inline](https://search.maven.org/search?q=g:org.mockito%20a:mockito-inline)
    * it's hard to mock
    * _Example bugs not caught by mocked stub tests:_
      * stub -- is called with a -- `null` message
      * Not calling `close()`
      * Sending invalid headers
      * Ignoring deadlines
      * Ignoring cancellation 
  * Use `InProcessTransport`
    * := light-weight / server and client runs in the same process without any socket/TCP connection
    * Where is it used in the examples ❓
    * if you want to test a gRPC client ->
      * create the client with a real stub -- via -- [InProcessChannel](../core/src/main/java/io/grpc/inprocess/InProcessChannelBuilder.java),
      * test the client against an [InProcessServer](../core/src/main/java/io/grpc/inprocess/InProcessServerBuilder.java) / a mock/fake service implementation
    * if you want to test a gRPC server ->
      * create the server as an InProcessServer
      * test teh server against a real client stub / InProcessChannel
* gRPC-java library
  * provides
    * [GrpcCleanupRule](../testing/src/main/java/io/grpc/testing/GrpcCleanupRule.java) 
      * := JUnit rule / does the graceful shutdown boilerplate for you

## Even more examples
* third-party examples can be found [here](https://github.com/saturnism/grpc-java-by-example)

## Protocol Buffers
* Check '/src/main/proto' &
  * add `SayHelloAgain`
  * adjust in the corresponding gRPC server and client
* Run again -- using on the previous ways indicated --
* Check the new log messages are displayed!!
