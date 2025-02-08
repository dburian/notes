---
tags: [ml,platform]
---

# Distributed training

Training ML models on multiple GPUs/servers is called distributed training. The
vocabulary here is:

- *node* -- the name for a single computing unit (server)
- *world size* -- the number of processes (usually one process == one GPU) for all nodes
- *rank* -- index of a particular process

So for training on 2 servers, each with 4 GPUs we have:

- 2 nodes (== 2 servers)
- 2 * 4 = 8 world size
- ranks go `[0, 1, 2, ..., 6, 7]`

## Strategies

There are multiple trategies:

- *Distributed Data Prallelism* (*DDP*) -- a model is copied onto each gpu and
  each gpu gets different inputs and resulting gradients are averaged across
  processes. In other words, the *data get distributed*, the model is replicated
  across processes.

## Distributed Training in PyTorch

PyTorch has recently been given [support for distributed
training](https://pytorch.org/tutorials/beginner/dist_overview.html) out of the
box. There is only some extra code, but not much. Extra packages (such as
[Accelerate from HuggingFace](https://huggingface.co/docs/accelerate/index))
build on this PyTorch functionality to provide even higher abstraction.

### Process groups

Process groups identify the models that should be synchronized together. It is
required that a process group is initialized before training on each processor:

```python
dist.init_process_group("gloo", rank=rank, world_size=world_size)
```

The above process group will act as the default process group for each
processor. Meaning that every distributed module created on given processor will
synchronize its state across processors in this, just initialized process group.

### Torch spawn/run

TODO

### `DistributedDataParallel`

`DistributedDataParallel` (or `DDP`) wraps model and synchronizes weights during
backward pass. There is a simple [code example on PyTorch's
docs](https://pytorch.org/docs/stable/notes/ddp.html#example). As can be seen
from the example, only the model needs to be wrapped by `DDP`, for the
distributed setup to work. `DDP` registers hooks for gradient accumulation and
*reduces the gradients accross processors*. Then all is done locally, since the
gradients are all the same on each processor.

`DDP` is not responsible for distributing data, only gradients.

### Data distribution

TODO


