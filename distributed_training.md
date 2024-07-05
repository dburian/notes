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
