---
tags: [aws, platform]
---

# AWS Identity Center

AWS Identity Center allows authorized access to AWS. Identity Center has a
separate list of users, that can be either manually created or imported from
external *Identity provider* (*Idp*). Through Identity Center, each user can be
given access to one or many AWS accounts or applications.

## Setup with AWS Organisations

Here we describe
- how to get credentials to log into AWS access portal and
- how to assign each log-in credentials access to a given user.

The end result is that in Identity Center we can manage log-in users. Each user
has a separate log in username, password and MFA. When logged in, he can
choose from the AWS accounts we've given him access to. Upon choosing an AWS
account, he can manage AWS with the permissions of that account.

### The high-level steps

1. Create an *AWS Organisation* and an AWS account that has permissions you want
   the users to have.
2. Setup AWS Identity Center for AWS Organisations. (Just click through the
   setup windows)
3. In the most basic setup, we configure Identity Center users manually. To do
   that we choose the *identity source* to be *Identity Center directory*.
   Alternatively, the identities of the users can come from external *Identity
   Provider* (*IdP*).
4. Create a AWS Identity Center user, who recieves email and can setup password
   and MFA.
5. Through Identity Center, create a *Permission set*, which defines the
   policies the user will have under the given account.
6. Assign an Identity User, to the created AWS account with the create
   permission set.

That's it now you have a user, that can login, choose the AWS account and the
permission set for that account through which he/she will manage AWS services.



