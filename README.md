gRPC-Java - An RPC library and framework
========================================

Supported Platforms
-------------------

* Java
  * 8+=
  * 8- -> use specific branch
    * Check [gRFC P5 JDK Version Support Policy][P5-jdk-version-support]
    
    Java version | gRPC Branch
    ------------ | -----------
    7            | 1.41.x

* Android
  * minSdkVersion 21+= (Lollipop)
    * +21 -- supported with -- [Java 8 language desugaring][android-java-8].
  * if you want TLS -> use Play Services Dynamic Security Provider -- Check [Security Readme](SECURITY.md) --

[android-java-8]: https://developer.android.com/studio/write/java8-support#supported_features
[P5-jdk-version-support]: https://github.com/grpc/proposal/blob/master/P5-jdk-version-support.md#proposal

Getting Started
---------------
* [quick start guide](https://grpc.io/docs/languages/java/quickstart)
  * guided tour
* [gRPC basics](https://grpc.io/docs/languages/java/basics)
  * more explanatory
* Standalone projects / show the use of gRPC
  * [examples](https://github.com/grpc/grpc-java/tree/v1.63.0/examples)
  * [Android example](https://github.com/grpc/grpc-java/tree/v1.63.0/examples/android)

Ways to download
--------
* [JARs][]
* Maven with non-Android
    ```xml
    <dependency>
      <groupId>io.grpc</groupId>
      <artifactId>grpc-netty-shaded</artifactId>
      <version>1.63.0</version>
      <scope>runtime</scope>
    </dependency>
    <dependency>
      <groupId>io.grpc</groupId>
      <artifactId>grpc-protobuf</artifactId>
      <version>1.63.0</version>
    </dependency>
    <dependency>
      <groupId>io.grpc</groupId>
      <artifactId>grpc-stub</artifactId>
      <version>1.63.0</version>
    </dependency>
    <dependency> <!-- necessary for Java 9+ -->
      <groupId>org.apache.tomcat</groupId>
      <artifactId>annotations-api</artifactId>
      <version>6.0.53</version>
      <scope>provided</scope>
    </dependency>
    ```
* Gradle / non-Android
    ```gradle
    runtimeOnly 'io.grpc:grpc-netty-shaded:1.63.0'
    implementation 'io.grpc:grpc-protobuf:1.63.0'
    implementation 'io.grpc:grpc-stub:1.63.0'
    compileOnly 'org.apache.tomcat:annotations-api:6.0.53' // necessary for Java 9+
    ```
* Android client
  * `grpc-okhttp` -- instead of -- `grpc-netty-shaded` and
  * `grpc-protobuf-lite` -- instead of -- `grpc-protobuf`
    ```gradle
    implementation 'io.grpc:grpc-okhttp:1.63.0'
    implementation 'io.grpc:grpc-protobuf-lite:1.63.0'
    implementation 'io.grpc:grpc-stub:1.63.0'
    compileOnly 'org.apache.tomcat:annotations-api:6.0.53' // necessary for Java 9+
    ```
* [Bazel](https://bazel.build)
  * [Maven](https://github.com/bazelbuild/rules_jvm_external) -- (with the GAVs from above) --OR
  * `@io_grpc_grpc_java//api` et al (see below)
* Development snapshots
  * [Sonatypes's snapshot repository](https://oss.sonatype.org/content/repositories/snapshots/)

[JARs]:
https://search.maven.org/search?q=g:io.grpc%20AND%20v:1.63.0

Generated Code
--------------
* For protobuf-based codegen
  * -> put your proto files in
    * `src/main/proto` & 
    * `src/test/proto`
* For protobuf-based codegen + integrated with the Maven build system
  * -> use [protobuf-maven-plugin][]
    * if Eclipse and NetBeans users - > check `os-maven-plugin`'s [IDE documentation](https://github.com/trustin/os-maven-plugin#issues-with-eclipse-m2e-or-other-ides))
    ```xml
    <build>
      <extensions>
        <extension>
          <groupId>kr.motd.maven</groupId>
          <artifactId>os-maven-plugin</artifactId>
          <version>1.7.1</version>
        </extension>
      </extensions>
      <plugins>
        <plugin>
          <groupId>org.xolstice.maven.plugins</groupId>
          <artifactId>protobuf-maven-plugin</artifactId>
          <version>0.6.1</version>
          <configuration>
            <protocArtifact>com.google.protobuf:protoc:3.25.1:exe:${os.detected.classifier}</protocArtifact>
            <pluginId>grpc-java</pluginId>
            <pluginArtifact>io.grpc:protoc-gen-grpc-java:1.63.0:exe:${os.detected.classifier}</pluginArtifact>
          </configuration>
          <executions>
            <execution>
              <goals>
                <goal>compile</goal>
                <goal>compile-custom</goal>
              </goals>
            </execution>
          </executions>
        </plugin>
      </plugins>
    </build>
    ```

[protobuf-maven-plugin]: https://www.xolstice.org/protobuf-maven-plugin/
* For non-Android protobuf-based codegen + integrated with the Gradle build system
  * -> use [protobuf-gradle-plugin][]:
    ```gradle
    plugins {
        id 'com.google.protobuf' version '0.9.4'
    }
    
    protobuf {
      protoc {
        artifact = "com.google.protobuf:protoc:3.25.1"
      }
      plugins {
        grpc {
          artifact = 'io.grpc:protoc-gen-grpc-java:1.63.0'
        }
      }
      generateProtoTasks {
        all()*.plugins {
          grpc {}
        }
      }
    }
    ```
    * prebuilt `protoc-gen-grpc-java` binary uses glibc on Linux
      * if you are compiling on Alpine Linux -> use [Alpine grpc-java package][] -- which uses `musl` instead --

[protobuf-gradle-plugin]: https://github.com/google/protobuf-gradle-plugin

[Alpine grpc-java package]: https://pkgs.alpinelinux.org/package/edge/community/x86_64/grpc-java
* For Android protobuf-based codegen + integrated with the Gradle build system
  * -> use `protobuf-gradle-plugin` / specify the 'lite' options

    ```gradle
    plugins {
        id 'com.google.protobuf' version '0.9.4'
    }
    
    protobuf {
      protoc {
        artifact = "com.google.protobuf:protoc:3.25.1"
      }
      plugins {
        grpc {
          artifact = 'io.grpc:protoc-gen-grpc-java:1.63.0'
        }
      }
      generateProtoTasks {
        all().each { task ->
          task.builtins {
            java { option 'lite' }
          }
          task.plugins {
            grpc { option 'lite' }
          }
        }
      }
    }
    ```
* For [Bazel](https://bazel.build)
  * -> use
    * [`proto_library`](https://github.com/bazelbuild/rules_proto) &
    * [`java_proto_library`](https://bazel.build/reference/be/java#java_proto_library)
      * no `load()` required
    * `load("@io_grpc_grpc_java//:java_grpc_library.bzl", "java_grpc_library")`
      * [Example: `BUILD.bazel`](https://github.com/grpc/grpc-java/blob/master/examples/BUILD.bazel)

API Stability
-------------
* Based on the annotations of the APIs
  * `@Internal`
    * uses
      * internal use by the gRPC library
      * NOT by gRPC users 
  * `@ExperimentalApi`
    * == subject to change in future releases
      * -> NOT depend on them
* [grpc-java-api-checker](https://github.com/grpc/grpc-java-api-checker)
  * := [Error Prone](https://github.com/google/error-prone) plugin / 
    * allows
      * checking usages in any library code / depends on gRPC of
        * `@ExperimentalApi`
        * `@Internal` 
      * checking consumption in non-library code of
        * `@ExperimentalApi`
        * `@Internal`

How to Build
------------
* ⚠️ JUST required, if you are making changes to gRPC-Java ⚠️
* Check [compiling instructions](COMPILING.md)

High-level Components
---------------------
* There are three distinct layers to the library
  * *Stub*
  * *Channel*
  * *Transport*

### Stub
* := layer /
  * exposed to most developers
  * provides
    * type-safe bindings -- to -- whatever datamodel/IDL/interface you are adapting
      * built-in [plugin](https://github.com/google/grpc-java/blob/master/compiler) to the protocol-buffers compiler
        * based on `.proto` files -- generates -- Stub interfaces
      * to other datamodel/IDL are easy and encouraged (❓to build)

### Channel
* := layer /
  * abstraction over Transport handling 
  * vs Stub
    * exposes more behavior to the application
  * uses
    * interception/decoration 
    * application frameworks for cross-cutting concerns -- _Example:_ logging, monitoring, auth, etc. --

### Transport
* := layer / 
  * characteristics
    * their interfaces are abstract just enough -- to allow -- plugging in of different implementations
      * built-in implementations
        * Netty-based HTTP/2 transport
          * based on [Netty](https://netty.io)
          * 👁️ NOT 👁️ officially supported on Android
          * 👁 exist a "grpc-netty-shaded" version 👁
            * 👁 generally preferred 👁
              * Reasons: 🧠less dependency management & easier to upgrade 🧠
        * OkHttp-based HTTP/2 transport
          * := lightweight transport based on
            * [Okio](https://square.github.io/okio/) &
            * low-level parts of [OkHttp](https://square.github.io/okhttp/)
          * uses
            * Android
        * in-process transport
          * uses
            * process with the server == process with the client
            * testing
        * Binder transport
          * uses
            * Android cross-process communication | single device 
    * internal to gRPC
    * vs core API under package `io.grpc`
      * weaker API guarantees
  * in charge of
    * put and take bytes off the wire 
