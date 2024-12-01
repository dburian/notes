---
tags: [aws, platform]
---
# AWS Image Builder

Image Builder solves the problem of having to do custom setup on many newly
created [EC2 instance](./aws_computing.md). Image Builder creates a new *Amazon
Machine Images* (*AMI*s), that we can use when creating new EC2 instances.

## Image Pipeline

Each new AMI gets created using an *Image Pipeline*. Image Pipeline encompases
all configuration around AMIs, e.g.:

- how the image is created 
- what infrastructure is used to create them
- how the images are distributed
- ...

When an Image Pipeline is run a new version of an image is created. This can be
done automatically every day/week/month or manually.

## Image Recipe

The part that defines how the image is created is called an *Image Recipe* and
it defines:

- the base image
- the steps that will be carried out on the base image called *Components*

Components are yaml files that define commands that are executed.

## Versioning

Everything is versioned -- components, images. By default a recipe will use the
most up-to-date components.
