# Docker Engine

## 1. Storage

### [1-1. Overview - Manage data in Docker](https://docs.docker.com/storage/)

* Docker has two options for containers to store files on the host machine, so that the files are persisted even after
  the container stops: volumes, and bind mounts.
* Docker also supports containers storing files in-memory on the host machine. Such files are not persisted. tmpfs
  mount(Linux)/named pipe(for Windows) is used to store files in the host's system memory.
* Recommend to use the `--mount` flag for both containers and services, for bind mounts, volumes, or tmpfs mounts, as
  the syntax is more clear.

#### 1-1-1. Choose the right type of mount

* Volumes are stored in a part of the host filesystem which is managed by Docker (/var/lib/docker/volumes/ on Linux).
  Non-Docker processes should not modify this part of the filesystem. Volumes are the best way to persist data in
  Docker.
  * You can create a volume explicitly using the `docker volume create` command, or Docker can create a volume during
    container or service creation.
  * A given volume can be mounted into multiple containers simultaneously. When no running container is using a volume,
    the volume is still available to Docker and is not removed automatically.
  * You can remove unused volumes using `docker volume prune`.
  * Volumes also support the use of volume drivers, which allow you to store your data on remote hosts or cloud
    providers, among other possibilities.

* Bind mounts may be stored anywhere on the host system. They may even be important system files or directories.
  Non-Docker processes on the Docker host or a Docker container can modify them at any time.
  * Bind mounts have limited functionality compared to volumes.
  * When you use a bind mount, a file or directory on the host machine is mounted into a container.
  * The file or directory doesn't need to exist on the Docker host already. It is created on demand if it does not yet
    exist.

* `tmpfs`/`npipe` mounts are stored in the host system's memory only, and are never written to the host system's
  filesystem.

#### 1-1-2. Good use cases for volumes

* Sharing data among multiple running containers.
  * When that container stops or is removed, the volume still exists.
  * Multiple containers can mount the same volume simultaneously, either read-write or read-only.
  * Volumes are only removed when you explicitly remove them.
* When the Docker host is not guaranteed to have a given directory or file structure.
  * Volumes help you decouple the configuration of the Docker host from the container runtime.
* When you want to store your container's data on a remote host or a cloud provider, rather than locally.
* When you need to back up, restore, or migrate data from one Docker host to another, volumes are a better choice. You
  can stop containers using the volume, then back up the volume's directory (such as /var/lib/docker/volumes/<
  volume-name>).
* When your application requires high-performance I/O on Docker Desktop.
  * Volumes are stored in the Linux VM rather than the host, which means that the reads and writes have much lower
    latency and higher throughput.
* When your application requires fully native file system behavior on Docker Desktop.
  * For example, a database engine requires precise control over disk flushing to guarantee transaction durability.
  * Volumes are stored in the Linux VM and can make these guarantees, whereas bind mounts are remoted to macOS or
    Windows, where the file systems behave slightly differently.

#### 1-1-3. Good use cases for bind mounts

* Sharing configuration files from the host machine to containers.
  * This is how Docker provides DNS resolution to containers by default, by mounting /etc/resolv.conf from the host
    machine into each container.
* Sharing source code or build artifacts between a development environment on the Docker host and a container.
  * For instance, you may mount a Maven target/ directory into a container, and each time you build the Maven project on
    the Docker host, the container gets access to the rebuilt artifacts.
  * If you use Docker for development this way, your production Dockerfile would copy the production-ready artifacts
    directly into the image, rather than relying on a bind mount.(?)
* When the file or directory structure of the Docker host is guaranteed to be consistent with the bind mounts the
  containers require.

#### 1-1-4. Good use cases for tmpfs mounts

* When you do not want the data to persist either on the host machine or within the container.
  * This may be for security reasons or to protect the performance of the container when your application needs to write
    a large volume of non-persistent state data.

#### 1-1-5. Tips for using bind mounts or volumes

* If you mount an `empty volume` into a directory in the container in which files or directories exist, these files or
  directories are propagated (copied) into the volume.
  * Similarly, if you start a container and specify a volume which does not already exist, an empty volume is created
    for you.
  * This is a good way to pre-populate data that another container needs.

* If you mount a `bind mount` or `non-empty volume` into a directory in the container in which some files or directories
  exist, these files or directories are obscured by the mount, just as if you saved files into /mnt on a Linux host and
  then mounted a USB drive into /mnt.
  * The contents of `/mnt` would be obscured by the contents of the USB drive until the USB drive were unmounted.
  * The obscured files are not removed or altered, but are not accessible while the bind mount or volume is mounted.

