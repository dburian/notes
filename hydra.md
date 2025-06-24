---
tags: [platform]
---
# Hydra

[Hydra](https://hydra.cc/) is a Python library that makes loading of configs in
scripts easy and very streamline. Together with [OmegaConf](./omegaconf.md), we
can add support for YAML configuration loading as follows:

```python
import hydra
from omegaconf import DictConfig, OmegaConf

# [!code highlight:3]
@hydra.main(version_base=None, config_path="conf", config_name="config")
def my_app(cfg : DictConfig) -> None:
    print(OmegaConf.to_yaml(cfg))

if __name__ == "__main__":
    my_app()
```

We can just run the above script by:
```bash
python script.py
```

If we wanted to override the `config_path` or `config_name` we can use:
```bash
python script.py --config-path ../alternative_configs --config-name special.yaml
```

We can also overwrite the config parameters in the commandline using [hydra
special cli argument
syntax](https://hydra.cc/docs/advanced/override_grammar/basic/).

## Instantiating

Hydra can instantiate classes from configs with the following format:

```yaml
_target_: package.folder.file.ClassName
keyword_arg1: value
keyword_arg2: another_value
```

Then we can just call:
```python
cfg = OmegaConf.load('instantiating_example.yaml')
hydra.utils.instantiate(cfg)
```

## Features

### Structural schemas

When defining a config, I'd like to create a dataclass that captures and
documents the config. Something like this:

```python
@dataclass
class TrainConfig:
  """Configuration to train a ML model.

  Arguments
  ---------
  device:
    Device on which to train.
  dataset:
    Abosolute path to a dataset.
  """

  device: str
  dataset: str
```

[OmegaConf](./omegaconf.md) can create a [structured
config](https://omegaconf.readthedocs.io/en/2.0_branch/structured_config.html)
from a dataclass. Hydra then enables to [use the structured as a
'schema'](https://hydra.cc/docs/tutorials/structured_config/schema/) to
check the loaded config against. The process is as follows:

1. Instantiate the dataclass and create a structured config out of it
2. Using OmegaConf merge the loaded yaml config into the structured config
  - OmegaConf will complain if the attributes' types do not correspond, or the
    yaml config introduces new attributes.

#### Setup

You tell hydra that named config `config_schema` (name is up to you) should
correspond to the dataclass:

```python
@hydra.main(config_name='train', config_path='../configs/')
def main(config: TrainConfig) -> None:
    pass

if __name__ == '__main__':
    cs = ConfigStore.instance()
    cs.store(name="config_schema", node=TrainConfig)

    main()
```

The config at `../config/train.yaml` then has to use the named config
`config_schema` as default (that is why the merge of the two configs happens):

```yaml
# Inside ../config/train.yaml

defaults:
  config_schema
  _self_
```


#### Shortcomings

The drawbacks are:
- even though you can declare that `main` receives `TrainConfig`, hydra actually
  gives you `omegaconf.DictConfig`.

### Custom resolvers

You can use [OmegaConf's custom resolvers](./omegaconf.md#custom-resolvers), but
usage with hydra has several shortcomings:

- hydra 'resolves' (= evaluates resolvers) lazily. Meaning if nobody is going to
  ask for that value, nobody will resolve it.
- hydra saves unresolved config. So if you're thinking to add `'${git_commit:}'`
  to your config that would resolve to current commit, hydra will save just the
  string `'${git_commit:}'`.


