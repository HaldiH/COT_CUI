---
theme: seriph
background: https://source.unsplash.com/collection/94734566/1920x1080
class: text-center
highlighter: shiki
lineNumbers: false
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
drawings:
  persist: false
transition: slide-left
title: Welcome to Slidev
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

And what about containers?

A container is a process that runs in an isolated environment. It does not have its own kernel, but it has its own filesystem, network, process tree, etc.

Definition (from OCI spec):

> Define a unit of software delivery called a Standard Container. The goal of a Standard Container is to encapsulate a software component and all its dependencies in a format that is self-describing and portable, so that any compliant runtime can run it without extra dependencies, regardless of the underlying machine and the contents of the container.

---

## Docker Engine components

- dockerd
- Docker REST API
- Docker CLI

---

## Docker resources

---

### Images

- Dockerfile
- How to build an image from a Dockerfile using CLI
- Image tagging
- Layering
- Registry

---

#### Dockerfile

Script that contains instructions to build an image.

- `FROM`: base image
- `ADD`/`COPY`: add files to the image
- `RUN`: run commands
- `CMD`: default command to run when the container starts
- `ENTRYPOINT`: default command to run when the container starts, but cannot be overridden
- `EXPOSE`: expose ports

Read more: [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)

---

### Containers

---

- Running instances of images
- Isolated from the host and other containers
- Can be started, stopped, restarted, killed, paused, etc.
- Produce logs
- We can attach to them to see the logs and run commands
- Container is like a lightweight VM

---

#### Running a container

---

### Volumes

---

### Networks

---

## Docker CLI overview

- List images, containers, volumes, networks
- Build images
- 

---

## Logging with Docker

---

## Optimizing Dockerfiles