### [1-2. Volumes](https://docs.docker.com/storage/volumes/)

* Volumes v.s. Bind Mounts
  * Volumes are the preferred mechanism for persisting data generated by and used by Docker containers.
  * While bind mounts are dependent on the directory structure and OS of the host machine, volumes are completely
    managed by Docker.
  * Volumes have several advantages over bind mounts:
    * ...
    * You can manage volumes using Docker CLI commands or the Docker API.
    * Volumes can be more safely shared among multiple containers.
    * Volume drivers let you store volumes on remote hosts or cloud providers, encrypt the contents of volumes, or add
      other functionality.

#### 1-2-1. Choose the -v or --mount flag

* In general, --mount is more explicit and verbose. And volumes used with services, only support `--mount`.

* -v or --volume: Consists of three fields, separated by colon characters (:). The fields must be in the correct order.
  * In the case of named volumes, the first field is the name of the volume, and is unique on a given host machine. For
    anonymous volumes, the first field is omitted.
  * The second field is the path where the file or directory are mounted in the container.
  * The third field is optional, and is a comma-separated list of options.

* --mount: Consists of multiple key-value pairs, separated by commas and each consisting of a <key>=<value> tuple.
  * The `type` of the mount, which can be `bind`, `volume`, or `tmpfs`. This topic discusses volumes, so the type is
    always volume.
  * The `source` of the mount. For named volumes, this is the name of the volume. For anonymous volumes, this field is
    omitted. Can be specified as `source` or `src`.
  * The `destination` takes as its value the path where the file or directory is mounted in the container. Can be
    specified as `destination`, `dst`, or `target`.
  * The `readonly` option, if present, causes the bind mount to be mounted into the container as read-only. Can be
    specified as `readonly` or `ro`.
  * The `volume-opt` option, which can be specified more than once, takes a key-value pair consisting of the option name
    and its value.

#### 1-2-2. Create and manage volumes

```
$ docker volume create my-vol
$ docker volume ls
local               my-vol
$ docker volume inspect my-vol
[
    {
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/my-vol/_data",
        "Name": "my-vol",
        "Options": {},
        "Scope": "local"
    }
]
$ docker volume rm my-vol
```

#### 1-2-3. Start a container with a volume

* If you start a container with a volume that doesn't yet exist, Docker creates the volume for you.
* The following example mounts the volume myvol2 into /app/ in the container:

```
$ docker run -d \
  --name devtest \
  --mount source=myvol2,target=/app \
  nginx:latest
```

* Use docker inspect devtest to verify that Docker created the volume and it mounted correctly. Look for the Mounts
  section:

```
"Mounts": [
    {
        "Type": "volume",
        "Name": "myvol2",
        "Source": "/var/lib/docker/volumes/myvol2/_data",
        "Destination": "/app",
        "Driver": "local",
        "Mode": "",
        "RW": true,  //read-write.
        "Propagation": ""
    }
],
```

* Stop the container and remove the volume. Note volume removal is a separate step.

```
$ docker container stop devtest
$ docker container rm devtest
$ docker volume rm myvol2
```

#### 1-2-4. Use a volume with Docker Compose

```
services:
  frontend:
    image: node:lts
    volumes:
      - myapp:/home/node/app
# Running docker compose up for the first time creates a volume. Docker reuses the same volume when you run the command subsequently.
volumes:
  myapp:

# You can create a volume directly outside of Compose  
# volumes:
#   myapp:
#     external: true
```

* Start a service with volumes

```
# The docker service create command doesn't support the -v or --volume flag. 
# When mounting a volume into a service's containers, you must use the --mount flag.

$ docker service create -d \
  --replicas=4 \
  --name devtest-service \
  --mount source=myvol2,target=/app \
  nginx:latest

$ docker service ps devtest-service
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
4d7oz1j85wwn        devtest-service.1   nginx:latest        moby                Running

$ docker service rm devtest-service
# Removing the service doesn't remove any volumes created by the service. Volume removal is a separate step.  
```

* Populate a volume using a container
  * If you start a container which creates a new volume, and the container has files or directories in the directory to
    be mounted such as /app/, Docker copies the directory's contents into the volume. The container then mounts and uses
    the volume, and other containers which use the volume also have access to the pre-populated content.

#### 1-2-5. Share data between machines

* ...TBD

#### 1-2-6. Back up, restore, or migrate data volumes

