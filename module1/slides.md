---
theme: default
background: https://source.unsplash.com/collection/94734566/1920x1080
class: text-center
highlighter: shiki
lineNumbers: false
info: |
  # Containerization and Orchestration Technologies
  A course by [Malik Algelly]() and [Hugo Haldi](https://haldih.github.io/) for the Informatic University Center at the University of Geneva.
drawings:
  persist: false
transition: slide-left
title: Containerization and Orchestration Technologies
mdc: true
---

# Containerization and Orchestration Technologies

A course by [Malik Algelly]() and [Hugo Haldi](https://haldih.github.io/) for the Informatic University Center at the University of Geneva.

---

## Contents

- [Containerization in depth](#containerization-in-depth)
- [Docker: Automating the Containerization Process](#docker-automating-the-containerization-process)
- [Orchestration overview](#orchestration-overview)
- [Kubernetes: Automating the Orchestration Process](#kubernetes-automating-the-orchestration-process)
- [Helm Basics](#helm-basics)

---

## What is Containerization?

### VMs vs Containers

#### Virtual Machines

- Virtual Machines (VMs) are an abstraction of physical hardware turning one computer into many computers
- VMs involves a [hypervisor](https://www.vmware.com/topics/glossary/content/hypervisor.html.html) and each VM contains a full copy of an operating system, one or more apps, necessary binaries and libraries

---

## What is Containerization?

### VMs vs Containers

#### Containers

- Containers are an abstraction at the app layer that packages code and dependencies together
- Containerization is a lightweight alternative to full machine virtualization
- Containers are isolated from one another and bundle their own software, libraries and configuration files
- They can communicate with each other through well-defined channels

---

## Why Containerization?

- **Portability**: Containers are highly portable
  - A piece of software that runs on a laptop can run on any other machine that runs Linux
- **Efficiency**: Containers are lightweight
  - Contrary to VMs, containers do not require a hypervisor nor a guest OS
  - Can be efficiently created and destroyed
- **Isolation**: Containers are isolated from each other
  - Containers can only access resources that are explicitly allowed
- **Security**: Containers are secure
  - Since containers are isolated from each other, a compromised container cannot compromise other containers

---

## Why Containerization? (cont.)

- **Scalability**: Containers are scalable
  - Containers can be easily scaled horizontally
  - E.g. if a high load is detected, more containers can be created to handle the load
- **Productivity**: Containers increase developer productivity
  - Containers allow developers to focus on writing code rather than setting up the environment
- **Cost**: Containers reduce costs
  - Containers allow you to run more workloads on the same hardware
- **CI/CD**: Containers are a key enabler of CI/CD
  - Containers allow you to build once and deploy anywhere

---

## How containerization works?

- Containers use a kernel feature called *namespaces* to provide a layer of isolation
- Containers use a kernel feature called *cgroups* to provide resource isolation
- Cgroups allow you to limit the amount of resources a process or group of processes can use and freeze processes

---

## Namespaces

> Namespaces are a feature of the Linux kernel that partitions kernel resources such that one set of processes sees one set of resources while another set of processes sees a different set of resources.
>
> -- <cite>Wikipedia</cite>

In other words, namespaces allow you to create an isolated environment for a process or a group of processes. The goal is to prevent processes from seeing resources that they are not allowed to see. For example, one can put a process in a namespace where it can only see a specific network interface.

Changes made to a resource in a namespace are only visible to processes in that namespace.

---

## Types of Namespaces

- **UTS**: Hostname and domain name
- **PID**: Process IDs
- **Network**: Network devices, stacks, ports, etc.
- **Mount**: Mount points
- **User**: User and group IDs
- ...

---

### UTS Namespace

- UTS stands for Unix Timesharing System
- UTS namespace allows each container to have its own hostname and domain name
- Changes made to the hostname in a UTS namespace are only visible to processes in that namespace

- A child process inherits the UTS namespace of its parent process
- Hostname and domain name are inherited from the parent namespace when a new namespace is created

---
layout: two-cols-header
---

#### In practice

::left::

```bash
# 1. Get the hostname
$ hostname
ms-7917

# 4. Check that the hostname
# has not changed in the initial
# UTS namespace
$ hostname
ms-7917

# 5. Change the hostname in the
# initial UTS namespace
$ hostname yggdrasil
```

::right::

```bash
# 2. Create a new UTS namespace
$ unshare -u
$ hostname
ms-7917

# 3. change the hostname in
# the new UTS namespace
$ hostname thor
$ hostname
thor

# 6. Check that the hostname
# has not changed in the new
# UTS namespace
$ hostname
thor
```

---

### Mount Namespace

#### What is a mount point?

- A mount point is a directory in the filesystem from which the content of a storage device is made available to the user
- E.g. when you plug in a USB drive, by default on Ubuntu the content of the USB drive which device file is `/dev/sdb1` is made available to the user in the `/media/$USER/$NAME_OF_THE_USB_DRIVE` directory
- A mount point source can be a device file, a directory or a remote filesystem

---
layout: two-cols
---

#### In practice

Here, we're mounting the directory `./test1` to the directory `./test2`. In this case, the source is a directory, thus the mount point is called a bind mount (`--bind` option), i.e. we're binding the directory `./test1` to the directory `./test2`.

This means that the directory `./test2` now points to the directory `./test1`.

After the mount operation, any change made to the directory `./test1` will be visible in the directory `./test2` and vice versa.

::right::

```console
$ ls -R ./
./:
test1  test2
./test1:
file1
./test2:

$ sudo mount --bind ./test1 ./test2

$ ls -R ./
./:
test1  test2
./test1:
file1
./test2:
file1

$ touch ./test2/file2

$ ls -R ./
./:
test1  test2
./test1:
file1  file2
./test2:
file1  file2
```

---

### Mount Namespace (cont.)

- Mount namespace allows each container to have its own set of mount points
- Mount namespace is hierarchical, i.e. a child mount namespace copies the mount points of its parent mount namespace
- Mount namespace is the only namespace that is shared between the host and the container

---
layout: two-cols
---

#### In practice

```shell
# 1. List the files in the current directory
$ ls -R ./
./:
test1  test2
./test1:
file1
./test2:

# 3. Create a new mount namespace
$ sudo unshare -m

# 4. Create the bind mount
$ mount --bind ./test1 ./test2

# 5. Check that the bind mount
# has been created
$ ls -R ./
./:
test1  test2
./test1:
file1
./test2:
file1
```

::right::

```shell
# 2. List the files in the current directory
$ ls -R ./
./:
test1  test2
./test1:
file1
./test2:

# 6. Check that the bind mount
# is not visible in other mount
# namespaces
$ ls -R ./
./:
test1  test2
./test1:
file1
./test2:
```

---

### PID Namespace

- PID stands for Process IDentifier
- PID namespace allows each a group of processes to have its own set of PIDs
- Two process can have the same PID in different PID namespaces, however they are not the same process
- The first process in a new PID namespace becomes PID 1 (*init* process)
- PID namespaces are hierarchical, thus they can be seen as parent-child relationships
- A process inside the parent PID namespace can see the processes in the child PID namespace **but not vice versa**

---

### PID Namespace (cont.)

![PID Namespaces](images/pid_namespaces.svg)

*Schema from [this conference by Michael Kerrisk](https://youtu.be/0kJPa-1FuoI)*

---
layout: two-cols-header
---

#### In practice

::left::

First, we create a new PID namespace with the `--fork` option. This means that the new PID namespace will be a child of the current PID namespace.

Since we cannot change the PID namespace of a running process, we need to create a new process in the new PID namespace. This is done with the `--fork` option.

We can see that the PID of the new process is 1, which means that it is the *init* process of the new PID namespace.

However, because of the `/proc` filesystem, the new process can see the processes in the parent PID namespace. We can see that using the `ps` command.

::right::

```shell
$ unshare --pid --fork
$ echo $$
1
$ ps
    PID TTY          TIME CMD
  18165 pts/2    00:00:00 sudo
  18166 pts/2    00:00:00 unshare
  18167 pts/2    00:00:00 bash
  18280 pts/2    00:00:00 ps
```

---

### PID Namespace (cont.)

What should we do if we want to isolate the new process from the parent PID namespace?

---
layout: two-cols-header
---

#### More complete example

::left::

We create both a new PID namespace and a new mount namespace. We then create a new process in the new PID namespace and we mount the `/proc` filesystem in the new mount namespace.

We can see that the new process is the *init* process of the new PID namespace and that it can only see the processes in its new PID namespace.

::right::

```shell
$ unshare --pid --fork --mount
$ mount -t proc proc /proc
$ ps
    PID TTY          TIME CMD
      1 pts/2    00:00:00 bash
     14 pts/2    00:00:00 ps
```

---


### Network Namespace

- Network namespace allows each container to have its own network stack (network interfaces, routing tables, iptables rules, etc.)
- One network interface can only be in one network namespace at a time
- We can use virtual network device pair to provide connectivity between network namespaces
- When a namespace is freed, a physical device is moved back to the initial network namespace while a virtual device is destroyed

---

#### Virtual Network Device Pair

![Virtual Network Device Pair](images/virtual_network_device_pair.svg)

---

#### In practice

```shell
$ ip netns add loki
$ ip link add eth0-l type veth peer name veth-l
$ ip link set eth0-l netns loki
$ ip link set veth-l up
$ ip address add 10.0.0.1/24 dev veth-l
$ ip netns exec loki ip link set lo up
$ ip netns exec loki ip link set eth0-l up
$ ip netns exec loki ip address add 10.0.0.2/24 dev eth0-l
```

---

## User Namespace

- Isolate identifiers (user and group IDs) and capabilities
- UID and GID can be mapped to other UID and GID inside the namespace
- Process can have unprivileged UID and GID outside the namespace and privileged UID and GID inside the namespace
- The process that creates the new namespace gains all capabilities inside the namespace

---

### Hierarchy

- User namespaces have a hierarchical relationship
  - each user namespace has a parent user namespace, except for the initial user namespace
- User namespaces can have multiple children user namespaces
- A maximum of 32 user namespaces can be nested
- A process is a member of exactly one user namespace

---

## User Namespace (cont.)

- A user namespace can own other namespaces (mount, PID, network, etc.)
- Capabilities only apply to the resources that are member of the namespace owned by the current user namespace
- E.g. a process having the `CAP_NET_ADMIN` capability can only modify network interface that are member of the namespace owned by the current user namespace

---

![User Namespaces](images/user_namespaces.svg)

*Schema from [this conference by Michael Kerrisk](https://youtu.be/0kJPa-1FuoI)*
