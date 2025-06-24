---
tags: [platform]
---
# OmegaConf

[OmegaConf](https://omegaconf.readthedocs.io/) is a python library that can
load, transform, export, merge, ... YAML configurations.

```python
conf = OmegaConf.create({"k" : "v", "list" : [1, {"a": "1", "b": "2", 3: "c"}]})
print(OmegaConf.to_yaml(conf))
```
gives:

```
k: v
list:
- 1
- a: '1'
  b: '2'
  3: c
```

## Features

### Custom resolvers

You can create a custom resolver like so:

```python
def utc_now(format_str: str) -> str:
    """Returns utc time formatted using `strftime`.

    Can be used as a custom OmegaConf's resolver.
    """
    iso = datetime.now(timezone.utc)
    return iso.strftime(format_str)

OmegaConf.register_new_resolver("utc_now", utc_now)
```

Then doing:
```python
OmegaConf.create({'utc_time': '${utc_now:%Y-%m-%dT%H:%m}')
```

will get you utc time in the given format.

