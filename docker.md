---
tags: [platform]
---
# Docker in a nutshell

Docker is an software that isolates running applications in containers. These
containers then can be run anywhere on the docker runtime = docker engine.

## Glossary

### Server/client architecture

There is a server side and client side. Server side is the runtime (called
*docker engine*), client side is basically the CLI.

### Images, containers

*Image* is a description of an application with its environment. *Container* is
a live (running) version of an image. So an image can have multiple containers
that have their own state, but the same origin=the image they are running.

### Dockerfiles

Images get *build* from *Dockerfiles*, which are declarative description how the
images are created.

### Docker registries and repositories

Images are stored in a registry that keeps all the version of particular image.
The most widely used is [hub.docker.com](https://hub.docker.com). But major
cloud providers (Amazon, Microsoft, Google, ...) have their own registries which
are usually private.

Registries store different types of images. Each type has an attached tag --
telling us about the version of the image or some other specification. Each type
of an image (=images with the same name) gets one docker *repository*. A
repository then stores all images with the same name, but different tags
(versions).

## Install

In the [usual installation](https://docs.docker.com/engine/install/), docker
engine is run by root. This means that to run docker commands one needs to have
sudo permissions. Alternatively, the user can be added to group `docker`, which
gives him root permissions and ergo the user doesn't have to write `sudo` all
the time.

To avoid the security risks, docker can be [installed in root-less
mode](https://docs.docker.com/engine/security/rootless/). Using user namespace
remapping of users, docker engine, and all containers can run without root
permissions. At the time of writing I do not know the details. Note however,
there are some
[limitations](https://docs.docker.com/engine/security/rootless/#known-limitations).

## Basic commands

Image can be pulled from the registry:
```bash
docker pull nginx:latest  # pull images from default registry
docker run nginx:latest   # optionally pull and execute a container created from an image

docker build -t <name_of_image>:<tag> . # build image from `./Dockerfile`

docker ps                 # display running containers
docker stop <container_id># stop a container
docker ps --all           # display all containers (even stopped ones)
docker start <container_id> # start a stopped container

docker system prune --volumes  # remove stopped containers and unused volumes
```

## Dockerfile

Docker files are named `Dockerfile` without any file extension.

```Dockerfile
FROM node:latest        # Specifies the base image
COPY . /opt/app         # Copies code from the CWD of the CLI to /opt/app inside the container
WORKDIR /opt/app        # Sets the CWD inside the container
RUN npm install         # Runs a command inside the container
RUN <<EOT cat >> ~/.bashrc # syntax to use heredocs in commands
alias l="ls -lah"
EOT
RUN apt update;\        # you can run multiple commands under single RUN
    apt install package package-b packabe-c
CMD node index.js       # Runs the last command of the Dockerfile that should start the dockerized service
```

## Docker compose

Docker images can be composed into larger architectures using
[docker-compose](./docker_compose.md).

## Docker hub

[Docker hub](https://hub.docker.com/) is the default registry. Its free to
create an account there and have public repositories, but private repositories
(from the second one) or integration with github costs money.

### Basic operations

```bash
docker login # login via browser and get a access token to push repositories
docker push <username>/<image>:<tag> # push local image <image>:<tag> to registry under your <username>
```
