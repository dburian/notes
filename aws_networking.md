---
tags: [aws]
---
# Networking in AWS

This note documents the contents of Module 4 of [AWS Cloud Practitioner
Essentials](./sources/aws_cloud_practitioner.md).

## Virtual Private Cloud

We group services in AWS into *Virtual Private Clouds* (*VPCs*). VPC behaves
like a private network. VPC can have several subnets, each of which can be
marked as private or public.

## Accessing VPCs

To access a VPC, it must have a gateway. Gateways map incoming traffic to one of
the services inside a VPC. For public-facing services we can use *Internet
Gateway*, which allows public traffic pass into the VPC. For private access use
VPN together with *Virtual Private Gateway*.

### Direct Connect

To connect directly to a VPC, through a physical, private fiber you can use
*Direct Connect*. The establish this connection, you must work with Amazons
local partners.

## Controlling access to services

We can limit the access to our services at several layers.

### Network Access Control List

Each VPC has a *Network Access Control List* (*ACL*), which is a **stateless**
filter of packets travelling both to the VPC (inbound) and from the VPC
(outbound). ACL filters packets based on subnets. Each AWS account has
default ACL, allowing all inbound and outbound traffic. For each VPC, one can
either use the account's default ACL or create a custom one, which starts off as
not allowing any traffic. To modify ACL you can add whitelist or blacklist
rules.

### Security Groups

Each [EC2 instance](./aws_computing.md) has additional networking layer of
protection called *Security Group*. Security groups are **statefull** filters of
packets travelling only to the EC2 (inbound). The **statefull** keyword means,
that respond packets to previous outbound request will be allowed in. By
default, Security Groups deny all inbound traffic.

Even though Security Groups manage access to an EC2 instance, they are checked
at the regional level by another physical machine. This means that EC2 instances
are [safe from brute attacks](./aws_security.md) that Security Groups can take
care of, without limiting the EC2 instance throughput.
