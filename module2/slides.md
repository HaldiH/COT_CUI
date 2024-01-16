---
theme: seriph
background: https://source.unsplash.com/collection/94734566/1920x1080
class: text-center
highlighter: shiki
lineNumbers: false
info: |
  # Module 2: Docker - Operation and Usage
  *Containerization and Orchestration Technologies*
drawings:
  persist: false
transition: slide-left
title: "Module 2: Docker - Operation and Usage"
mdc: true
---

# Module 2: Docker - Operation and Usage

*Containerization and Orchestration Technologies*

---

## Prerequisites

Linux and Mac:
- Install [Git](https://git-scm.com/) on your machine
- [Install Docker Engine](https://docs.docker.com/engine/install/) on your machine

Other:

- Install [WSL2](https://learn.microsoft.com/en-us/windows/wsl/install) if you are on Windows
- Install [Git](https://git-scm.com/) on your machine
- [Install Docker Engine](https://docs.docker.com/engine/install/) on your machine

---

## What is Docker?

Abstraction layer on top of Linux containers. Provides a way to run applications in an isolated environment using a CLI and REST API.

## ... so what are containers?

Not VMs!

... but what are VMs?

Full emulation of a computer system.

---

And what about containers?

A container is a process that runs in an isolated environment. It does not have its own kernel, but it has its own filesystem, network, process tree, etc.

Definition (from OCI spec):

> Define a unit of software delivery called a Standard Container. The goal of a Standard Container is to encapsulate a software component and all its dependencies in a format that is self-describing and portable, so that any compliant runtime can run it without extra dependencies, regardless of the underlying machine and the contents of the container.

---

## Docker Engine components

- dockerd: daemon that runs on the host
- Docker REST API: API used to communicate with the daemon
- Docker CLI: CLI used to communicate with the daemon

---

### Docker daemon (dockerd)

Dockerd is the daemon that runs on the host. It is responsible for managing Docker objects (e.g. images, containers, volumes, networks). It is also responsible for building images, running containers, etc. 

- Dockerd exposes a REST API that can be used to communicate with it.
- Interacts with containerd to manage containers.
- Responsible for building images, running containers, etc.
- Creates networks rules to allow containers to communicate with each other

Dockerd runs as root, so it has access to all the resources on the host. So any user that can access the Docker daemon can gain root access to the host. By default, any user of the group `docker` can access the Docker daemon.

We will use the Docker CLI to communicate with the Docker daemon using the REST API.

---

## Docker resources

- [Images](#images)
- [Containers](#containers)
- [Storage](#storage)
- [Networks](#networks)

---

### Images

#### What is an image?

An image is an immutable file that contains the source code, libraries, dependencies, tools, and other files needed for an application to run. More concretly, an image is a read-only template with instructions for creating a Docker container.

Technically, an image:

- Is a collection of layers
- Can be built from a Dockerfile or imported from a tarball
- Is built on top of a base image

The ultimate base image is `scratch`

#### ... but what are layers?

---

#### Layers

A layer is the files resulting from the execution of a keyword in a Dockerfile.

E.g. if we have the following keywords in a Dockerfile:

```dockerfile
RUN apt-get update
```

The corresponding layer will contain the files resulting from the execution of the `apt-get update` command.

An image can be seen as a set of layers stacked on top of each other, where each layer is also an image. Each layer is a set of files that are added to the image and identified by a hash of its contents. If a layer is changed, the hash changes and the layer is rebuilt, along with all the layers on top of it.

Layers are cached, so if a layer is not changed, it is not rebuilt.

It is important to keep the layers small, so that they can be reused.

---

#### Dockerfile

Script that contains instructions to build an image.

- `FROM`: base image
- `ADD`/`COPY`: add files to the image
- `WORKDIR`: set the working directory for subsequent instructions
- `RUN`: run commands when building the image
- `ENTRYPOINT`: default command to run when the container starts, but cannot be overridden. Only one `ENTRYPOINT` instruction can be used in a Dockerfile.
- `CMD`: default command to run when the container starts. Only one `CMD` instruction can be used in a Dockerfile.
- `EXPOSE`: expose ports

The `RUN` commands are executed when the image is built, while the `CMD`/`ENTRYPOINT` commands are executed when the container is started.

Read more: [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)

---

##### Entrypoint vs CMD

- `ENTRYPOINT` defines the executable to run when the container starts.
- `CMD` defines the default arguments for the executable defined by `ENTRYPOINT`.

```dockerfile
ENTRYPOINT ["bash", "-c"]
CMD ["echo", "Hello World"]
```

Now by defalut, the container will run `bash -c echo "Hello World"`. We can override the default command by passing arguments to the `docker run` command.

```bash
docker run <image_tag:version> echo "Bonjour le monde"
```

#### Building an image

In the directory containing the `Dockerfile`, run the following command:

```bash
docker image build --tag <image_tag:version> .
```

- `--tag/-t` stands for `tag`, which defines the image name and version
  - by default `latest` is used as the version
- `.` defines the path to the context, which is usually the directory containing the `Dockerfile`
- The context defines from where the files will be copied when using `COPY`/`ADD` in the Dockerfile
- To see all options `docker image build --help`

---

#### Registry

The registry is a repository for images. It can be public or private. The default registry is [Docker Hub](https://hub.docker.com/). It is used to store and distribute images. You can use the `docker pull` and `docker push` commands to pull and push images to the registry.

By default, images are pulled from Docker Hub. So if we run `docker pull ubuntu:latest`, it is equivalent to `docker pull docker.io/library/ubuntu:latest`.

To pull an image from a different registry, we need to specify the registry name in the image name. E.g. `registry.example.com/image:tag`.

You can create your own registry using [Docker Registry](https://docs.docker.com/registry/):

Generally, enterprises either use their own private registry to store images, host it on their GitLab instance, or use a third-party service like [AWS ECR](https://aws.amazon.com/ecr/), [Azure Container Registry](https://azure.microsoft.com/en-us/services/container-registry/), [Google Container Registry](https://cloud.google.com/container-registry), etc.

---

### Containers

A Docker container is a running instance of an image. It is an isolated environment that runs a process. It is created from an image and can be started, stopped, restarted, killed, paused, etc.

To create and run a container from an image, run the following command:

```bash
docker run <image_tag:version> [command]
```

- `--name/-n` stands for `name`, which defines the name of the container
- `--detach/-d` runs the container in the background
- `--publish/-p` publishes a container's port to the host
- `--volume/-v` mounts a volume to the container
- `--env/-e` sets an environment variable
- `--rm` removes the container when it is stopped
- `command` can be used to override the default command defined in the Dockerfile
- To see all options `docker run --help`

---

### Storage

By default, any content inside a container is volatile. When a container is removed, all its data is lost. To persist data, we need to use volumes or bind mounts.

- Bind mounts are mapped to a path on the host (e.g. `docker run -v /home/user/data:/data [...]` means that the `/data` directory in the container is mapped to the `/home/user/data` directory on the host)
- Volumes are managed by Docker and are mapped to a path on the host (e.g. `docker run -v data:/data [...]` means that the `/data` directory in the container is mapped to the `data` volume on the host)

A single volume can be shared by multiple containers. Volumes are the preferred way to persist data.

To create a volume, run the following command:

```bash
docker volume create <volume_name>
```

---

### Networks

By default, containers are isolated from each other. To allow containers to communicate, we need to create a network. There are three types of networks: bridge, host, and overlay. All these networks are managed by Docker and present on the host.

One can expose a container to the outside world by publishing a port. This means that the container's port is mapped to a port on the host. E.g. `docker run -p 8080:80 [...]` means that the container's port `80` is mapped to the host's port `8080`.

By doing this, any client on the same network as the host can access the container through the port `8080`.

To create a network, run the following command:

```bash
docker network create [--driver TYPE] <network_name>
```

And to list the networks, run the following command:

```bash
docker network ls
```

---
layout: two-cols-header
---

#### Bridge network

::left::

In Linux, a network bridge allows multiple LAN interfaces to communicate with each other. It is a software bridge that connects two or more network segments together.

Bridge network is the default network. This means that when we create a container, it is added to the bridge network. Any container added to the bridge network can communicate with each other.

However we can create new bridge networks to isolate containers.

As we seen in the first module, when we create a container, we create a virtual pair of interfaces. One interface is added to the bridge network, while the other is added to the container's network namespace.

::right::

![Bridge network](/images/bridge-network.png)

---

#### Host and overlay network

Host network exposes the container directly to the host network. This means that the container is accessible as if it was running natively on the host like any other process. There is no isolation between the container and the host.

E.g. if we run a web server in a container using the host network with port `8080`, any client on the same network as the host can access the web server through the port `8080`. However, we cannot run another container or process using the same port `8080`, since it is already in use by the web server.

Overlay network is used for communication between containers running on different hosts. It is used by Docker Swarm, but we will not cover it in this course.

---

## Docker CLI overview

- `docker exec`: execute a command in a running container
  - i.e. `docker exec -it busybox ip addr` will execute the `ip addr` command in the `busybox` container
- `docker attach`: attach local standard input, output, and error streams to a running container
  - i.e. `docker attach busybox` will attach the terminal to the running process in the `busybox` container
- `docker ps`: list running containers
- `docker images`: list images
- ...

For more information, `docker --help` and `docker COMMAND --help` are your friends.

---

## Logging with Docker

Logs can be used to debug applications and to monitor them.

By default, Docker logs to the standard output and standard error streams from the entrypoint process. This means that we can use the `docker logs` command to view the logs of a container.

```bash
docker logs <container_name>
```

- `--follow/-f` follows the logs: i.e. it prints the logs as they are generated
- `--tail/-n` prints the last `n` lines of the logs: i.e. `docker logs --tail 10 <container_name>` prints the last 10 lines of the logs
- `--timestamps/-t` prints the timestamps of the logs: i.e. `docker logs --timestamps <container_name>` prints the logs with timestamps
- `--since/--until` prints the logs between the specified dates: i.e. `docker logs --since 2021-01-01 <container_name>` prints the logs since 2021-01-01

---

## Optimizing Dockerfiles

### Keep the layers small

As we have seen, Docker images are stacked on top of each other and a final image embed all the lower layers. This means that even if a layer remove a file from a previous layer, the space initially occupied by the file persists in the final image. Thus, it is important to make the layers as small as possible. This means that we should avoid adding unnecessary files to the image.

E.g. if we need to install a package, we should remove the package manager cache after installing the package.

```dockerfile
RUN apt-get update && apt-get install -y <package> && rm -rf /var/lib/apt/lists/*
```

Do not forget to add the `-y` flag to the `apt-get install` command, otherwise the command will wait for user input, and eventually timeout.

---

### Take advantage of the cache

When building an image, Docker will first check if the layer already exists. If it does, it will use it. Otherwise, it will build the layer. So it is pertinent to add the layers that are less likely to change at the top of the Dockerfile because if a lower layer changes, all the layers on top of it will be rebuilt.

E.g. if we have the following Dockerfile:

```dockerfile
FROM ubuntu:latest
COPY . /app
RUN apt-get update && apt-get install -y <package>
```

We should inverse the `COPY` and `RUN` instructions, so that if we change the source code, only the `COPY` instruction will be rebuilt.

---

### Use multi-stage builds

Multi-stage builds allow us to use multiple `FROM` instructions in a Dockerfile. This means that we can use multiple base images in a single Dockerfile. This is useful when we need to build an application, but we do not want to include the build tools in the final image.

A stage is defined by a `FROM` instruction. Each stage can be named using the `AS` keyword. The final image is defined by the last stage.

```dockerfile
FROM <base_image> AS builder
RUN <build_command>

FROM <base_image> AS base
COPY --from=builder <source> <destination>

FROM base AS dev
RUN <dev_command>

FROM base AS prod
RUN <prod_command>
```