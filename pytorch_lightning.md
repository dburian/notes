# PyTorch Lightning

PyTorch Lightning is a framework that automates parts of the typical PyTorch ML
training code. While it does avoid the boilerplate code, it also takes care of
the engineering-heavy tasks such as multi-GPU training, and so on.

```python
import lightning as L
```

## `L.LightningModule`s

Lightning modules are enhanced `torch.nn.Module`s that add necessary features
for the module to be used in PL's automatic training. These features include:

- `training_step(batch)` -- computes loss & metrics for given batch
- `validation_step(batch)` -- does the same during validation
- `test_step(batch)` -- does the same during evaluation on test split
- `configure_optimizers()` -- configures optimizers to be used for this module
  during training

The module also has some nifty features like logging, which is simply taken care
of by calling `log(key, value)` or `log_dict(dict_values)`. The log method
automatically (can be defined manually) how and when to log. The logging usually
relies on tf's Tensorboard.

## `L.Trainer`

PL's Trainer automates training of a `LightningModule` on a dataset. It takes
in any iterable containing the training (optionally also validation) data and
the lightning module. The trainer class can be configured in many ways:

- gpu or cpu
- grad accumulation steps, grad clipping
- precision -- fp16, bf16, ...
- logger (e.g. Tensorboard logger) the trained `LightningModule` uses

Other, more custom behaviour can be controlled by passing in callbacks.

## `L.Callback`

Callback classes can be given to `L.Trainer`s to do stuff on different occasions
such as on epoch start/end, batch start/end, ... There are different types of
predefined callbacks:

- `ModelCheckpoint` -- configures saving checkpoints of model during training
- `EarlyStopping` -- stops the training if a metric stops improving
- `LearningRateMonitor` -- logs learning rate during training
