---
tags: [python,coding]
---
# Python packages

Creating python package is now very easy. So easy I would argue it should be the
default way of structuring code even for the smallest of projects.

## pyproject.toml

[Read more on pip's
docs](https://pip.pypa.io/en/stable/reference/build-system/pyproject-toml/)

`pyproject.toml` is declarative way how to define a package. Place it in the
root of your project directory. Here is a basic template:

```toml
[build-system]
requires = ["setuptools"]
build-backend = "setuptools.build_meta"

[project]
name = "<package_name>"
version = "<major_version>.<minor_version>.<micro_version>.<..."
authors = [{name = "<author_1_name>", email = "<author_1_email>"}, { ... }]
dependencies = [
  "<package_1>{>=,==,<=,><,==}<version>,..."
  ...
]

[project.scripts]
# When path changes, you need to reinstall
<script_name> = "<package_name>.<module>:<function_with_no_args>"

[project.urls]
homepage = "<url>"
repository = "<url>"
```

Then put code under `src/<package_name>/`.

### Editable installs

Since `pip`'s version 21.3 you can install packages defined using
`pyproject.toml` in editable mode:

```bash
pip install -e .
```


TODO: poetry?
