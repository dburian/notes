---
tags: [aws]
---
# AWS Computing

This note documents the contents of Module 2 of [AWS Cloud Practitioner
Essentials](./sources/aws_cloud_practitioner.md).

Computing in Amazon Web Services is done using Elastic Compute Cloud Service
(*EC2*). Computing on EC2 is done through *EC2 instances*, which are just
virtual servers.

## Instance types

There are many instance types ([full
list](https://aws.amazon.com/ec2/instance-types/)), which are classified to
*instance families*. Each family is for different purpose:

- *general purpose* -- balance of compute, memory, networking and storage (e.g.
  plain web server)
- *compute optimized* -- for compute-bound applications (e.g. web server with
  intensive back end), provides performant CPUs
- *memory optimized* -- for memory intensive tasks (e.g. low-latency database)
- *accelerated computing* -- for compute-bound applications requiring
  accelerators (e.g. deep learning), provides GPUs
- *storage optimized* -- for tasks requiring large storage (e.g. large
  databases)

## Scaling

EC2 instances can be scaled horizontally or vertically. Horizontal scaling can
be done with *EC2 Auto Scaling*, which enables to configure an *Auto Scaling
group* which scales the number of EC2 instances based on current workload. You
can set:

- *minimum* number of instances -- always running
- *desired* number of instances -- if set, Auto Scaling Group will do its best
  to match it (basically the same as minimum?)
- *maximum* number of instances -- ceiling of horizontal scaling

### Balancer

To establish one point of contact (e.g. between APIs) while having horizontally
scalable compute we need a load balancer, that redistributes requests between
multiple endpoints. In AWS its called *Elastic Load Balancer* (*ELB*).

### Notifications and Queues

To further decouple our architecture we can introduce messages and queues.
Services can send messages to each other using Amazon's *Simple Notification
Service* (*SNS*), which can be put into queues using Amazon's *Simple Queue
Service* (*SQS*). Of course other messaging or queueing tool can be used.

## Pricing

There are multiple ways how to price compute, each with different conditions.
That way, all businesses can get the model that works for them the best:

- *on-demand* -- on-demand compute, highest flexibility, highest price. Best for
  irregular, short-term computing.
- *reserved instances* -- reserved amount of compute with lower price. There are:
    - *standard* -- instance type and size, platform and tenancy doesn't change
    - *convertible* -- all specs can change
- *EC2 instance saving plans* -- for a commitment on hourly spending you can get
  a discounted saving plan, no further specification on instance type, ...
  required
- *spot instances* -- cheapest option for unused computing capacity, however,
  the computation must be able to stop within 2 mins (AWS gives us heads-up 2
  mins before stopping the instance)
- *dedicated hosts* -- rent entire physical server, most expensive option

## More managed services

Besides EC2 there are other, more managed, computing that AWS offers. Mainly:


### Amazon's Lambdas

*Amazon Lambdas* are functions that should finish within 15 mins. Serverless
setup (no OS), AWS manages the environment, scaling, queuing completely. Pay for
each minute an instance of the lambda is running.

### Containerized environments

For running [Docker containers](./docker.md), you can either manage your own EC2
instances running a docker daemon or use *AWS Fargate* that abstracts away the
OS, and you only need to worry about the orchestration of the containers.

For container orchestration you can use either
- *Elastic Containers Service* (*ECS*) -- Amazon's in-house solution
- *Elastic Kubernetes Service* (*EKS*) -- using Kubernetes to orchestrate your
  docker containers
