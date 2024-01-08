# Docker Build

* Docker Build is one of Docker Engine's most used features. Whenever you are creating an image you are using Docker
  Build.
* Build is a key part of your software development life cycle allowing you to package and bundle your code and ship it
  anywhere.
* Snd it's not only about packaging your code. It's a whole ecosystem of tools and features that support not only common
  workflow tasks but also provides support for more complex and advanced scenarios.

## 1. Docker Build architecture

* Docker Build implements a client-server architecture, where:
  * Buildx is the client and the user interface for running and managing builds
  * BuildKit is the server, or builder, that handles the build execution.

* As of Docker Engine 23.0 and Docker Desktop 4.19, Buildx is the default build client.

### 1-1. Buildx

* Buildx is a CLI tool that provides a user interface for working with builds.
* In newer versions of Docker Desktop and Docker Engine, you're using Buildx by default when you invoke
  the `docker build` command. In earlier versions, to build using Buildx you would use the `docker buildx build`
  command.
* Buildx is more than just an updated build command. It also contains utilities for creating and managing builders.

### 1-2. Builders

* "Builder" is a term used to describe an instance of a BuildKit backend.
* A builder may run on the same system as the Buildx client, or it may run remotely, on a different system.
* You can run it as a single node, or as a cluster of nodes.
* Builder nodes may be containers, virtual machines, or physical machines.

### 1-3. BuildKit

* BuildKit, or `buildkitd`, is the daemon process that executes the build workloads.
* A build execution starts with the invocation of a docker build command. Buildx interprets your build command and sends
  a build request to the BuildKit backend.
* The build request includes: The Dockerfile, Build arguments, Export options, Caching options

* BuildKit resolves the build instruction and executes the build steps.
* For the duration of the build, Buildx monitors the build status and prints the progress to the terminal.
* BuildKit only requests the resources(e.g., Local filesystem build contexts, Build secrets, SSH sockets, Registry
  authentication tokens) that the build needs, when they're needed. The legacy builder, in comparison, always takes a
  copy of the local filesystem.

## 2. Building images

### 2-1. Packaging your software

#### 2-1-1. Dockerfile

* Docker builds images by reading the instructions from a Dockerfile. A Dockerfile is a text file containing
  instructions for building your source code.
* Most common types of instructions:
  * `FROM <image>`:            Defines a base for your image.
  * `RUN <command>`:        Executes any commands in a new layer on top of the current image and commits the
    result. `RUN` also has a shell form for running commands.
  * `WORKDIR <directory>`:    Sets the working directory for any `RUN`, `CMD`, `ENTRYPOINT`, `COPY`, and `ADD`
    instructions that follow it in the Dockerfile.
  * `COPY <src> <dest>`:    Copies new files or directories from `<src>` and adds them to the filesystem of the
    container at the path `<dest>`.
  * `CMD <command>`:        Lets you define the default program that is run once you start the container based on this
    image. Each Dockerfile only has one `CMD`, and only the last `CMD` instance is respected when multiple exist.

* Filename
  * The default filename to use for a Dockerfile is `Dockerfile`, without a file extension.
  * Some projects may need distinct Dockerfiles for specific purposes.
    * A common convention is to name these `<something>.Dockerfile`. You can specify the Dockerfile filename using
      the `--file` flag for the `docker build` command.

#### 2-1-2. Docker images

* Docker images consist of layers. Each layer is the result of a build instruction in the Dockerfile.
* Example

```
# syntax=docker/dockerfile:1
FROM ubuntu:22.04

# install app dependencies
RUN apt-get update && apt-get install -y python3 python3-pip
RUN pip install flask==2.1.*

# install app
COPY hello.py /

# final configuration
ENV FLASK_APP=hello
EXPOSE 8000
CMD flask run --host 0.0.0.0 --port 8000
```

* Dockerfile syntax
  ```
  # syntax=docker/dockerfile:1
  ```
  * The first line to add to a Dockerfile is a # syntax parser directive. While optional, this directive instructs the
    Docker builder what syntax to use when parsing the Dockerfile, and allows older Docker versions with BuildKit
    enabled to use a specific Dockerfile frontend before starting the build.
  * Parser directives must appear before any other comment, whitespace, or Dockerfile instruction in your Dockerfile,
    and should be the first line in Dockerfiles.
  * We recommend using `docker/dockerfile:1`:
    * which always points to the latest release of the version 1 syntax.
    * BuildKit automatically checks for updates of the syntax before building, making sure you are using the most
      current version.

* Base image
  ```
  FROM ubuntu:22.04
  ```
  * There are many public images you can leverage in your projects, by importing them into your build steps using the
    Dockerfile `FROM` instruction.
  * Docker Hub contains a large set of official images that you can use for this purpose.

* Environment setup
  ```
  # install app dependencies
  RUN apt-get update && apt-get install -y python3 python3-pip
  ```
  * This `RUN` instruction executes a shell in Ubuntu that updates the APT package index and installs Python tools in
    the container.

