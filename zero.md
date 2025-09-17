# ZeRO

*Zero Redundancy Optimizer* or (*ZeRO*) is a [distributed
training](./distributed_training.md) implementation that heavily optimizes
distributed training using Data parallelisation (*DP*). ZeRO was introduced and
later expanded in:
- [Rajbhandari et al. (2019)](https://arxiv.org/pdf/1910.02054)
- Zero-Offload: [Ren, Rajbhandari et al. (2021)](https://arxiv.org/pdf/2101.06840) -- focuses
  on offloading compute and memory to CPU
- Zero-Infinity: [Rajbhandari et al. (2021)](https://arxiv.org/pdf/2104.07857)
  -- improves offloading and makes it possible to offload to NVMe disk
- Zero++: [Wang et al. (2023)](https://arxiv.org/pdf/2306.10209) -- improvement
  from DeepSpeed that adds quantized parameters and gradients, and
  hierarchical partitioning.

ZeRO aims to reach the computational efficiency of DP, while achieving the
memory efficiency of TP. Even the introductory paper shows that ZeRO enables
training of 1TB model in an age where just single digit B models were SOTA.

We first describe the first introductory paper, then focus on the
extensions/improvements.

## Vanilla ZeRO (from 2019)

Before diving into ZeRO itself, we mention the author's extended introduction
into efficient training:
- [What takes up memory in training deep models?](./dl_memory_consumption.md)
- Optimization with a single GPU training:
  - [Gradient/Activation checkpointing](./gradient_checkpointing.md) -- complementary to ZeRO
  - CPU offload -- often can hinder training as a lot of time is wasted during
    GPU-CPU-GPU transfers, ZeRO uses CPU offloading only scarcely
  - Memory Efficient optimizers -- again complementary to ZeRO

The key insight of the paper is:

> DP and MP keeps all model states (optimizer states, parameters, gradients),
> but not all are used all the time.

The main idea is to leverage this information, partition the redundant memory
(thereby decreasing the required memory per processor) and only gather it once
it is needed.

ZeRO is split into two parts:
- ZeRO-DP -- implementation of DP with tweaks
- ZeRO-R -- optimization of residual memory

### ZeRO-DP

ZeRO-DP has 3 stages:
- $P_{os}$ -- partitioning of optimizer states
- $P_g$ -- partitioning of gradients
- $P_p$ -- parameter partitioning

The stages are enabled cumulatively and each saves more memory. All stages incur
no extra communication overhead compared to pure DP, except for $P_p$ which
increases communication $\times1.5$.

Below is a visualization of memory savings for cumulatively enabled stages.
Variable meaning:
- $\Psi$ is the number of model parameters,
- $K$ is optimizer state multiplier (for each parameter, optimizer takes up $K$ bytes).
- $N_d$ degree of DP parallelisation

[fp16](./dl_memory_consumption.md) is assumed, so 1 parameter == 2 bytes.

![Visualization of 3 ZeRO-DP stages from
https://arxiv.org/pdf/1910.02054](./imgs/zero.png)

The important thing to notice is that in the third stage $P_{os+g+p}$ ZeRO-DP
effectively achieves the same thing as MP: the memory required per process
decreases as number of processes grows. In other words, given enough processes,
we can train arbitrary large model (to an extent).

#### Vanilla DP

Since we compare ZeRO-DP to vanilla DP, it's helpful to briefly describe DP in a
little more [detail](./distributed_training.md#data-parallelisation-(DP)).

#### $P_{os}$

$P_{os}$ partitions optimizer states across all processes, so each GPU holds
only $\frac{1}{N_d}$ of optimizer states. Again we assume fp16, so *optimizer
states also contain fp32 copy of parameters*. While it's clear that this saves
memory, it's not clear how to achieve same communication overhead as vanilla DP.

Training step in detail with $P_{os}$:
1. Batch is partitioned across processes, same as DP
2. each process does forward and backward pass, computing fp16 gradients
3. [reduce-scatter](https://docs.nvidia.com/deeplearning/nccl/user-guide/docs/usage/collectives.html#reducescatter)
operation averages fp16 gradients splits, where each process averages the
gradients that correspond to it's partition of optimizer states
4. each process updates its optimizer states -- first, second moment of
   gradients and fp32 copy of parameters
5. fp16 parameters are synchronized using [all-gather](https://docs.nvidia.com/deeplearning/nccl/user-guide/docs/usage/collectives.html#allgather) operation.

The communication differs from pure DP only in 3. and 5. point which together
induces $2\Psi$ communication volume, which is the same as DP.

#### $P_{os+g}$

$P_{os+g}$ partitions optimizer states *and fp16 gradients*. This is
similar to $P_{os}$ except that after fp16 gradients are synchronized *as they
become available*. To be more specific: as backward pass progresses, gradients
are assigned to buckets that correspond to the partitions. Once a partition is
filled it's reduced on the process that is responsible for updating
corresponding parameters and **the gradients are deleted on all other
processes**. This reduces the memory footprint from $2\Psi$ bytes to
$\frac{2\Psi}{N_d}$.

So the difference between $P_{os}$ and $P_{os+g}$ is that:
- with $P_{os}$ all gradients are computed on all processes, and then are
  reduced
- with $P_{os+g}$ gradients are moved to the responsible process as soon as they
  are available for the entire partition

This doesn't incur extra communication since the reduce-scatter operation from
$P_{os}$'s step 3 is split into $N_d$ operations with $\frac{1}{N_d}$ the volume.

#### $P_{os+g+p}$

Parameter partitioning partitions parameters across processes. This means that
during the forward and the backward passes processes responsible for given
partition need to broadcast their parameters to other processes. In other words,
parameters are fetched from other processes **as they are needed**.

Now the required memory per process is $O(\frac{\Psi}{N_d})$ for Adam, we have
$\frac{16\Psi}{N_d}$. This means that we can train arbitrarily large models as
long as we have enough processes.

The communication volume now increases to $3\Psi$ which is $1.5\times$ of
vanilla DP:
1. $N_d\frac{\Psi}{N_d}$ -- $N_d$ times we need to broadcast $\frac{\Psi}{N_d}$
   parameters during forward pass
2. $N_d\frac{\Psi}{N_d}$ -- $N_d$ times we need to broadcast $\frac{\Psi}{N_d}$
   parameters during backward pass
3. $N_d\frac{\Psi}{N_d}$ -- $N_d$ times we reduce $\frac{\Psi}{N_d}$ gradients
   in the process responsible for updating the corresponding parameters

There is a great video that illustrates a training step with ZeRO stage 3 in
[Microsoft blog
post](https://www.microsoft.com/en-us/research/blog/zero-deepspeed-new-system-optimizations-enable-training-models-with-over-100-billion-parameters/).

### ZeRO-R

ZeRO-R has 3 separate tricks that kind-of accompany ZeRO-DP:
- $P_a$ -- decreases memory required to store activations **when using TP**
- $C_B$ -- introduces constant-sized buffers to avoid blow-up of temporary
  buffers  
- $M_D$ -- better management of fragmented memory

#### $P_a$

When using Tensor parallelism (such as MegatronLM (TODO: link)), each layer gets
partitioned across several processes. However, to compute the outputs for all
partitions, all inputs (activations of the previous layer) must exist in all
processes. This leads to *replication of the activations*.

$P_a$ addresses this by partitioning activations across the tensor parallel
processes (which share the same batch). This decreases the memory consumption by
TP degree.

With activation checkpointing, the activation checkpoints are partitioned
instead of the actual activations.

Additionally, activation checkpoints can be offloaded to CPU, which is called
$P_{a+cpu}$

TODO: communication

#### $C_B$

Some operations tend to have better efficiency when done in large volume (such
as all-reduce). So, some libraries allocate buffers to accumulate more tensors
on which to carry out the operations. These buffers can blow-up when model is
large. $C_B$ mitigates this by using a fixed sized buffers, even for larger
models. This strikes balance between memory requirements and compute efficiency.

#### $M_D$

Memory fragmentation is caused by overlaying the allocation of short lived
tensors (e.g.: activations, activation gradients) and long-lived tensors (e.g.:
activation checkpoints, parameter gradients). $M_D$ mitigates this by allocating
continuous memory for activation checkpoints and parameter gradients, and
copying them over, once they are produced.

### ZeRO and TP

> Note: authors use MP to name parallelisation of a single layer across
> processes (such as MegatronLM). IMHO this is now referred to as Tensor
> parallelisation (*TP*).

Though ZeRO itself doesn't do anything for TP (other than $P_a$), the authors
tested using ZeRO along TP. They were able to scale to much larger sizes than
before (from 16B to 170B), but the main highlight for me was the following:

- using TP frees up memory for using larger batch sizes
- small batch size in DP means: few computations for potentially slow
  synchronization of few samples
- using TP therefore enables to have more efficient DP as:
  - batch size can be larger
  - more computation can be done for given number of synchronization calls
  - synchronization calls are more efficient since they are done in bigger
    volume

### Evaluation

Unless otherwise stated, evaluation is done using ZeRO-DP Stage 2 ($P_{os+g}$)
plus ZeRO-R.

#### Comparison to previous SOTA -- MegatronLM

The authors show that within s single node ZeRO-DP achieves small 1-1.5x speed
up over just MegatronLM. However, once MegatronLM fails to fit the model within
a single node, internode communication heavily bottlenecks training. Meanwhile
with ZeRO-DP, TP processes always fits within a node and the speedup hovers
around 5x-10x over MegatronLM.

#### Scalability of ZeRO-DP

The authors also showed that observed Tflops scaled super-linearly (faster than
linearly) when adding GPUs. This super-linear scaling was achieved by being able
to fit larger batch sizes into each process, thereby making ZeRO-DP more
efficient.

#### Comparison to previous SOTA DP -- DDP

When compared to
[`DistributedDataParallel`](./distributed_training_with_pytorch.md), ZeRO-DP
sows that it can train (~10x) larger models with (~2x) larger throughput. DDP
trains model up to 1.4B, whereas ZeRO-DP can go up to 13B on 128GPUs.

#### Comparison of ZeRO configs (without Stage 3)

The biggest surprises are:
- $P_a$ is a big deal with TP and Stage 2 ZeRO-DP -- increase of max. model size
  from 60B to 140B 
- $P_{a+cpu}$ doesn't do anything noticeable if models are small -- it is only
  worth it for larger models
- as expected the decrease in memory requirements equals performance improvement
- as expected $P_{a+cpu}$ hurts performance compared to just $P_a$