* Back up a volume

```
$ docker run -v /dbdata --name dbstore ubuntu /bin/bash

# Launch a new container and mount the volume from the dbstore container
# Mount a local host directory as /backup
# Pass a command that tars the contents of the dbdata volume to a backup.tar file inside our /backup directory.
$ docker run --rm --volumes-from dbstore -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar /dbdata

# When the command completes and the container stops, it creates a backup of the dbdata volume.
```

* Restore volume from a backup

```
$ docker run -v /dbdata --name dbstore2 ubuntu /bin/bash
$ docker run --rm --volumes-from dbstore2 -v $(pwd):/backup ubuntu bash -c "cd /dbdata && tar xvf /backup/backup.tar --strip 1"
```

* Remove volumes
  * A Docker data volume persists after you delete a container. There are two types of volumes to consider:
  * Named volumes have a specific source from outside the container, for example, awesome:/bar.
  * Anonymous volumes have no specific source. Therefore, when the container is deleted, you can instruct the Docker
    Engine daemon to remove them.

```
# Remove anonymous volumes:
## To automatically remove anonymous volumes, use the --rm option. 
$ docker run --rm -v /foo -v awesome:/bar busybox top
## This command creates an anonymous /foo volume. 
## When you remove the container, the Docker Engine removes the /foo volume but not the awesome volume.

# Remove all unused volumes:
$ docker volume prune
```

### [1-3. Bind mounts](https://docs.docker.com/storage/bind-mounts/)

* The file or directory does not need to exist on the Docker host already. It is created on demand if it does not yet
  exist.
* Bind mounts are very performant, but they rely on the host machine's filesystem having a specific directory structure
  available.

#### 1-3-1. Choose the -v or --mount flag

* Differences between `-v` and `--mount` behavior
  * If you use `-v` or `--volume` to bind-mount a file or directory that does not yet exist on the Docker host, `-v`
    creates the endpoint for you. It is always created as a directory.
  * If you use `--mount` to bind-mount a file or directory that does not yet exist on the Docker host, Docker does not
    automatically create it for you, but generates an error.

#### 1-3-2. Start a container with a bind mount

```
$ docker run -d \
  -it \
  --name devtest \
  --mount type=bind,source="$(pwd)"/target,target=/app \
  nginx:latest
```

* Use `docker inspect devtest` to verify that the bind mount was created correctly. Look for the Mounts section:

```
"Mounts": [
    {
        "Type": "bind",
        "Source": "/tmp/source/target",
        "Destination": "/app",
        "Mode": "",
        "RW": true,
        "Propagation": "rprivate"
    }
],
```

#### 1-3-3. Mount into a non-empty directory on the container

* If you bind-mount a directory into a non-empty directory on the container, the directory's existing contents are
  obscured by the bind mount.

```
$ docker run -d \
  -it \
  --name broken-container \
  --mount type=bind,source=/tmp,target=/usr \
  nginx:latest

docker: Error response from daemon: oci runtime error: container_linux.go:262:
starting container process caused "exec: \"nginx\": executable file not found in $PATH".
```

#### 1-3-4. Use a read-only bind mount

```
$ docker run -d \
  -it \
  --name devtest \
  --mount type=bind,source="$(pwd)"/target,target=/app,readonly \
  nginx:latest

"Mounts": [
    {
        "Type": "bind",
        "Source": "/tmp/source/target",
        "Destination": "/app",
        "Mode": "ro",
        "RW": false,  //readonly
        "Propagation": "rprivate"
    }
],
```

#### 1-3-5. Configure bind propagation

* ...TBD

#### 1-3-6. Configure the selinux label

* ...TBD

#### 1-3-7. Use a bind mount with compose

```
services:
  frontend:
    image: node:lts
    volumes:
      - type: bind
        source: ./static
        target: /opt/app/static
volumes:
  myapp:
```

### 1-4. tmpfs mounts

* ...TBD

### [1-5. Storage drivers](https://docs.docker.com/storage/storagedriver/)

#### 1-5-1. About storage drivers

##### 1-5-1-1. Storage drivers versus Docker volumes

* Storage drivers:
  * Docker uses storage drivers to store image layers, and to store data in the writable layer of a container.
  * The container's writable layer does not persist after the container is deleted, but is suitable for storing
    ephemeral data that is generated at runtime.
  * Storage drivers are optimized for space efficiency, but (depending on the storage driver) write speeds are lower
    than native file system performance, especially for storage drivers that use a copy-on-write filesystem.
    * Write-intensive applications, such as database storage, are impacted by a performance overhead, particularly if
      pre-existing data exists in the read-only layer.

