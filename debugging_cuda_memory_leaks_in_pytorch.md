# Debugging cuda memory leaks in PyTorch

There is a [web tool which visualizes a memory
checkpoints](https://pytorch.org/memory_viz).

Before any allocations run:

```python
torch.cuda.memory._record_memory_history(True)
```
which starts collecting info for any allocation, for what purpose was it
allocated and what the memory holds.

To create a snapshot run:

```python
import pickle

with open('snapshot.pickle', mode='wb') as outfile:
    snapshot = torch.cuda.memory._snapshot()
    pickle.dump(snapshot, outfile)
```

Then upload the snapshot to the web app mentioned above.

## Caution

There is [this outdated guide on PyTorch
website](https://pytorch.org/docs/stable/torch_cuda_memory.html). Which may
still offer some helpful information, but the `torch` API has changed and
methods the guide mentions don't exist anymore.
