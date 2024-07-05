# HuggingFace `datasets`

HuggingFace's `datasets` library composes several useful features:

- dataset repository -- source of data
- dataset transformation tools
- API that is useful for ML training


## `IterableDataset`

IterableDataset are efficient for ML training since they are lazy: all
transformations are done lazily and the data are *streamed* from disk. This
means that compared to classic map-stype `Dataset`

- it is more memory efficient. 
- but also that it cannot support full shuffle (it implements approximate
  shuffle with buffer)

## Sharding

Datasets can be split into multiple *shards* -- pieces of dataset, designed to
run on different [*node* (GPU)](./distributed_training.md).
`split_dataset_by_node` retrieves shards assigned to a a given node.
