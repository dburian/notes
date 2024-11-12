---
tags: [ml]
---
# Adam

Adam is an algorithm to iteratively optimize neural networks through gradient
decent. Introduced by [Kingma et al. (2014)](https://arxiv.org/pdf/1412.6980).

The idea of Adam is to:

- avoid zigzag gradients by using an '*average*' direction of gradient
- avoid exploding gradients by normalizing by an '*average*' gradient norm

The first idea is also known as *momentum* and has been applied to gradient
optimizers before Adam.

## The update

To obtain an average direction and size of gradient, Adam keeps exponential
moving averages of the first (mean) moment and the second raw (uncentered
variance) moment:

$$
\begin{align}
  m_t &= \beta_1 m_{t - 1} + (1 - \beta_1)g_t \\
  v_t &= \beta_2 v_{t - 1} + (1 - \beta_2)g_t^2
\end{align}
$$

where:
- the initial averages are initialized to 0: $m_0 = v_0 = 0$
- the default values for betas are: $\beta_1 = 0.9$ and $\beta_2 = 0.999$
    - note that the exponential average of the second moment 'forgets' more
      slowly -- is biased more towards history

If we compute the exponential moving averages at time step $t$ we notice that
the 'average' is not really an average. If we take an expectation across
gradients, and compute $m_t$ (it's the same as for $v_t$):

$$
\begin{align}
m_t &= \mathbb{E}_{g \sim p(g)}\left[
  \beta_1^{t - 1}(1 - \beta_1)g_1 + \beta_1^{t - 2}(1 - \beta_1)g_2 + \ldots +
  (1 - \beta_1)g_t
\right] \\
&= \mathbb{E}_{g \sim p(g)}[(1 - \beta_1)\sum_{i = 1}^t \beta_1^{t - i} g_i] \\
&= \mathbb{E}_{g \sim p(g)}[g] (1 - \beta_1)\sum_{i = 1}^t \beta_1^{t - i} \\
&= \mathbb{E}_{g \sim p(g)}[g] (1 - \beta_1^t) \\
\end{align}
$$

We see that $m_t$ is $(1 - \beta_1^t)$ larger than an average gradient. Ergo we
must correct our exponential moving averages. The total update is then as
follows:

$$
\begin{aligned}
\hat{m}_t &= \frac{m_t}{1 - \beta_1^t} \\
\hat{v}_t &= \frac{v_t}{1 - \beta_2^t} \\
\theta_{t+1} &= \theta_t - \alpha \frac{\hat{m}_t}{\sqrt{\hat{v}_t} + \epsilon}
\end{aligned}
$$

The $\epsilon$ is there only due to numerical stability, avoiding huge steps
when gradient is very close to 0 (by default $\epsilon = 10^{-8}$).
