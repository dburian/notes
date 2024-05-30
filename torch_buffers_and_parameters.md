# Buffers vs Parameters in PyTorch

PyTorch `Module` can have both parameters and buffers. Both are registered by
torch and both do stuff behind the scenes.

## Parameter

Its a special subclass of a `Tensor` that is automatically registered as a
model's parameter (e.g. when `model.parameters()` is called) when assigned as an
attribute like so:

```python
class Model(torch.nn.Module):
    def __init__(self, *args):
        self.x = torch.nn.Parameter(data=torch.Tensor(...))
```

## Buffer

A memory that is associated with the model that is not considered a model
parameter (won't be touched by an optimizer). Typical example is `BatchNorm`'s
running mean.

```python
self.register_buffer("running_mean", torch.zeros(dim))
```

Also helfpuf is:

```python
torch.cuda.memory_summary()
```

## Both

- are saved
- are registered by pytorch
