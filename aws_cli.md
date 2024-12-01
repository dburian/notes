---
tags: [platform, aws]
---
# AWS CLI

AWS command line interface (CLI) is an application that sends API calls to AWS
to control it. Since everyting in AWS is controllable it is as powerful as AWS
web console.

## Getting credentials

There are [several ways how to gain
credentials](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-authentication.html).
The recommended way is through [*IAM Identity
Center*](./aws_identity_center.md), which is designed to handle multi-account
organisations, but for single accounts is kind-of convoluted. Still if you know
what to do, it can be done in minutes.

Steps:
1. Setup Identity Center user, and give it access to a AWS Account. Described in
   [AWS Identity Center note](./aws_identity_center.md).
2. Upon logging in, in the access portal, one can click on '*Access Keys*' next
   to a permission set for given AWS account. Click on it.
3. A modal pops up that guides you through setting up aws cli.