* Docker volumes:
  * Use Docker volumes for write-intensive data, data that must persist beyond the container's lifespan, and data that
    must be shared between containers.

##### 1-5-1-2. Images and layers

* A Docker image is built up from a series of layers.
* Each layer represents an instruction in the image's Dockerfile.
* Each layer except the very last one is read-only.
* e.g.,

```
# This Dockerfile contains four commands(FROM, LABEL, COPY, RUN). Commands that modify the filesystem create a layer. 

FROM ubuntu:22.04
# The FROM statement starts out by creating a layer from the ubuntu:22.04 image. 

LABEL org.opencontainers.image.authors="org@example.com"
# The LABEL command only modifies the image's metadata, and doesn't produce a new layer. 

COPY . /app
#  The COPY command adds some files from your Docker client's current directory. 

RUN make /app
# The first RUN command builds your application using the make command, and writes the result to a new layer. 

RUN rm -r $HOME/.cache
# The second RUN command removes a cache directory, and writes the result to a new layer. 

CMD python /app/app.py
# The CMD instruction specifies what command to run within the container, which only modifies the image's metadata, which doesn't produce an image layer.
```

* Each layer is only a set of differences from the layer before it.
* Note that both adding, and removing files will result in a new layer. In the example above, the $HOME/.cache directory
  is removed, but will still be available in the previous layer and add up to the image's total size.
* The layers are stacked on top of each other.
* When you create a new container, you add a new writable layer on top of the underlying layers. This layer is often
  called the "container layer". All changes made to the running container, such as writing new files, modifying existing
  files, and deleting files, are written to this thin writable container layer.

##### 1-5-1-3. Container and layers

* The major difference between a container and an image is the top writable layer.
  * All writes to the container that add new or modify existing data are stored in this writable layer.
  * When the container is deleted, the writable layer is also deleted. The underlying image remains unchanged.

* Because each container has its own writable container layer, and all changes are stored in this container layer,
  multiple containers can share access to the same underlying image and yet have their own data state.

* Docker uses storage drivers to manage the contents of the image layers and the writable container layer.
  * Each storage driver handles the implementation differently, but all drivers use stackable image layers and the
    copy-on-write (CoW) strategy.

##### 1-5-1-4. Container size on disk

* `docker ps -s`:
  * `size`: the amount of data (on disk) that's used for the writable layer of each container.
  * `virtual size`: the amount of data used for the read-only image data used by the container plus the container's
    writable layer size.
    * Multiple containers may share some or all read-only image data. Two containers started from the same image share
      100% of the read-only data, while two containers with different images which have layers in common share those
      common layers.
    * Therefore, you can't just total the virtual sizes. This over-estimates the total disk usage by a potentially
      non-trivial amount.

* This also doesn't count the following additional ways a container can take up disk space:
  * Disk space used for log files stored by the logging-driver. This can be non-trivial if your container generates a
    large amount of logging data and log rotation isn't configured.
  * Volumes and bind mounts used by the container.
  * Disk space used for the container's configuration files, which are typically small.
  * Memory written to disk (if swapping is enabled).
  * Checkpoints, if you're using the experimental checkpoint/restore feature.

##### 1-5-1-5. The copy-on-write (CoW) strategy

* Copy-on-write is a strategy of sharing and copying files for maximum efficiency. If a file or directory exists in a
  lower layer within the image, and another layer (including the writable layer) needs read access to it, it just uses
  the existing file.
  * The first time another layer needs to modify the file (when building the image or running the container), the file
    is copied into that layer and modified. This minimizes I/O and the size of each of the subsequent layers.

