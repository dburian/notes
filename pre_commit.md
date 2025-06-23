---
tags: [platform]
---

# Pre-commit

[Pre-commit](https://pre-commit.com/) is a tool that runs other tools like
linters, formatters, checkers on `git commit`.

Internally `pre-commit` installs a 'pre-commit' [git
hook](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) that runs when
you run `git commit`.

## Installation

```bash
# Install the library
pip install pre-commit

# Install pre-commit git hook
pre-commit install
```

## Configuration

To specify which tools `pre-commit` runs, there is a configuration named
`.pre-commit-config.yaml`. The full spec is given in [pre-commit's
docs](https://pre-commit.com/#adding-pre-commit-plugins-to-your-project).

Here is an example:
```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v2.3.0
    hooks:
      - id: check-yaml
      - id: end-of-file-fixer
      - id: trailing-whitespace
  - repo: https://github.com/astral-sh/ruff-pre-commit
    # Ruff version.
    rev: v0.11.13
    hooks:
      # Run the linter.
      - id: ruff-check
        args: [ --fix ]
      # Run the formatter.
      - id: ruff-format
  - repo: local
    hooks:
      - id: pytest-check
        name: pytest-check
        entry: pytest
        language: system
        pass_filenames: false
        always_run: true
```

## Usefull commands

### Run on specified files

```bash
# Run on all python files
pre-commit --files "src/**/*.py"

# Run on all files
pre-commit --all-files
```

### Skip pre-commit

Bypass pre-commit git hook with:
```bash
git commit --no-verify
```

### Rebasing

Pre-commit git hook doesn't run on rebase. To run pre-commit on rebased commits
run:

```bash
git rebase -x 'pre-commit run --from-ref HEAD~ --to-ref HEAD' main
```

The `-x` tells git to run following command after every commit. So if we added
`-i` we'd see that commits are interleaved with the given `pre-commit` command.

