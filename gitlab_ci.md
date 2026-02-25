# Gitlab CI

The CI has [extensive official documentation](https://docs.gitlab.com/ci/) so I
include only stuff to help me/you get going quickly.

## `.gitlab-ci.yml`

The CI is defined declaratively using a yaml. The CI is run on several events,
each of which runs a pipeline specified by initial docker image and stages.

Then you can define job, for each job, you can specify:
- when it is run -- on which gitlab events
- in which stage it is run
- what it does: define the `script`

Jobs in a given stage run in parallel, stages run sequentially.

```yaml
image: ubuntu:latest

variables:
  PYTHON: python3

stages:
  - install
  - check

install_repo:
  stage: install
  script:
    - pip install Cython==3.0.11 wheel
    - pip install -r requirements.txt
    - pip install -e .


pre-commit-mr:
  stage: check
  rules:
    # For merge request.
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
  script:
    - git fetch
    - changed_files=`git diff --name-only ${CI_MERGE_REQUEST_SOURCE_BRANCH_NAME}..${CI_MERGE_REQUEST_TARGET_BRANCH_NAME}`
    - pre-commit run --files ${changed_files}

pre-commit-commit-to-master:
  stage: check
  rules:
    # On master.
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
  script:
    - pre-commit run --all-files
```
