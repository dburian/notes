---
tags: [platform]
---
# Kubernetes

## Everything is an object
- You specify objects with YAMLs (called **manifests**) and declaratively define
  their attributes

- Kubernetes tries to keep the objects in the specified state after running:

```bash
kubectl apply -f <manifest>.yaml
```

## Manifests 

Each manifest must define:
```yaml
version: v1 # version of Kubernetes API
kind: Pod # what kind of object are we defining
metadata:
  name: nginx # identifier of the object
spec: # specification of the objec
  ...
```

## Pods

Pods are the basic object in Kubernetes, and represent *the smallest unit of
computing managed by Kubernetes*.

Pods can wrap a single [container](./docker.md) or multiple.
