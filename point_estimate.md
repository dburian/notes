---
tags: [statistics]
---

# Point estimate

Point estimate is a single value that serves as an estimate or approximation of
a population's parameter. The value is produced by a point estimator.

Formally, point estimator of a parameter $\theta$ of a population with
distribution $F_X$ is a function $g: \mathbb{R}^{n} \to \Theta$ whose definition
is **independent** on the distribution $F_X$ nor of its parameters. The value
$\hat{\theta} := g(x)$ is called point estimate.

Point estimate is itself a random variable, that is dependent on $F_X$, but as
the definition above says, the dependency is not direct. Typically, the point
estimator's definition would be dependent on a sample of size $n$.
