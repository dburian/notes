---
tags: [platform, ml]
---

# NeMo's `NeuralModule`

`NeuralModule` represents a model that wraps pretty much all we do with models.
Effectively, it inherits from

- `torch.nn.Module` -- forward method
- [PL](./pytorch_lightning.md)'s `LightningModule` -- able to be trained with
  PL's trainer
- `nemo.core.Serialization` -- serializable to/from dict
- `nemo.core.Typing` -- adds (optional) input and output tensor type checks for
  forward pass

I say effectively, because it doesn't do all of that in a single class. TODO

The '*all we do with models*' covers:

- load dataset (train/val/test)
- preprocess dataset
- postprocess predictions
- loading/saving all attached `NeuralModule`s from/to a checkpoint

## Checkpoints

NeMo's `NeuralModule` aims to encompas everything, not just weights. Therefore,
there are `save_to` and `restore_from` methods, that save/restore:
- the weights
- serialized config
- any other parts of the model (e.g. tokenizer)

To extract just the weights we can call `extract_state_dict_from(<path>)`, which
gets us the PyTorch's state dict.

## `nemo.core.Typing`

The `Typing` class adds two properties `input_types` and `output_types` which
may specify the types of inputs and outputs of the `forward` method using
dict of `NeuralType`s. **By default, no type checking is done.** Check are only done if
we decorate the forward method with `typecheck()` **and** the input tensors have
`neural_type` property (to support backwards compability).

The consequence of `Typing` is that the modules must be called **with kwargs
only**.


### `NeuralType`

`NeuralType` is a class that specifies tensors by their
- dtype -- `float`, `bfloat`, `float16`, ...
- the dtypes semantic -- encoder output, logits, embeddings, probabilities,
  labels, ...
- the dimensions' semantic -- each dimension is given an `AxisKind` that can be
  described by a string. E.g.:
    - `B`, `batch`, `b` -- batch dimension
    - `T`, `time`, `t_*` -- time dimension
    - `C`, `d`, `channel` -- channel dimension
    - ...

The semantic of the dtype is represented by a class. NeMo provides some basic
classes, but it is encouraged that more specific per-model types will be
created. `@typecheck` respects type hierarchy. TODO: covariant or contravariant?

### Example of input/output type spec

Let's say there is an output type spec (i.e. something that would `output_types`
return):

```python
{
  'y': NeuralType(axes=('B', 'T', 'C'), elements_type=EncodedRepresentation()),
  'h_c': [NeuralType(axes=('D', 'B', 'C'), elements_type=EncodedRepresentation())],
}
```

Which says:
- tuple is returned with elements named `y` and `h_c`
- `y` is a encoded representation with batch, time and channel dimensions
- `h_c` is a *tuple* of equally typed tensors:
    - again are encoded representation
    - with generic dimension-type, batch and channel dimensions

## `ModelPT`

Actually `NeuralModule` is fairly slim. Most of the functionality resides in
`ModelPT`, which implements NeMo's functionality (the dataset loading, the
checkpoints) into Lightning's `LightningModule`. Quite surprisingly,
`NeuralModule` does not inherit from `ModelPT`, yet if we want our
`NeuralModule` to be trainable **we need to inherit `ModelPT`**.

## Typical implementation

- usually only accepts
    - `cfg` -- config
    - `trainer` -- trainer to be used to train the model, optionally set after
      the construction with `set_trainer`
- usually the model is composed of several sub-models all implemented using
  `NeuralModule`
    - then the constructor can just call
      `self.from_config_dict(cfg.<name_of_submodule>)` to create the submodule

## Configuration

NeMo's `NeuralModule` are configurable through [OmegaConf](./omegaconf.md) and
[Hydra](./hydra.md). OmegaConf takes care of parsing dicts/yamls and converting
them to object with unified API. Hydra steps in to get configurable,
hierarchical configuration from CLI.

Each `NeuralModule` gets a default config.

### Config format

Each config part contains `_target_` which is the `__module__.__name__` path to
a class which is instanted by Hydra with the rest of the parameters as kwargs
given to the constructor.

OmegaConf supports variable interpolation so we can do:

```yaml
model:
  n_embed: 256
decoder:
  n_embed: ${model.n_embed}
```

meaning `decoder.n_embed = model.n_embed`. The interpolations are evaluated
lazily by default.

## Data methods

To support multi-dataset and multi-GPU support one has to override
`multi_validation_epoch_end` and `multi_test_epoch_end` for the model, which
take care of the collation of results for multiple datasets or multiple forward
epochs/steps on different GPUs.


