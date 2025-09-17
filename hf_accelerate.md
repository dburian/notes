# 🤗 Accelerate library

Hugging Face's `accelerate` library has
[documentation](https://huggingface.co/docs/accelerate/index), but it's poor.

## How it roughly works

By running:
```bash
acclerate --args --config_file path/to/config.json path/to/script.py --script_args`
```

the following happens:

1. `accelerate` arguments are parsed
2. `accelerate` decides what the user wants -- in this case: launch a training
3. accelerate's config `path/to/config.json` is read and saved as defaults for
   `--args`
4. all of `--args` are then saved to environment variables
5. using `torch.distributed.run`, `accelerate` runs `path/to/script.py` with
   `--script_args` and the given environment variables
6. inside `path/to/script.py` and `accelerate.Accelerator` with minimal config
   (since most of the config is read by `accelerate.AcceleratorState` from the
   environment variables)
7. **Result**: the `path/to/script.py` is running on multiple machines, with the
   same `accelerate.AcceleratorState` which defines how the computation is
   distributed/accelerated

## Avoiding using accelerate config

### Motivation

Particularly when running accelerate with trainer, you're bound to give the
script some config (what to train, train dataset, learning rate, etc.). However,
accelerate needs an extra config:
```bash
accelerate --config accelerate_config.json path/to/script.py
training_config.yaml
```

IMHO it's very cumbersome to take care of two sets of configs, and I'd like
to configure `accelerate` from `training_config.yaml`.

### Real situation

What I describe above is certainly doable. `accelerate`:
- loads config and CLI arguments
- sets environment variables
- uses `torch.distributed.run` to run the script

So it could be straightforward to replicate such behaviour, but already inside
the script. But:


#### Unclear how `accelerate` is initialized

It's unclear how exactly gets `accelerate.Accelerator` is initialized, or more
importantly how `accelerate.AcceleratorState` gets initialized.
`AcceleratorState` defines the environment the computation runs in, and so how
will get the computation 'accelerated'.

The class takes some arguments, but also takes values from environment
variables, and it's messy without clear '*overriding*' variable.

Without clear overriding variable, it's hard to make certain you've set all the
necessary arguments/environment variables.

#### Interaction between `TrainingArguments`, `Trainer` and `accelerate`

The interaction between these classes is also badly documented and maybe even
considered as "*insides*" of the code (something the end user shouldn't even
look at). Without clear API, you don't want to mess around.

#### So?

The result is that if you're bothered with extra configs as I am, you should
probably investigate how to **wrap accelerate rather than replace it**.