* Sharing promotes smaller images
  * When you use docker pull to pull down an image from a repository, or when you create a container from an image that
    doesn't yet exist locally, each layer is pulled down separately, and stored in Docker's local storage area(
    /var/lib/docker/ on Linux hosts).
  * Now imagine that you have two different Dockerfiles. You use the first one to create an image
    called `acme/my-base-image:1.0`.
  ```
  FROM alpine
  RUN apk add --no-cache bash
  ```
  * The second one is based on `acme/my-base-image:1.0`, but has some additional layers:
  ```
  FROM acme/my-base-image:1.0
  COPY . /app
  RUN chmod +x /app/hello.sh
  CMD /app/hello.sh
  ```
  * The second image contains all the layers from the first image, plus new layers created by the `COPY` and `RUN`
    instructions, and a read-write container layer.
  * Docker already has all the layers from the first image, so it doesn't need to pull them again. The two images share
    any layers they have in common.

  * If you build images from the two Dockerfiles, you can use `docker image ls` and `docker image history` commands to
    verify that the cryptographic IDs of the shared layers are the same.
  ```
  $ docker image history acme/my-base-image:1.0

  IMAGE          CREATED         CREATED BY                                      SIZE      COMMENT
  da3cf8df55ee   5 minutes ago   RUN /bin/sh -c apk add --no-cache bash # bui…   2.15MB    buildkit.dockerfile.v0
  <missing>      7 weeks ago     /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B
  <missing>      7 weeks ago     /bin/sh -c #(nop) ADD file:f278386b0cef68136…   5.6MB

  $ docker image history  acme/my-final-image:1.0

  IMAGE          CREATED         CREATED BY                                      SIZE      COMMENT
  8bd85c42fa7f   3 minutes ago   CMD ["/bin/sh" "-c" "/app/hello.sh"]            0B        buildkit.dockerfile.v0
  <missing>      3 minutes ago   RUN /bin/sh -c chmod +x /app/hello.sh # buil…   39B       buildkit.dockerfile.v0
  <missing>      3 minutes ago   COPY . /app # buildkit                          222B      buildkit.dockerfile.v0
  <missing>      4 minutes ago   RUN /bin/sh -c apk add --no-cache bash # bui…   2.15MB    buildkit.dockerfile.v0
  <missing>      7 weeks ago     /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B
  <missing>      7 weeks ago     /bin/sh -c #(nop) ADD file:f278386b0cef68136…   5.6MB
  
  # Notice that all steps of the first image are also included in the final image. The final image includes the two layers from the first image, and two layers that were added in the second image.
  # The <missing> lines in the docker history output indicate that those steps were either built on another system and part of the alpine image that was pulled from Docker Hub, or were built with BuildKit as builder. 
  ```
  * Use the `docker image inspect` command to view the cryptographic IDs of the layers in each image:
  ```
  $ docker image inspect --format "{{json .RootFS.Layers}}" acme/my-base-image:1.0
  [
    "sha256:72e830a4dff5f0d5225cdc0a320e85ab1ce06ea5673acfe8d83a7645cbd0e9cf",
    "sha256:07b4a9068b6af337e8b8f1f1dae3dd14185b2c0003a9a1f0a6fd2587495b204a"
  ]
  
  $ docker image inspect --format "{{json .RootFS.Layers}}" acme/my-final-image:1.0
  [
    "sha256:72e830a4dff5f0d5225cdc0a320e85ab1ce06ea5673acfe8d83a7645cbd0e9cf",
    "sha256:07b4a9068b6af337e8b8f1f1dae3dd14185b2c0003a9a1f0a6fd2587495b204a",
    "sha256:cc644054967e516db4689b5282ee98e4bc4b11ea2255c9630309f559ab96562e",
    "sha256:e84fb818852626e89a09f5143dbc31fe7f0e0a6a24cd8d2eb68062b904337af4"
  ]
  
  # Notice that the first two layers are identical in both images. The second image adds two additional layers. 
  # Shared image layers are only stored once in /var/lib/docker/ and are also shared when pushing and pulling an image to an image registry. 
  # Shared image layers can therefore reduce network bandwidth and storage.
  ```

* Copying makes containers efficient
  * When you start a container, a thin writable container layer is added on top of the other layers. Any changes the
    container makes to the filesystem are stored here.
  * Any files the container doesn't change don't get copied to this writable layer. This means that the writable layer
    is as small as possible.

  * When an existing file in a container is modified, the storage driver performs a copy-on-write operation. The
    specifics steps involved depend on the specific storage driver.
  * For the overlay2 driver, the copy-on-write operation follows this rough sequence:
    * Search through the image layers for the file to update. The process starts at the newest layer and works down to
      the base layer one layer at a time. When results are found, they're added to a cache to speed future operations.
    * Perform a `copy_up` operation on the first copy of the file that's found, to copy the file to the container's
      writable layer.
    * Any modifications are made to this copy of the file, and the container can't see the read-only copy of the file
      that exists in the lower layer.

  * Containers that write a lot of data consume more space than containers that don't. This is because most write
    operations consume new space in the container's thin writable top layer.
  * Note that changing the metadata of files, for example, changing file permissions or ownership of a file, can also
    result in a copy_up operation, therefore duplicating the file to the writable layer.
    * Tip: Use volumes for write-heavy applications.

  * A `copy_up` operation can incur a noticeable performance overhead.
  * This overhead is different depending on which storage driver is in use. Large files, lots of layers, and deep
    directory trees can make the impact more noticeable.
  * This is mitigated by the fact that each copy_up operation only occurs the first time a given file is modified.

