---
tags: [math]
---

# Normal distribution

Normal distribution is a continuous probability distribution defined as:

$$
\mathcal{N}(\mu, \sigma^2) = \frac{1}{\sqrt{2 \pi \sigma^2}} \exp{\left(
  -\frac{
    (x - \mu)^2
  }{
    2\sigma^2
  }
\right)}
$$

where $\mu$ is mean and $\sigma^2$ is the variance.

## Sum of two normal distributions

The sum of independent random variables $x_1 \sim \mathcal{N}(\mu_1,
\sigma_1^2)$ and $x_2 \sim \mathcal{N}(\mu_2, \sigma_1^2)$ can be expressed as
another random variable:

$$
x = x_1 + x_2 \text{ where }
x \sim \mathcal{N}(\mu_1 + \mu_2, \sigma_1^2 + \sigma_2^2)
$$

In particular for $\mathbf{e}_1, \mathbf{e}_2 \sim \mathcal{N}\left(0,
\mathbf{I}\right)$:

$$
\sigma_1 \mathbf{e}_1 + \sigma_2 \mathbf{e}_2 = \sqrt{\sigma^2_1 +
\sigma^2_2}\mathbf{e}
$$

where $\mathbf{e} \sim \mathcal{N}\left(0, \mathbf{I}\right)$.
