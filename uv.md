---
tags: [platform]
---
# uv

[uv](https://docs.astral.sh/uv/) is package and virtual environment manager for
python. The official docs are great, so here I only include my learnings and
cheetsheet.

## Installing dependencies

### Groups and extras

Besides regular dependencies you can install packages either as part of an
*extra* or *group*.

*Extras* are
- installed as part of `project.optional-dependencies`,
- are standartized -- recognized by pip and other tools
- are published with your package -- someone will be able to do `pip install
  yourpackage[yourextra]`.

On the other hand, *groups* are:
- uv's creation
- are not published with your code -- only for development

### Conflicts

You can define conflicting groups or extras via `tool.uv.conflicts`. Here there
is a snippet for groups, extras would be the same except for substituing `group`
for `extra` in `tool.uv.conflicts`:

```toml
[dependency-groups]
torch27 = [
    "torch==2.7.1",
    "torchaudio==2.7.1",
    "torchvision==0.22.1",
]

torch26 = [
    "torch==2.6.0",
    "torchaudio==2.6.0",
    "torchvision==0.21.0",
]

[tool.uv]
conflicts = [
    [
      { group = "torch26" },
      { group = "torch27" },
    ],
]
```

### Sources

Each dependency's source (no matter if it's inside a group, or extra) can be
further specified with `tool.uv.sources`. This specification is only recognized
by `uv`. Here we can specify two indices for `pytorch`, depending on the extra
chosen.

```toml
[[tool.uv.index]]
name = "pytorch-cu128"
url = "https://download.pytorch.org/whl/cu128"
explicit = true

[[tool.uv.index]]
name = "pytorch-cu126"
url = "https://download.pytorch.org/whl/cu126"
explicit = true


[tool.uv.sources]
torch = [
  { index = "pytorch-cu128" , extra="torch27"},
  { index = "pytorch-cu126" , extra="torch26"}
]
```

### Syncing and locking

uv works with the concept of environment lock -- a definition of the environment
described in `uv.lock`, that guarantees reproducibility. `uv.lock` file
describes all versions of all installed dependencies (and subdependencies and
their dependencies ...).

`uv.lock` would be part of your repository, so you usually *just have it*, but
in case you want to create it you can use: `uv lock`.

To make sure your environment (saved on the disk at `.venv` at root of the
project) is equal to what is described in `uv.lock`, you run `uv sync`.

## Python version

uv respects `.python-version` file. It's something like `uv.lock` for your
python, since the usable python versions should be defined in `pyproject.toml`.

## Running something with uv

Although uv installs everything into a virtual environment, but **it's not
recommended to use it directly.** Instead, authors recommend running:

```bash
uv run python path/to/script.py
```

which has the added benefit that it syncs the environment before executing
anything.
