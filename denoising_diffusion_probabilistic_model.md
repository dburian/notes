---
tags: [ml, generative]
---
# DDPM

Denoising Diffusion Probabilistic Model (*DDPM*) is a [Diffusion Probabilistic
Model (*DPM*)](./diffusion_probabilistic_model.md) model introduced by [Ho et
al. (2020)](https://arxiv.org/pdf/2006.11239).

DDPM model follows the high-level intuition from
[DPM](./diffusion_probabilistic_model.md), but the authors connect DPM with
score-based models (TODO: note), while being able to produce much better looking
images - basically fine-tuning the method.

It follows the high level intuition from ['Deep Unsupervised Learning using
Nonequilibrium Thermodynamics](./diffusion_probabilistic_model.md), specifies the
following details:

1. How does the forward process look like. What sort of noise are we adding?
2. How does the true reverse process look like?
3. What is the goal of the reverse process? What should the model parametrize
   and predict?
4. How does the loss look like?
5. How to put it all together & simplify?
6. How is the system trained? How do we generate new sample?

## Training

Same as [DPM](./diffusion_probabilistic_model.md), DDPM is trained by reversing
a forward process, which adds noise to a training input.

### Forward process

The forward process is Markovian process $q$ where for step $t = 1 \ldots T$, we
add a Gaussian noise with std. deviation $\beta_t$ to input $x_{t-1}$:

$$
\begin{aligned}
q(\mathbf{x}_t | \mathbf{x}_{t-1}) &= \mathcal{N}\left(
  \mathbf{x}_t;
  \sqrt{\beta_t - 1}\mathbf{x}_{t-1},
  \beta_t\mathbf{I}
\right) \\
&= \sqrt{\beta_t - 1} \mathbf{x}_{t-1} + \mathbf{e}\sqrt{\beta_t}
  \text{ for } \mathbf{e} \sim \mathcal{N}(0, \mathbf{I})
\end{aligned}
$$

We can express $q$ even after $t$ steps of forward process (not just one) using:
$\alpha_t = 1-\beta_t$, $\bar{\alpha_t} = \prod_{i=1}^t \alpha_i$, and
$\mathbf{e}_t, \mathbf{\bar{e}}_t \sim \mathcal{N}(0, \mathbf{I})$:

$$
\begin{aligned}
\mathbf{x}_t
&= \sqrt{\alpha_t}\mathbf{x}_{t-1} + \sqrt{1-\alpha_t}\mathbf{e}_t \\
&= \sqrt{\alpha_t}\left(\sqrt{\alpha_{t-1}}\mathbf{x}_{t-2} +
  \sqrt{1-\alpha_{t-1}}\mathbf{e}_{t-1}\right) + \sqrt{1-\alpha_t}\mathbf{e}_t \\
&= \sqrt{\alpha_t\alpha_{t-1}}\mathbf{x}_{t-2} +
  \sqrt{\alpha_t(1-\alpha_{t-1}) + (1-\alpha_t)}\mathbf{e}_{t-1} \\
\end{aligned}
$$

since [the sum of normally distributed variables is also a normally distributed
variable with variance equal to sum of variables'
variances](./normal_distribution.md). Continuing:

$$
\begin{aligned}
&= \sqrt{\alpha_t\alpha_{t-1}}\mathbf{x}_{t-2} +
  \sqrt{1-\alpha_t\alpha_{t-1}}\mathbf{e}_{t-1} \\
&= \sqrt{\alpha_t\alpha_{t-1}\alpha_{t-2}}\mathbf{x}_{t-3} +
  \sqrt{1-\alpha_t\alpha_{t-1}\alpha_{t-2}}\mathbf{e}_{t-2} \\
&= \cdots \\
&= \sqrt{\bar{\alpha_t}}\mathbf{x}_0 + \sqrt{1-\bar{\alpha_t}}\mathbf{e}_0
\end{aligned}
$$

So even after $t$ steps, $q$ is **normally distributed**:

$$
q(\mathbf{x}_t | \mathbf{x}_0) = \mathcal{N}\left(
  \sqrt{\bar{\alpha_t}}\mathbf{x}_0,
  1-\bar{\alpha_t} \mathbf{I}
\right)
$$

Moreover, if $\bar{\alpha_t} \rightarrow 0$ as $t \rightarrow \inf$ then $x_t \sim
\mathcal{N}(0, \mathbf{I})$. This property will be handy when sampling, since we
can just start with $x_T \sim \mathcal{N}(0, \mathbf{I})$ and the model should
gradually de-noise it to something similar to a training sample. 

We could have labelled mean and variance in any way. The peculiar and the thing
to notice is that their definitions are tied. And only thanks to these tied
definitions, $q(\mathbf{x}_T | \mathbf{x}_0)$ can be defined so simply and has
the above mentioned properties.

### Reverse process

If $\beta_t$ is sufficiently small the reverse of $q(x_t | x_{t-1})$, which we
label as $p_\theta$ is also a normal distribution (proven by [Feller
(1949)](https://projecteuclid.org/ebooks/berkeley-symposium-on-mathematical-statistics-and-probability/On-the-Theory-of-Stochastic-Processes-with-Particular-Reference-to/chapter/On-the-Theory-of-Stocha%20stic-Processes-with-Particular-Reference-to/bsmsp/1166219215)):

$$
p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t) =
\mathcal{N}(
  \mathbf{x}_{t-1};
  \mu_\theta(\mathbf{x}_t,t),
  \sigma^2_t\mathbf{I}
)
$$

#### The true reverse process

So essentially we need to find the correct values for $\mu_\theta$ and
$\sigma_t$. We do so, by trying to find reverse of $q(x_t | x_{t-1})$. However,
this reverse is only tractable if we condition it on the original sample $x_0$:


$$
\begin{align}
& q(\mathbf{x}_{t-1}|\mathbf{x}_t, \mathbf{x}_0) =\\
&= q(\mathbf{x}_{t-1}, \mathbf{x}_t | \mathbf{x}_0)
\frac{1}{q(\mathbf{x}_t | \mathbf{x}_0)}
\\
&= q(\mathbf{x}_t | \mathbf{x}_{t-1}, \mathbf{x}_0)
\frac{
    q(\mathbf{x}_{t-1} | \mathbf{x}_0)
  }{
    q(\mathbf{x}_t | \mathbf{x}_0)
  }\\
&= q(\mathbf{x}_t | \mathbf{x}_{t-1})
\frac{
    q(\mathbf{x}_{t-1} | \mathbf{x}_0)
  }{
    q(\mathbf{x}_t | \mathbf{x}_0)
  }\\
\end{align}
$$

These are just normal distributions we defined above, where the last equation
holds since $q(x_0, x_t | x_{t-1}) = q(x_t | x_{t-1})$ ($q(x_t)$ is either
defined using $x_{t-1}$ or $x_0$, both are not needed). Expending the definition
of [Normal distribution](./normal_distribution.md) and leaving out the
normalization constant we get:

$$
\begin{align}
& \textcolor{purple}{q(\mathbf{x}_t | \mathbf{x}_{t-1})}
\frac{
    \textcolor{red}{q(\mathbf{x}_{t-1} | \mathbf{x}_0)}
  }{
    \textcolor{green}{q(\mathbf{x}_t | \mathbf{x}_0)}
  }\\
&\begin{multlined}[t]
  \propto \exp \bigg(-\frac{1}{2}\bigg[
      \color{purple}
      \frac{
        (\mathbf{x}_t-\sqrt{\alpha_t}\mathbf{x}_{t-1})^2
      }{
        \beta_t
      } +
      \color{red}
      \frac{
        (\mathbf{x}_{t-1}-\sqrt{\bar{\alpha}_{t-1}}\mathbf{x}_0)^2
      }{
        1-\bar{\alpha}_{t-1}
      }\\
      \color{green}
      - \frac{
        (\mathbf{x}_t-\sqrt{\bar{\alpha}_t}\mathbf{x}_0)^2
      }{
        1-\bar{\alpha}_t
      }
    \bigg]\bigg)
\end{multlined} \\
&\begin{multlined}[t]
= \exp\bigg(-\frac{1}{2}\bigg[
      \frac{
        \mathbf{x}_t^2-
        2\sqrt{\alpha_t}\mathbf{x}_t\mathbf{x}_{t-1}+
        \alpha_t\mathbf{x}_{t-1}^2
      }{
        \beta_t
      } + \\
      \frac{
        \mathbf{x}_{t-1}^2-2\sqrt{\bar{\alpha}_{t-1}}\mathbf{x}_{t-1}\mathbf{x}_0+\bar{\alpha}_{t-1}\mathbf{x}_0^2
      }{
        1-\bar{\alpha}_{t-1}
      } +
      \cdots
  \bigg]
\bigg)
\end{multlined}\\
&\begin{multlined}[t]
= \exp\bigg(-\frac{1}{2}\bigg[
    \left(
      \frac{\alpha_t}{\beta_t} +
      \frac{1}{1-\bar{\alpha}_{t-1}}
    \right) \mathbf{x}_{t-1}^2 - \\
    2\left(
      \frac{ \sqrt{\alpha_t}}{\beta_t}\mathbf{x}_t +
      \frac{\sqrt{\bar{\alpha}_{t-1}}}{1-\bar{\alpha}_{t-1}}\mathbf{x}_0
    \right)\mathbf{x}_{t-1} +
    \cdots
  \bigg]
\bigg)
\end{multlined}
\end{align}
$$

From this we derive $ q(\mathbf{x}_{t-1}|\mathbf{x}_t, \mathbf{x}_0) =
\mathcal{N}\left(x_{t-1}; \mathbf{\mu_\theta}(x_t, x_0),
\tilde{\beta_t}\mathbf{I}\right)$,
where:

TODO: cleanup the math so it fits, from here on down

$$
\begin{aligned}
\tilde{\beta}_t &= \left(\frac{\alpha_t}{\beta_t} + \frac{1}{1-\bar{\alpha}_{t-1}}\right)^{-1}\\
&= \left(\frac{\alpha_t(1-\bar{\alpha}_{t-1})+\beta_t}{\beta_t(1-\bar{\alpha}_{t-1})}\right)^{-1}\\
&= \left(\frac{\alpha_t+\beta_t-\bar{\alpha}_t}{\beta_t(1-\bar{\alpha}_{t-1})}\right)^{-1} \\
&= \frac{1-\bar{\alpha}_{t-1}}{1-\bar{\alpha}_t}\beta_t\\
\end{aligned}
$$

$$
\begin{aligned}
\tilde{\mu}_t(\mathbf{x}_t, \mathbf{x}_0) =
\left(\frac{\sqrt{\alpha_t}}{\beta_t}\mathbf{x}_t + \frac{\sqrt{\bar{\alpha}_{t-1}}}{1-\bar{\alpha}_{t-1}}\mathbf{x}_0\right)\frac{1-\bar{\alpha}_{t-1}}{1-\bar{\alpha}_t}\\
\end{aligned}
$$
$$
\begin{aligned}
\beta_t &= \frac{\sqrt{\bar{\alpha}_{t-1}}\beta_t}{1-\bar{\alpha}_t}\mathbf{x}_0 + \frac{\sqrt{\alpha_t}(1-\bar{\alpha}_{t-1})}{1-\bar{\alpha}_t}\mathbf{x}_t
\end{aligned}
$$

### Loss

As with all ML models, the goal is to maximize the likelihood of the training
data. We simplify using [Jensen's
inequality](https://en.wikipedia.org/wiki/Jensen's_inequality).

$$
\begin{aligned}
-\mathbb{E}_{q(\mathbf{x}_0)}[\log p_\theta(\mathbf{x}_0)] &= -\mathbb{E}_{q(\mathbf{x}_0)}[\log\mathbb{E}_{p_\theta(\mathbf{x}_{1:T})}[p_\theta(\mathbf{x}_0|\mathbf{x}_{1:T})]] \\
&= -\mathbb{E}_{q(\mathbf{x}_0)}\left[\log\mathbb{E}_{q(\mathbf{x}_{1:T}|\mathbf{x}_0)}\left[\frac{p_\theta(\mathbf{x}_{0:T})}{q(\mathbf{x}_{1:T}|\mathbf{x}_0)}\right]\right] \\
&\leq -\mathbb{E}_{q(\mathbf{x}_{0:T})}\left[\log\frac{p_\theta(\mathbf{x}_{0:T})}{q(\mathbf{x}_{1:T}|\mathbf{x}_0)}\right] = \mathbb{E}_{q(\mathbf{x}_{0:T})}\left[\log\frac{q(\mathbf{x}_{1:T}|\mathbf{x}_0)}{p_\theta(\mathbf{x}_{0:T})}\right] \\
&= \mathbb{E}_{q(\mathbf{x}_{0:T})}\left[-\log p_\theta(\mathbf{x}_T) + \sum_{t=2}^T\log\frac{q(\mathbf{x}_t|\mathbf{x}_{t-1})}{p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t)} + \log\frac{q(\mathbf{x}_1|\mathbf{x}_0)}{p_\theta(\mathbf{x}_0|\mathbf{x}_1)}\right] \\
&= \mathbb{E}_{q(\mathbf{x}_{0:T})}\left[-\log p_\theta(\mathbf{x}_T) + \sum_{t=2}^T\log\left(\frac{q(\mathbf{x}_{t-1}|\mathbf{x}_t,\mathbf{x}_0)}{p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t)}\cdot\frac{q(\mathbf{x}_t|\mathbf{x}_0)}{q(\mathbf{x}_{t-1}|\mathbf{x}_0)}\right) + \log\frac{q(\mathbf{x}_1|\mathbf{x}_0)}{p_\theta(\mathbf{x}_0|\mathbf{x}_1)}\right] \\
&= \mathbb{E}_{q(\mathbf{x}_{0:T})}\left[-\log p_\theta(\mathbf{x}_T) + \sum_{t=2}^T\log\frac{q(\mathbf{x}_{t-1}|\mathbf{x}_t,\mathbf{x}_0)}{p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t)} + \log\frac{q(\mathbf{x}_T|\mathbf{x}_0)}{q(\mathbf{x}_1|\mathbf{x}_0)} + \log\frac{q(\mathbf{x}_1|\mathbf{x}_0)}{p_\theta(\mathbf{x}_0|\mathbf{x}_1)}\right] \\
&= \mathbb{E}_{q(\mathbf{x}_{0:T})}\left[\log\frac{q(\mathbf{x}_T|\mathbf{x}_0)}{p_\theta(\mathbf{x}_T)} + \sum_{t=2}^T\log\frac{q(\mathbf{x}_{t-1}|\mathbf{x}_t,\mathbf{x}_0)}{p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t)} - \log p_\theta(\mathbf{x}_0|\mathbf{x}_1)\right] \\
&= \mathbb{E}_{q(\mathbf{x}_{0:T})}\left[\underbrace{D_{KL}(q(\mathbf{x}_T|\mathbf{x}_0)\|p_\theta(\mathbf{x}_T))}_{L_T} + \sum_{t=2}^T\underbrace{D_{KL}(q(\mathbf{x}_{t-1}|\mathbf{x}_t, \mathbf{x}_0)\|p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t))}_{L_t} - \underbrace{\log p_\theta(\mathbf{x}_0|\mathbf{x}_1)}_{L_0}\right]
\end{aligned}
$$

From these terms:

- $L_T$ is constant w.r.t. to $\theta$, since $q$ is fixed, and $x_T \sim
  \mathcal{N}(0, \mathbf{I})$
- $L_t = D_{KL}(q(\mathbf{x}_{t-1}|\mathbf{x}_t, \mathbf{x}_0)\|p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t))$ can be expressed analytically as:

- $L_0 = - \log p_\theta(x_0 | x_1)$ -- we ignore for now (TODO)

### Simplifying loss

TODO: express the loss for noise-producing model instead of $\mu$-producing
model

## Training and sampling

TODO: rewrite from slides


