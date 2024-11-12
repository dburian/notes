---
tags: [ml]
---

# LAMB optimizer

LAMB optimizer, introduced by [You et al.
(2019)](https://www2.eecs.berkeley.edu/Pubs/TechRpts/2019/EECS-2019-103.pdf) is
optimizer targeted at large batches.

## TLDR

I think of LAMB as [Adam](./adam.md), with (almost) **gradient clipping** and a
trick to make update to each parameter proportional to it.

## Update intuition

LAMB is basically [Adam](./adam.md), except it **doesn't correct** its estimates
of first and second gradient moments and it has few tricks:

1. normalize gradients per each layer
2. scale the gradient per parameter with a scaling function $\phi(|\theta_i|)$

Since the gradients are scaled (both by the normalization and the scaling), the
corrections of first and second moment estimates are not needed (they only scale
the gradient).

The authors argue that the normalization fights exploding gradients. However,
this the same motivation as for normalization by the second moment (which LAMB
also does). My intuition is that normalization by second moment does this on
*average*, while normalization of the gradient does this always. So I think of
it as a *hard-coded* protection against exploding gradients, much as **gradient
clipping**.

The normed gradient is later scaled per-element, where the scaling is
conditioned on the size of the gradient's corresponding parameter $\theta_i$.
This ensures that the size of the gradient is of the same order as the
parameter, which the authors found *speeds up* convergence for deep nets.