* Comments
  * Comments are denoted using the same symbol as the syntax directive on the first line of the file.
  * The symbol is only interpreted as a directive if the pattern matches a directive and appears at the beginning of the
    Dockerfile. Otherwise, it's treated as a comment.

* Installing dependencies
  ```
  RUN pip install flask==2.1.*
  ```

* Copying files
  ```
  COPY hello.py /
  ```
  * A build context is the set of files that you can access in Dockerfile instructions such as `COPY` and `ADD`.

* Setting environment variables
  ```
  ENV FLASK_APP=hello
  ```
  * If your application uses environment variables, you can set environment variables in your Docker build using the ENV
    instruction.

* Exposed ports
  ```
  EXPOSE 8000
  ```
  * The `EXPOSE` instruction marks that our final image has a service listening on port 8000.
  * This instruction isn't required, but it is a good practice and helps tools and team members understand what this
    application is doing.

* Starting the application
  ```
  CMD flask run --host 0.0.0.0 --port 8000
  ```
  * Finally, `CMD` instruction sets the command that is run when the user starts a container based on this image.

### 2-2. Build context

* The build context is the set of files that your build can access.
* The positional argument that you pass to the build command specifies the context that you want to use for the build:
  ```
  $ docker build [OPTIONS] PATH | URL | -
                           ^^^^^^^^^^^^^^
  ```
* You can pass any of the following inputs as the context for a build:
  * The relative or absolute path to a local directory
  * A remote URL of a Git repository, tarball, or plain-text file
  * A plain-text file or tarball piped to the docker build command through standard input

* Filesystem contexts
  * When your build context is a local directory, a remote Git repository, or a tar file, then that becomes the set of
    files that the builder can access during the build.
  * Build instructions such as `COPY` and `ADD` can refer to any of the files and directories in the context.
  * A filesystem build context is processed recursively:
    * When you specify a local directory or a tarball, all subdirectories are included
    * When you specify a remote Git repository, the repository and all submodules are included

* Text file contexts
  * When your build context is a plain-text file, the builder interprets the file as a Dockerfile. With this approach,
    the build doesn't use a filesystem context.

#### 2-2-1. Local context

* To use a local build context, you can specify a relative or absolute filepath to the docker build command.
* Local directories
  ```
  .
  ├── index.ts
  ├── src/
  ├── Dockerfile
  ├── package.json
  └── package-lock.json
  ```
  * Dockerfile instructions can reference and include these files in the build if you pass this directory as a context.
  ```
  # syntax=docker/dockerfile:1
  FROM node:latest  
  WORKDIR /src
  COPY package.json package-lock.json .
  RUN npm ci
  COPY index.ts src .
  ```

* Local context with Dockerfile from stdin
  * Use the following syntax to build an image using files on your local filesystem, while using a Dockerfile from
    stdin.
    ```
    $ docker build -f- PATH
    ```
  * The following example uses the current directory (.) as the build context, and builds an image using a Dockerfile
    passed through stdin using a here-document.
    ```
    # create a directory to work in
    mkdir example
    cd example

    # create an example file
    touch somefile.txt

    # build an image using the current directory as context
    # and a Dockerfile passed through stdin
    docker build -t myimage:latest -f- . <<EOF
    FROM busybox
    COPY somefile.txt ./
    RUN cat /somefile.txt
    EOF
    ```
* Local tarballs
  * When you pipe a tarball to the build command, the build uses the contents of the tarball as a filesystem context.
  ```
  .
  ├── Dockerfile
  ├── Makefile
  ├── README.md
  ├── main.c
  ├── scripts
  ├── src
  └── test.Dockerfile
  ```
  * You can create a tarball of the directory and pipe it to the build for use as a context:
  ```
  $ tar czf foo.tar.gz *
  $ docker build - < foo.tar.gz
  ```
  * You can use the --file flag to specify the name and location of the Dockerfile relative to the root of the tarball.
  ```
  $ docker build --file test.Dockerfile - < foo.tar.gz
  ```

#### 2-2-2. Remote context

* ...TBD

### 2-3.Multi-stage builds

* Multi-stage builds are useful to anyone who has struggled to optimize Dockerfiles while keeping them easy to read and
  maintain.
* Use multi-stage builds
  * With multi-stage builds, you use multiple `FROM` statements in your Dockerfile. Each `FROM` instruction can use a
    different base, and each of them begins a new stage of the build.
  * You can selectively copy artifacts from one stage to another, leaving behind everything you don't want in the final
    image.

    ```
    # syntax=docker/dockerfile:1
    FROM golang:1.21
    WORKDIR /src
    COPY <<EOF ./main.go
    package main

    import "fmt"

    func main() {
      fmt.Println("hello, world")
    }
    EOF
    RUN go build -o /bin/hello ./main.go

    FROM scratch
    COPY --from=0 /bin/hello /bin/hello
    CMD ["/bin/hello"]
    ```
  * The second `FROM` instruction starts a new build stage with the scratch image as its base. The `COPY --from=0` line
    copies just the built artifact from the previous stage into this new stage.
  * The Go SDK and any intermediate artifacts are left behind, and not saved in the final image.
  * The build result is a tiny production image with nothing but the binary inside. None of the build tools required to
    build the application are included in the resulting image.

