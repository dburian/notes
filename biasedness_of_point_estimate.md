---
tags: [statistics]
---

# Biasedness of a point estimate

We say that a [point estimate](./point_estimate.md) of parameter of a population
is unbiased if its mean equals the true value of the parameter.

Formally, let's have point estimate $\hat{\theta}_n$ based on random samples
$X_1, X_2, \ldots, X_n$ of a parameter $\theta$. $\hat{\theta}_n$ is unbiased
if:

$$
\mathbb{E}_{X_1, X_2, \ldots, X_n} \big[\hat{\theta}_n\big] = \theta
$$
