---
tags: [ml]
---
# Memory consumption of deep learning

A common question in deep learning (*DL*) is: "*How much memory do I need to
train this model?*" It stems from the complicated nature of low-level behaviour
DL training loops. Some of this complexity can be explained (this note) and the
rest has to be tested.

> This note is a summary of section 3 of [ZeRO paper](./zero.md).

## Objects we need to allocate

We need to allocate:

- **parameters** -- used for forward computation of the model
- **gradients** -- during backward pass, we compute the gradient:
$$
\frac{\partial f(x)}{\partial \omega}
$$

for each parameter $\omega$.
- **activations** -- to compute gradients we need to remember activations ($f(x)$ in
  the equation above). Note that this can be mitigated to an extent with
  [gradient checkpointing](./gradient_checkpointing.md)
- **optimizer states** -- each optimizer can hold several states for each parameter.
  For example [Adam](./adam.md) has the following 2 states for each parameter:
  - moving average of first momentum -- mean
  - moving average of second momentum -- non-centered variance
- **temporary buffers** -- we need to store intermediate results either during the
  forward computation or due to the implementation of the training loop (e.g.
  for distributed setups, it's worth it to reduce large *temporary* buffer of
  numbers at once).
- **memory fragmentation** -- though this is not something you need to store, memory
  fragmentation takes up memory. It is a direct result of different lifespans of
  allocated objects, e.g.:
  - temporary buffer -- exist for part of a forward pass,
  - activation checkpoint -- exists for entire forward pass
  - parameter -- exists for the entire training

So for a model with $\Psi$ parameters we need to store:
- $\Psi$ parameters
- $2\Psi$ optimizer states
- $\Psi$ gradients

throughout the training. This part can be explained and expected. However, the
rest cannot be easily predicted:
- activations -- for 1.5B GPT-2 model with sequence length of 1K, batch size of
  32 (small to modest hyperparameters) requires around 60GB of memory *just to
  store activations*. With [activation
  checkpointing](./gradient_checkpointing.md) its around 8GB.
- temporary buffers -- a buffer which stores all parameters of 1.5B GPT-2 model
  takes up 6GB. Such buffer may be needed if we reduce all gradients at the same
  time across processes.
- memory fragmentation -- in extreme cases, OOM errors can occur with 30% of
  memory still available

## Typical case

For large models, the majority of the memory is taken up by model states:
optimizer states, parameters and gradients. The rest is taken up by activations,
temporary buffers, and fragmented memory chunks.

## Mixed-precision training

With mixed-precision training we compute with half precision (so 16-bit)
labelled as *fp16*. However, to avoid large errors, the weights and optimizer
states are still stored as fp32 (so full precision). This means that to train a
model with $\Psi$ parameters using Adam we need (in bytes):
- $4\Psi$ -- fp32 parameters
- $2\Psi$ -- fp16 copy of parameters
- $2*4\Psi$ -- fp32 optimizer states
- $2\Psi$ -- fp16 gradients

This sums to $16\Psi$, for GPT-2 with $\Psi=1.5e^{9}$ (1.5B parameters) we get
24GB of required VRAM.
