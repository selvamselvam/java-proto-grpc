# proto (gRPC + Protocol Buffers example)

This repository is a small example Java project that uses Protocol Buffers and gRPC.
It contains a simple Greeter service implementation (`HelloWorldServer`) that listens on port 50051 and responds to `SayHello` requests.

Quick highlights
- Java + Gradle project
- Uses the Gradle Protobuf plugin to generate Java sources from `.proto` files
- gRPC server implementation under `src/main/java/org/example/HelloWorldServer.java`

## Table of contents
- Overview
- Prerequisites
- Common tasks (build / generate protos / run / test)
- Project layout
- Development notes
- Troubleshooting
- Contribution and License

## Overview
This project demonstrates how to wire Protocol Buffers and gRPC into a Java project using Gradle and the `com.google.protobuf` plugin. The server implements a `Greeter` service that accepts a `HelloRequest` and returns a `HelloResponse`.

## Prerequisites
- JDK 11 or newer. (Java 17+ recommended.)
- Git (for cloning) and network access to download Gradle dependencies.
- The project includes a Gradle wrapper (`./gradlew`), so you don't need a separate Gradle installation.

Notes / assumptions
- The project `build.gradle` does not explicitly set `sourceCompatibility`; this README assumes a modern JDK (11+). If you must use an older JDK, update `build.gradle` appropriately.

## Build
1. Clean and build (this also triggers protobuf code generation):

```bash
./gradlew clean build
```

This will:
- Download dependencies
- Run the Protobuf plugin to generate Java sources from `src/main/proto` into `build/generated`
- Compile Java sources and produce artifacts in `build/`

If you want to skip tests during development to build faster:

```bash
./gradlew clean build -x test
```

## Generate protobuf sources (explicit)
The Protobuf Gradle plugin integrated in `build.gradle` will run as part of `build`. If you want to explicitly run the generate task:

```bash
./gradlew generateProto
```

Generated Java sources will be placed under `build/generated/source/proto` (and variant-specific subfolders). The Gradle build classpath already includes these generated sources, so a normal `./gradlew build` will compile everything.

## Run the server
The project does not currently declare the `application` plugin or a runnable JAR manifest. Run the server directly from the compiled classes with the JVM classpath. From the project root after a successful build:

```bash
java -cp build/classes/java/main:build/resources/main:build/libs/* org.example.HelloWorldServer
```

This will start the gRPC server and listen on port `50051` (see `HelloWorldServer.java`).

If you'd prefer a convenience `run` task, you can add the Gradle `application` plugin and configure `mainClass` to `org.example.HelloWorldServer`.

## Test the server (using grpcurl)
Install `grpcurl` (https://github.com/fullstorydev/grpcurl) and run the following while the server is running:

```bash
# Example: call SayHello with {"name":"World"}
grpcurl -plaintext -d '{"name":"World"}' localhost:50051 com.example.Greeter/SayHello
```

Expected response (JSON):

```json
{"message":"Hello World"}
```

Notes:
- `-plaintext` is used because the server in this example runs without TLS.
- The fully-qualified service name is `com.example.Greeter` as used in the generated proto classes.

## Project layout
- build.gradle - Gradle build script with protobuf / gRPC plugin configuration
- src/main/proto - .proto files (e.g., `GreeterService.proto`)
- src/main/java - Java server implementation (e.g., `org.example.HelloWorldServer`)
- build/ - build outputs, including generated proto Java classes

## Development notes
- To change the server port, edit `HelloWorldServer.java` and update the `port` variable in `start()`.
- The server uses a fixed thread pool executor (`Executors.newFixedThreadPool(2)`). Adjust according to your CPU/core count and workload.
- The build currently depends on grpc/protobuf versions declared in `build.gradle`. Keep those up to date if you plan to upgrade.

## Troubleshooting
- If you see compile errors referencing generated proto classes (e.g., `com.example.HelloRequest` not found), run `./gradlew clean generateProto build` to force regeneration and compilation.
- If Gradle fails downloading dependencies, check your network/HTTP proxy settings.
- If you get a `NoClassDefFoundError` when running `java -cp ...`, ensure you included the required jars on the classpath (the `build/libs/*` wildcard and `build/classes/java/main` cover the usual cases after a `./gradlew build`).

## Next steps / Suggested improvements
- Add the `application` Gradle plugin and configure a `run` task to simplify running the server.
- Add a proper runnable JAR with a `Main-Class` manifest for easy distribution.
- Add integration tests that start the server and exercise RPC endpoints.

## License
This repository does not include an explicit license file. Add a `LICENSE` file if you intend to publish or share it under a specific license.


---
Generated on: 2025-10-22

