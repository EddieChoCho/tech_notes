# Docker Engine

## 1. Storage

### 1-1. Overview - Manage data in Docker

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

### 1-2. Volumes

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





