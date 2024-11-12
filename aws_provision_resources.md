---
tags: [aws]
---
# Provisioning AWS resources

This note documents a part of Module 3 of [AWS Cloud Practitioner
Essentials](./sources/aws_cloud_practitioner.md).

We can configure, interact, setup, ...(provision) AWS resources either manually
or automatically.

## Manually

We can use 3 ways:

- Management Console -- web UI useful for setting up test environments
- CLI -- useful for scripting & automation
- SKDs -- libraries in various languages that do the heavy lifting

## Automatically

Apart from the manual configuration we can also use 2 semi-automatic ways:

- *AWS Elastic Beanstalk* -- automatically sets up capacity, load balancing,
  automatic scaling, application health monitoring for given code and
  configuration
- *AWS Cloudformation* -- declarative specification that is able to repeatedly
  build up infrastracture from a JSON/YAML file.
