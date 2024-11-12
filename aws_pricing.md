---
tags: [aws]
---

# Pricing in AWS

This note documents the contents of Module 8 of [AWS Cloud Practitioner
Essentials](./sources/aws_cloud_practitioner.md).

There are few pricing concepts in AWS:

1. Pay for what you use -- the baseline pricing plan
    - you pay for the compute time/storage volume/# of reads or writes
2. Pay less whey you reserve -- get a discount for long-time commitment
    - commitments usually for 1 or 3 years
3. Pay less with volume-based discounts -- get a discount for large volume of
   used services
    - the more you use the less cost per 1 unit (e.g. per 1 GB of storage)


To compute exact price, without committing to pay it, you can use *AWS Pricing
Calculator*. For an exact configuration, gives you an estimate you can work
with.

## Free Tier

For a number of services AWS offers a *Free Tier*, where for some conditions you
don't get charged for your usage. Depending on the service it can be either:

1. No conditions -- always free
2. First 12 months free -- don't pay 1 year after setting up your AWS account
3. Free trial -- try out the service for free

## View what you owe

There is a [Billing and Cost Management
Dashboard](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/billing-what-is.html),
which shows

- What you owe for the **current** billing cycle
- What you spent **previous** billing cycle as well as prediction for the
  **following** one
- The most-used services
- The most-used Free Tier services and percentage of free allowance limit
- Access to your bills per service broken down to regional costs
- Access to *AWS Budgets*
- Access to *Cost Explorer*

### Consolidated billing

For an [*AWS Organisation*](./aws_security.md), customers can consolidate costs
from all organisational accounts. This has number of advatages:

- All expenses are paid once
- Consolidated usages may yield larger large-volume (bulk) discount

### AWS Budgets

With *AWS Budgets* you can set a spending or usage limit and notifications when:

- you are at some percent of the limit
- you are forecasted to exceed the limit
- you exceed the limit
- ...

The Budget can be limited to just a region or just a service. When a budget is
set, the dashboard tells you how close are you to the limit and if you are
forecasted to exceed the limit within a billing cycle. The expenses are updated
**3 times a day**.

### Cost Explorer

*AWS Cost Explorer* collects your AWS costs over time and helps you understand
any changes in your spending. There are visualizations with filters per given
time period. Additionally you can **tag** any service (e.g. tag services by
project name), which allows you then see the spending per given tag.

## Support Plan

Each AWS account gets some degree of support from AWS. There are the following
plans:

1. *Basic support* -- the free minimum
2. *Developer* -- for testing out AWS
3. *Business* -- for small businesses
4. *Enterprise On-Ramp* -- small enterprises
5. *Enterprise* -- large enterprises

The higher the level the higher degree of support you have from AWS, and the
more urgent your help requests are to AWS. All **levels include everything from
the previous ones**.

### Basic Support

*Basic support* is free and includes:

- documentation/whitepaper access
- access to support communities
- ability to contact AWS for billing questions or [service limit
  increases](./aws_analytics_and_monitoring.md)
- access to limited [Trusted Advisor](./aws_analytics_and_monitoring.md) checks
- access to *AWS Personal Health Dashboard* -- provides alerts and guidance on
  events that might effect your services

### Developer Support

*Developer support* includes:

- Best practice guidance
- Client-side diagnostic tooling
- Building block support -- guidance how to structure AWS services into larger
  architectures

### Business support

*Business support* includes:

- Use-case guidance for user-specific needs
- All [Trusted Advisor](./aws_analytics_and_monitoring.md) checks
- Limited support for common 3rd party software

### Enterprise On-Ramp support

*Enterprise On-Ramp support* includes:

- pool of *Technical Account Managers* (*TAMs*) -- the primary point of contact
  with AWS support, provide **pro-active support** such as architecture
  consultations
- 30 min response time for business critical issues
- extra tools for monitoring costs and performance

### Enterprise support

*Enterprise support* includes:

- designated TAM
- 15 min response time for business critical issues
- extra tools for monitoring costs and performance

## Marketplace in AWS

AWS has *Marketplace* through which we can buy software from 3rd parties that
runs on AWS or integrates with it. There are also complete solutions for very
specific problems.
