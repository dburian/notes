---
tags: [ml, generative]
---

# Score matching

Score matching is built on the idea that model should match a score of the data
-- a log of density (non-normalized probability) of the data.

The trick is that we try to estimate the gradient of the log density,
rather than the probability. This enables estimating the un-normalized
probability density, cause when written down, optimizing the gradient of the log
density, we don't have to know the normalization constant, which is in many
cases intractable.

The introductory paper (TODO), enables use of score matching for basic models
such as multivariate gaussian. For models w/ high parameter count, score
matching is untractable.

[Sliced score matching](./sliced_score_matching.md) solves this by avoiding the
computation of a trace of hessian matrix.

