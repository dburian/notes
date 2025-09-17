# Distributed training with PyTorch

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

### Torch multiprocessing's `spawn`

I don't know much about this, but if we have a training script, we can just:

1. Collect the parameters (configs, command arguments, ...)
2. Abstract training into a function (e.g. `train`)
3. Use `torch.multiprocessing.spawn` to spawn more processes with given
   function:

```python
import torch.multiprocessing as mp
from torch.distributed import init_process_group

def train(rank:int, a, h):
    if h.num_gpus > 1:
        # initialize distributed
        init_process_group(
            backend=h.dist_config["dist_backend"],
            init_method=h.dist_config["dist_url"],
            world_size=h.dist_config["world_size"] * h.num_gpus,
            rank=rank,
        )
...

if __name__ == '__main__':
    ...

    if h.num_gpus > 1:
        mp.spawn(
            train,
            nprocs=h.num_gpus,
            args=(a, h),
        )
    else:
        train(0, a, h)
```

### `DistributedDataParallel`

`DistributedDataParallel` (or `DDP`) wraps model and synchronizes weights during
backward pass. There is a simple [code example on PyTorch's
docs](https://pytorch.org/docs/stable/notes/ddp.html#example). As can be seen
from the example, only the model needs to be wrapped by `DDP`, for the
distributed setup to work. `DDP` registers hooks for gradient accumulation and
*reduces the gradients accross processors*. Then all is done locally, since the
gradients are all the same on each processor.

`DDP` is not responsible for distributing data, only gradients.

### Data distribution with `DistributedSampler`

To distribute data with `DistributedDataParallel` we can use
`DistributedSampler` together with `DataLoader`. If `torch.distributed` is
available (i.e. `torch.distributed.is_available()` evaluates to `True` -- I
assume it is when we initialize distributed as for each process as seen above)
then we can just wrap our training set with `DistributedSampler` and use it as a
`sampler` when initializing `DataLoader`:


```python

trainset = [batch for batch in ...]

train_sampler = DistributedSampler(trainset)

train_loader = DataLoader(
    trainset,
    sampler=train_sampler,
    ...
)
```

With `torch.distributed` `DistributedSampler` figures out on its own the number
of gpus, which rank is the current one and everything else. Then it distributes
the trainset across used GPUs.

### `DataParallel`

There also exist a module for data-parallelism. This is a simple class which
computes the forward pass on a portion of a batch per each gpu in a single
process. It's good default for multi-GPU training, but not as performant as
`DistributedDataParallel` even on a single machine.
