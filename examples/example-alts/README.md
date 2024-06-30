gRPC ALTS Example
==================
* Goal
  * display HelloWorld client -- communicate securely via authentication by Google's Application Layer Transport Security (ALTS) with -- HelloWorld server
    * [ALTS Whiltepaper](https://cloud.google.com/security/encryption-in-transit/application-layer-transport-security) 
    * `gRPC-client`-backed handshaker is installed in the channel protocol negotiator of
      * ALTS client &
      * ALTS server
    * Once, ALTS Client -- is connected with the -- ALTS Server -> protocol negotiators will trigger the ALTS handshaking process /
      * fires multiple rounds of gRPC communication between multiple parties
        * _Example:_ ALTS client, ALTS server and pre-deployed handshaker server
        * shared secret / encrypt the following RPC calls
* Ecosystem
  * GCP environment
  * via pre-deployed handshaker service -> valid in any environments

### How to build?
* If you want non-released versions -> [Install gRPC Java library SNAPSHOT locally, including code generation plugin](../../COMPILING.md)
* In this directory:
    ```
    $ ../gradlew installDist
    ```
    * Problems:
      * Problem1: " Could not find io.grpc:grpc-alts:1.64.0-SNAPSHOT"
        * Solution: Adjust to 'io.grpc:grpc-alts:1.64.0'
  * It creates the scripts `hello-world-alts-server`, `hello-world-alts-client` | `build/install/example-atls/bin/` -- directory to run it --

### How to run in a GCP environment
* separate handshaker service is available in the GCP environment
* run the server
    ```bash
    # Run the server:
    ./build/install/example-alts/bin/hello-world-alts-server
    ```
* run the client, in another terminal
    ```
    ./build/install/example-alts/bin/hello-world-alts-client
    ```

### How to test in a non-GCP environment?
* Deploy a [handshaker service](https://github.com/grpc/grpc/blob/7e367da22a137e2e7caeae8342c239a91434ba50/src/proto/grpc/gcp/handshaker.proto#L224-L234) 
* Configure both the -- TODO: Try --
  * [ALTS client](https://github.com/grpc/grpc-java/blob/master/alts/src/main/java/io/grpc/alts/AltsChannelBuilder.java#L63-L76)
  * [ALTS server](https://github.com/grpc/grpc-java/blob/master/alts/src/main/java/io/grpc/alts/AltsServerCredentials.java#L55-L72)
  * [Example](https://github.com/grpc/grpc-java/blob/master/interop-testing/src/test/java/io/grpc/testing/integration/AltsHandshakerTest.java#L45)

## References
* [grpc.io tutorial](https://grpc.io/docs/languages/java/alts/)
