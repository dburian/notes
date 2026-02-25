---
tags: [ml, generative]
---

# Sliced score matching

Makes [score matching](./score_matching.md) tractable for models with high
parameter count by avoiding the computation of a trace of an Hessian matrix. The
idea is to project the gradients of log densities using random vectors and
optimize the difference of the projections.

In practice the authors found that using one random vector gives enough guidance
to the back-propagation optimization.
