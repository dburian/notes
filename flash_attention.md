---
tags: [ml,transformers]
---
[paper]: https://arxiv.org/abs/2205.14135
[paper_v2]: https://tridao.me/publications/flash2/flash2.pdf

# Flash attention

An IO-aware implementation of [transformers'
self-attention](./transformer_self_attention.md) and of [sparse
self-attention](./transformer_sparse_self_attention.md) introduced by [Dao et
al. (Jun 2022) \[v1\]][paper] and updated by [Dao, Tri (July 2023)
\[v2\]][paper_v2].

The paper is a reaction to the fact that sparse attentions usually fail to
increase wall-clock time of training.

The main point is to optimize IO reads. This can achieve large wall-clock
speedups (2x-15x). The kernelized version doesn't compute the full $QK^T$ matrix
and instead does first a loop over rows and then over columns, where each row is
loaded into GPU SRAM (on chip *S*hared *MEM*ory -- basically the memory the
computing cores have straight access to) and only saved to GPU HBM
(*H*igh-*B*andwidth *M*emory -- higher latency than SRAM) once the full row of
$\textrm{softmax}(\frac{QK^T}{d_v})V$ is computed.

> Did you know that an operation on GPU is called a *kernel*. That's way we are
> talking about *kernelized* self-attention since it is one operation on a GPU.

## Second version

The second version of Flash Attention, focused on parallelization inside GPU
card. They achieved even higher performance boosts by optimizing how
self-attention operation is parallelized over the GPU cores.

