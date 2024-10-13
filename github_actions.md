---
tags: [platform]
---
# Github Actions

Github Actions are Github's way of doing Contnuous Integration and Continuous
Delivery (CI/CD). They allow to run custom scripts or pre-made scripts (called
actions) on a set of events.

## Dictionary & Introduction

Github Actions are described by declarative *workflow* files. Each workflow
defines a set of *jobs* that can be dependent on each other in an arbitrary way.
This means that some jobs can run in parallel, while the dependent ones must run
sequentially. Each job is defined by a set of steps which are carried out in
order to finish the job. The steps share context (they are run on the same
Virtual Machine), so they can share data and state. Each step is a custom script
or an *action*, which is just a packaged re-usable script.

## Workflow file syntax

Here is an example

```yaml

# Name of the workflow
name: learn-github-actions

# Name of the run
run-name: ${{ github.actor }} is learning GitHub Actions

# Events on which the workflow runs
on: [push]

# Environment variables for all jobs
env:
  NODE_ENV: production

jobs:
  publish_to_github_pages:

    # Configures runner of the job
    runs-on: ubuntu-latest

    # Sequential steps that carry-out the job
    steps:

      # Checkouts current repo 
      - uses: actions/checkout@v4

      # Actions can be named for debug purposes and configured via the 'with'
      # key.
      - name: Setup node
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      # Runs a script
      - run: npm i

      # Use of environment variables
      - run: echo "Node env is $NODE_ENV"

      # Overwrite environment variables
      - run: echo "Node env is $NODE_ENV"
        env:
          NODE_ENV: development
```

## On navigation

The steps run isolated, from the working directory of the workspace,
given in the GITHUB_WORKSPACE environment variable. So changing directory in
one step does not change it in another.

## Actions development

Actions can be defined in other repo or in the current one.

### Current repo's actions

Identified by `{filename}`, for action under `.github/workflows/{filename}`.

### Actions defined

Identified by `{github_user}/{repo}@{ref}`, where `ref` can be commit SHA,
release tag or branch. The action is defined at
`{owner}/{repo}/.github/workflows/{filename}@{ref}`.
