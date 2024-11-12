---
tags: [aws]
---
# AWS Global Infrastructure

This note documents part of Module 3 of [AWS Cloud Practitioner
Essentials](./sources/aws_cloud_practitioner.md).

## Regions

Amazon splits the globe to *regions*. Traffic cannot go from one region to
another by default. This is very important for security and compliance (e.g.
data from Germany may be prohibited to be stored outside Germany).

### Selecting a region

When selecting in which region you'll deploy your service you go through the
following points in order:

1. Compliance -- the regulation of local government may prevent you from
   choosing some regions
2. Proximity to customers -- the closer to customers the smaller the latency
3. Available features -- not all AWS services are available in all regions, this
   could limit your choice
4. Pricing -- some regions are cheaper than others

## Availability Zones

Within each region there are several (at least 3) *Availability zones* (*AZs*)
tens of kilometers apart. Each AZ is composed of one or several datacenters. The
best practice is to distribute code across at least 2 AZs for availability, in
case of local nature disaster/power outage/...

Some services are *regionally scoped*, which means that they use at least two
availability zones by default. Such as Elastic Load Balancer, Simple Queue
Service, or Simple Notification Service.

## CloudFront

Sometimes we need to run our services from one region, yet we have customers all
over the world. This would result in high latency for some customers far away
from our region. To overcome this AWS uses content delivery network (*CDN*)
service called *CloudFront*. CloudFront caches data at a *Edge location* close
to the customer, ensuring fast repeated accesses to our service.

### Edge locations

Edge locations are separate from regions and AZs. Edge locations also run
Amazon's DNS called *Amazon Route 53*.

## AWS Outposts

Amazon can run AWS even outside their infrastructure using *AWS Outposts*. AWS
Outposts can turn even an on-premises server into a private region, in which you
can use AWS services.
