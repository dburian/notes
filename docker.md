---
tags: [platform,ml]
---
# Docker in a nutshell

Docker is an software that isolates running applications in containers. These
containers then can be run anywhere on the docker runtime = docker engine.

#### Server/client architecture

There is a server side and client side. Server side is the runtime (called
docker engine), client side is basically the CLI.

#### Images, containers

Image is a description of an application with its environment. Container is a
live (running) version of an image. So an image can have multiple containers
that have their own state, but the same origin=the image they are running.

#### Docker registries and repositories

Images are stored in a registry that keeps all the version of particular image.
The most widely used is [hub.docker.com](https://hub.docker.com). But major
cloud providers (Amazon, Microsoft, Google, ...) have their own registries which
are usually private.

Registries store different types of images. Each type has an attached tag --
telling us about the version of the image or some other specification. Each type
of an image (=images with the same name) gets one docker *repository*. A
repository then stores all images with the same name, but different tags
(versions).

#### Basic operations


Image can be pulled from the registry:
```bash
docker pull nginx:latest
```

Each time we run an image we create a new container:
```
docker run nginx:latest
```

Running containers can be displayed by:
```bash
docker ps
```

Containers can be stopped by:
```bash
docker stop <container_id>
```

Stopped containers are still stored. The list of all containers can be displayed
by:
```bash
docker ps --all
```

Containers can be then restarted (from the state they stopped at) by:
```bash
docker start <container_id>
```

#### Docker file

Docker files describe an image. Docker files are named `Dockerfile` without any
file extension.

```Dockerfile
FROM node:latest            # Specifies the base image
COPY . /opt/app             # Copies code from the CWD of the CLI to /opt/app inside the container
WORKDIR /opt/app            # Sets the CWD inside the container
RUN npm install             # Runs a command inside the container
CMD ["node", "index.js"]    # Runs the last command of the Dockerfile that should start the dockerized service
```

To get an image from a Dockerfile (which is a mere description) we need to
*build* the image by:
```bash
# In CWD of the Dockerfile
docker build -t <name_of_image>:<tag> .
```

### Docker compose

Docker compose is a separate program that is usually used with docker to speed
ub the process of creating and running multiple docker containers.

Docker compose allows to describe multiple images in one file in YAML format
(the file is usually called `docker-compose.yaml`):

```yaml
version: "3"                # Version of docker
services:                   # Containers to spin up
    my_node:                # Name of an image
        image: node:latest
        port: "8888:80"     # Port forwarding
    proxy:
        image: nginx:latest
        port: 8080
networks:                   # Network to setup between the containers and the host
    ...
```

The command for operating with docker compose is `docker-compose` or `docker
compose`.

To compile images described in docker-compose yaml files use:

```bash
docker-compose build [image_name]
```

To create (or update) containers defined by local `docker-compose.yaml` run:
```bash
docker-compose up [image_name]
```

To stop the stop the set of containers defined by local `docker-compose.yaml`
run:
```bash
docker-compose down [image_name]
```