* Name your build stages
  * By default, the stages aren't named, and you refer to them by their integer number, starting with 0 for the
    first `FROM` instruction.
  * However, you can name your stages, by adding an `AS <NAME>` to the `FROM` instruction.
  ```
  # syntax=docker/dockerfile:1
  FROM golang:1.21 as build
  WORKDIR /src
  COPY <<EOF /src/main.go
  package main

  import "fmt"

  func main() {
    fmt.Println("hello, world")
  }
  EOF
  RUN go build -o /bin/hello ./main.go

  FROM scratch
  COPY --from=build /bin/hello /bin/hello
  CMD ["/bin/hello"]
  ```
  * This example improves the previous one by naming the stages and using the name in the `COPY` instruction.
  * This means that even if the instructions in your Dockerfile are re-ordered later, the `COPY` doesn't break.

* Stop at a specific build stage
  * When you build your image, you don't necessarily need to build the entire Dockerfile including every stage. You can
    specify a target build stage.
  * The following command assumes you are using the previous Dockerfile but stops at the stage named build:
    ```
    $ docker build --target build -t hello .
    ```
  * A few scenarios where this might be useful are:
    * Debugging a specific build stage
    * Using a `debug` stage with all debugging symbols or tools enabled, and a lean `production` stage
    * Using a `testing` stage in which your app gets populated with test data, but building for production using a
      different stage which uses real data

* Use an external image as a stage
  * You can use the `COPY --from` instruction to copy from a separate image, either using the local image name, a tag
    available locally or on a Docker registry, or a tag ID.
  * The Docker client pulls the image if necessary and copies the artifact from there. The syntax is:
    ```
    COPY --from=nginx:latest /etc/nginx/nginx.conf /nginx.conf
    ```

* Use a previous stage as a new stage
  * You can pick up where a previous stage left off by referring to it when using the `FROM` directive. For example:
    ```
    # syntax=docker/dockerfile:1

    FROM alpine:latest AS builder
    RUN apk --no-cache add build-base

    FROM builder AS build1
    COPY source1.cpp source.cpp
    RUN g++ -o /binary source.cpp

    FROM builder AS build2
    COPY source2.cpp source.cpp
    RUN g++ -o /binary source.cpp
    ```

* Differences between legacy builder and BuildKit
  * legacy builder: The legacy Docker Engine builder processes all stages of a Dockerfile leading up to the
    selected `--target`. It will build a stage even if the selected target doesn't depend on that stage.
  * BuildKit: BuildKit only builds the stages that the target stage depends on.
    ```
    # syntax=docker/dockerfile:1
    FROM ubuntu AS base
    RUN echo "base"

    FROM base AS stage1
    RUN echo "stage1"

    FROM base AS stage2
    RUN echo "stage2"
    ```
  * With BuildKit enabled, building the `stage2` target in this Dockerfile means only `base` and `stage2` are processed.
  * There is no dependency on `stage1`, so it's skipped.
    ```
    $ DOCKER_BUILDKIT=1 docker build --no-cache -f Dockerfile --target stage2 .
    [+] Building 0.4s (7/7) FINISHED                                                                    
    => [internal] load build definition from Dockerfile                                            0.0s
    => => transferring dockerfile: 36B                                                             0.0s
    => [internal] load .dockerignore                                                               0.0s
    => => transferring context: 2B                                                                 0.0s
    => [internal] load metadata for docker.io/library/ubuntu:latest                                0.0s
    => CACHED [base 1/2] FROM docker.io/library/ubuntu                                             0.0s
    => [base 2/2] RUN echo "base"                                                                  0.1s
    => [stage2 1/1] RUN echo "stage2"                                                              0.2s
    => exporting to image                                                                          0.0s
    => => exporting layers                                                                         0.0s
    => => writing image sha256:f55003b607cef37614f607f0728e6fd4d113a4bf7ef12210da338c716f2cfd15    0.0s
    ```
  * On the other hand, building the same target without BuildKit results in all stages being processed:
    ```
    $ DOCKER_BUILDKIT=0 docker build --no-cache -f Dockerfile --target stage2 .
    Sending build context to Docker daemon  219.1kB
    Step 1/6 : FROM ubuntu AS base
    ---> a7870fd478f4
    Step 2/6 : RUN echo "base"
    ---> Running in e850d0e42eca
    base
    Removing intermediate container e850d0e42eca
    ---> d9f69f23cac8
    Step 3/6 : FROM base AS stage1
    ---> d9f69f23cac8
    Step 4/6 : RUN echo "stage1"
    ---> Running in 758ba6c1a9a3
    stage1
    Removing intermediate container 758ba6c1a9a3
    ---> 396baa55b8c3
    Step 5/6 : FROM base AS stage2
    ---> d9f69f23cac8
    Step 6/6 : RUN echo "stage2"
    ---> Running in bbc025b93175
    stage2
    Removing intermediate container bbc025b93175
    ---> 09fc3770a9c4
    Successfully built 09fc3770a9c4
    ```
