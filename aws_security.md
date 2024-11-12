---
tags: [aws]
---

# Security in AWS

This note documents the contents of Module 6 of [AWS Cloud Practitioner
Essentials](./sources/aws_cloud_practitioner.md).

## Shared Responsibility Model

AWS's *Shared Responsibility Model* divides the responsibility of all
applications running on AWS Cloud into two parts. One part is taken care of by
AWS. The other part is the AWS's customer's responsibility.

> AWS is responsible for security **of the cloud**. Customers are responsible for
> security **in the cloud**.

AWS's **of the cloud** responsibilities include:
- physical housing, access to physical servers
- infrastructure -- AZs, regions
- hardware security
- all the basis for AWS's services: compute, storage, networking, databases
- any software that is part of AWS's service

Customer's **in the cloud** responsibilities include:
- data encryption, both during transit and in storage
- any software the customer installs -- operating systems, web servers
- any configuration the customer sets -- firewall, network
- customer data -- no leaks, encryption, compliance
- platform management -- account/IAM users permissions, managing identities

## Users administration

Administration of AWS's accounts can get pretty complicated.

### AWS Account and root user

In the beginning we have *AWS Account*. Each account gets a *root user*, that
has (and always will have) permissions to do **anything** the account owns. It's
recommended to use Multi-Factor Authentication for root user. Also it's best
practice to use the root user as least as possible.

### AWS Identity and Access Management (IAM)

*AWS Identity and Access Management* (*IAM*) is a service that enables you to
create multiple *IAM Users*, each with dedicated login credentials and
permissions. Each user, starts with **zero permissions**. Permissions are set by
JSON *policies* that allow/deny users access to API call on a particular
resource. According to the *principle of least privilege*, users should get only
the minimal permissions to carry out their tasks.

Users can be grouped to *IAM Groups*, which can be assigned policies as well.
Each user within a given group, has the permissions the group's policy sets.

Additionally users can (with the right permissions) assume *IAM Roles*. IAM Role
is a temporary set of permissions (again defined by a policy), that the user can
assume. By assuming a role, user's previous permissions are **completely
replaced**, by the role's permissions. It is also possible to work with roles
only, assigning each user a permission to assume particular role.

### Organisations

Typically a company will has several accounts. Customers can administrate these
using *AWS Organisations*. Each Organisation gets a *root account* a parent
account of all other AWS accounts. In AWS Organisations you can set *Service
Control Policies* (*SCPs*) that restrict different accounts on access to:

- AWS services
- AWS resources (e.g. billing)
- individual API calls (that the users/roles under given account can call)

You can group accounts into *Organisational Units* (*OUs*), that group accounts
with the same SCPs.

## Compliance

Companies typically need to comply with legal standards. Customers can access
AWS compliance reports via *AWS Artifact*. Artifact has two parts:

- *AWS Artifact Agreements* -- manage agreements with AWS to handle specific
  data
- *AWS Artifact Reports* -- compliance reports from 3rd party auditors, gives
  instructions what customers need to do to comply with certain regulations

To learn more visit AWS's [Customer Compliance
Center](https://aws.amazon.com/compliance/customer-center/).

## Attacks

As with any service that is accessible from the Internet, AWS services need to
be protected against attacks over the net. Typical example is Distributed Denial
Of Service (DDoS). In DDoS attacker commands several stations (the *distributed*
part), which simultaneously bombard the target service with requests, until the
service is unable to process any true requests (i.e. *deny service* to them).

AWS already protects services against brute-froce attacks such as DDoS using raw
power of scaling. Any regional service (e.g. [ELBs](./aws_computing.md) or
[Security Groups](./aws_networking.md)) has so much available compute that to
overwhelm such a service would cost so much, the attack wouldn't make sense.

### AWS Shield

*AWS Shield* adds another layer of security of protecting services from attacks.
The basic version *AWS Shield Standard* is free and active for all AWS services
and protects against common attacks. *AWS Shield Advanced* is a paid service
with analytics, which protects against more sophisticated attacks.

### AWS WAF

AWS offers a *Web Application Firewall* (*WAF*) that monitors incoming traffic
and filters it using *web access control list* (*web ACL*). It integrates with
[CloudFront](./aws_infrastructure.md) and [ELB](./aws_computing.md).

### Amazon GuardDuty

*Amazon GuardDuty* is a service that continuously checks your infrastructure for
threat detection. It integrates with other services' logs such as
[VPC](./aws_infrastructure.md) flow logs or DNS logs. Reports back the findings
in the Management console together with recommendations how to fix them. You can
setup [AWS Lambda](./aws_computing.md) to fix issues automatically.

### Amazon Inspector

*Amazon Inspector* runs an automated security checks on target services and
reports back the results together with recommendation how to fix the issue.

## Data encryption

For encrypting data either in transit or in storage customers can use *AWS Key
Management Service* (*AWS KMS*). KMS enables you to manage, create encryption
keys, which which you can encrypt data (e.g. [DynamoDB](./aws_storage.md)).

