---
tags: [platform]
---

# Docker compose

Docker compose is a separate program, which extends the [`docker`](./docker.md)
command. Docker compose deals with higher-level view where docker images are
combined into bigger architectures. The definition of the architecture is
specified with YAML files, usually called `docker-compose.yaml` (though there is
no rule):

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

## Basic operations

To compile images described in docker-compose yaml files use:

```bash
docker compose build [image_name] # build images from docker-compose files
docker compose up [image_name] # optionally build and run an image
docker-compose down [image_name] # stop containers
```
All containers and images can be controled by standard `docker` command. `docker
compose` is more user-friendly since it associates, `[image_name]` with the id
of the container/image using the `docker-compose.yaml` file in cwd.