## 2. Networking

### [2-1. Networking overview](https://docs.docker.com/network/)

* A container has no information about what kind of network it's attached to, or whether their peers are also Docker
  workloads or not.
* A container only sees a network interface with an IP address, a gateway, a routing table, DNS services, and other
  networking details. That is, unless the container uses the `none` network driver.

#### 2-1-1. User-defined networks

* You can create custom, user-defined networks, and connect multiple containers to the same network.
* Once connected to a user-defined network, containers can communicate with each other using container IP addresses or
  container names.
* e.g.,

```
$ docker network create -d bridge my-net
# Creates a network using the bridge network driver

$ docker run --network=my-net -itd --name=container3 busybox
# running a container in the created network
```

* There are multiple network drivers. e.g., bridge, host, none, overlay, ipvlan, maclan ...etc.

#### 2-1-2. Container networks

* You can attach a container to another container's networking stack directly, using the `--network container:<name|id>`
  flag format.
* e.g.,

```
$ docker run -d --name redis example/redis --bind 127.0.0.1
# Runs a Redis container, with Redis binding to localhost

$ docker run --rm -it --network container:redis example/redis-cli -h 127.0.0.1
# Runs the redis-cli command and connecting to the Redis server over the localhost interface.
```

#### 2-1-3. Published ports

* Use the `--publish` or `-p` flag to make a port available to services outside of Docker. This creates a firewall rule
  in the host, mapping a container port to a port on the Docker host to the outside world.
* e.g., `-p 8080:80`: Map port 8080 on the Docker host to TCP port 80 in the container.

* If you want to make a container accessible to other containers, it isn't necessary to publish the container's ports.
  * You can enable inter-container communication by connecting the containers to the same network, usually a `bridge`
    network.

* Important
  * Publishing container ports is insecure by default. Meaning, when you publish a container's ports it becomes
    available not only to the Docker host, but to the outside world as well.
  * If you include the localhost IP address (127.0.0.1) with the publish flag, only the Docker host can access the
    published container port. e.g., `-p 127.0.0.1:8080:80`
  * Warning
    * Hosts within the same L2 segment (for example, hosts connected to the same network switch) can reach ports
      published to localhost.

* IP address and hostname
  * When a container starts, it can only attach to a single network, using the `--network` flag.
  * You can connect a running container to additional networks using the `docker network connect` command.
  * A container's hostname defaults to be the container's ID in Docker. You can override the hostname
    using `--hostname`.
  * When connecting to an existing network using docker network connect, you can use the `--alias` flag to specify an
    additional network alias for the container on that network.

* DNS services
  * Containers use the same DNS servers as the host by default, but you can override this with `--dns`.
  * By default, containers inherit the DNS settings as defined in the /etc/resolv.conf configuration file. Containers
    that attach to the default bridge network receive a copy of this file.
  * Containers that attach to a custom network use Docker's embedded DNS server. The embedded DNS server forwards
    external DNS lookups to the DNS servers configured on the host.
  * Using these flags, you can configure DNS resolution on a per-container
    basis: `--dns`, `--dns-search`, `--dns-opt`, `--hostname`

* Nameservers with IPv6 addresses
  * If the /etc/resolv.conf file on the host system contains one or more nameserver entries with an IPv6 address, those
    nameserver entries get copied over to /etc/resolv.conf in containers that you run.
  * For containers using musl libc (in other words, Alpine Linux), this results in a race condition for hostname lookup.
    * As a result, hostname resolution might sporadically fail if the external IPv6 DNS server wins the race condition
      against the embedded DNS server.
  * It's rare that the external DNS server is faster than the embedded one.
    * But things like garbage collection, or large numbers of concurrent DNS requests, can sometimes result in a round
      trip to the external server being faster than local resolution.

* Custom hosts
  * Your container will have lines in /etc/hosts which define the hostname of the container itself, as well as localhost
    and a few other common things.
  * Custom hosts, defined in /etc/hosts on the host machine, aren't inherited by containers.

### [2-2. Network drivers](https://docs.docker.com/network/drivers/)

* ...TBD




