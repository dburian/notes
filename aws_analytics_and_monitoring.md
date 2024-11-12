---
tags: [aws]
---

# AWS Analytics and Monitoring

This note documents the contents of Module 7 of [AWS Cloud Practitioner
Essentials](./sources/aws_cloud_practitioner.md).

## Monitoring

### Metrics

AWS has built-in service for monitoring called *CloudWatch*. CloudWatch
continually measures metrics, which can be displayed in a **dashboard** or we
can set alarms when metrics reach a particular limit. Alarms notify or
automatically carry out an action, such as sending notification to [Auto Scaling
Group](./aws_computing.md) to increase number of EC2 instances due to high
utilization.

### API calls

AWS also caches all API calls by a service called *CloudTrail*. CloudTrail
listens to API calls and logs the following information for each call:

- What API was called and with which parameters?
- Who called the API? (Which IAM user?)
- When was the API called?
- How was the request made?

These logs can also be of assistance during auditing (e.g. the root user didn't
touch anything). CloudTrail can optionally detect unusual API activities with
*CloudTrail Insights*.

### Recommendations on AWS environment

*AWS Trusted Advisor* is a web service that inspects our AWS environment and
provides real-time recommendations in 5 categories:

- Cost Optimization -- e.g. unused EC2 instances, bad pricing plan
- Performance -- e.g. EBS volumes performance
- Security -- e.g. MFA on IAM accounts
- Fault Tolerance -- e.g. service only in one AZ
- Service Limits -- e.g. hitting the limit for maximum number of VPC per region

Trusted Advisor always includes basic checks. According to our Support Plan,
more checks may be enabled.
