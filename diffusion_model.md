---
tags: [ml]
---

# Diffusion model in ML

Diffusion in ML is a way of generating data of arbitrary distribution. Even
though the concept is relatively simple, the math behind it makes it hard to
grasp. There are several papers on the subject:

1. [Deep Unsupervised Learning using Nonequilibrium Thermodynamics by
   Sohl-Dikstein et al.
   (2015)](https://proceedings.mlr.press/v37/sohl-dickstein15.pdf) -- applies an
   idea from physics to ML in order to learn arbitrary distribution using MLP
2. [Denoising Diffusion Probabilistic Models (DDPM) by Ho et al.
   (2020)](https://arxiv.org/pdf/2006.11239) -- applied Diffusion model to
   images and got competitive results
3. 


## High level intuition

From a high level, diffusion models attempt to capture *'noise' in a sample* (e.g.
image), such that if we subtract the predicted noise from the input, we should
move closer to the distribution in the training data.

To train diffusion models, there are two processes:

- forward: adds noise to the input image
- reverse: removes noise from the input image -- **this is what we want to model**

To reproduce the reverse process during inference, we want to parametrize it and
design a model that would predict those parameters.

Thanks to noice math, the reverse process can be explicitly defined in terms of
its parameters, which gives us the *true* values the model should predict. With
these in hand we can simply apply MSE on those predicted parameters and thats
it.

Moreover, we can **sample from the modelled distribution** by applying the reverse
process on a random noise.

## DDPM

Denoising Diffusion Probabilistic Model (DDPM) is the 'basic' diffusion variant.
It follows the High level intutition described above, which omits the following
details:

1. How does the forward process look like. What sort of noise are we adding?
2. How does the true reverse process look like?
3. What is the goal of the reverse process? What should the model parametrize
   and predict?
4. How does the loss look like?
5. How to put it all together & simplify?
6. How is the system trained? How do we generate new sample?

### Forward process

The forward process is Markovian process $q$ where for step $t = 1 \ldots T$, we
add a Gaussian noise with std. deviation $\beta_t$ to input $x_{t-1}$:

$$
\begin{aligned}
q(x_t | x_{t-1}) &= \mathcal{N}(x_t; \sqrt{\beta_t - 1}x_{t-1}, \beta_t\mathbf{I}) \\
&= \sqrt{\beta_t - 1} x_{t-1} + \mathbf{e}\sqrt{\beta_t} \text{ for } \mathbf{e}
\sim \mathcal{N}(0, \mathbf{I})
\end{aligned}
$$

To define how will the process noise the image after $t$ total steps we express
the last equation using $\alpha_t = 1-\beta_t$ and $\bar{\alpha_t} =
\prod_{i=1}^t \alpha_i$:

TODO: cleanup the math so it fits, from here on down

$$
\begin{aligned}
\mathbf{x}_t
&= \sqrt{\alpha_t}\mathbf{x}_{t-1} + \sqrt{1-\alpha_t}\mathbf{e}_t \\
&= \sqrt{\alpha_t}(\sqrt{\alpha_{t-1}}\mathbf{x}_{t-2} + \sqrt{1-\alpha_{t-1}}\mathbf{e}_{t-1}) + \sqrt{1-\alpha_t}\mathbf{e}_t \\
&= \sqrt{\alpha_t\alpha_{t-1}}\mathbf{x}_{t-2} + \sqrt{\alpha_t(1-\alpha_{t-1}) + (1-\alpha_t)}\mathbf{e}_{t-1} \\
&= \sqrt{\alpha_t\alpha_{t-1}}\mathbf{x}_{t-2} + \sqrt{1-\alpha_t\alpha_{t-1}}\mathbf{e}_{t-1} \\
&= \sqrt{\alpha_t\alpha_{t-1}\alpha_{t-2}}\mathbf{x}_{t-3} + \sqrt{1-\alpha_t\alpha_{t-1}\alpha_{t-2}}\mathbf{e}_{t-2} \\
&= \cdots \\
&= \sqrt{\bar{\alpha_t}}\mathbf{x}_0 + \sqrt{1-\bar{\alpha_t}}\mathbf{e}_0
\end{aligned}
$$

This means that after $t$ steps $x_t$ is still **normally distributed**:

$$
q(x_t | x_0) = \mathcal{N}(\sqrt{\bar{\alpha_t}}\mathbf{x}_0, 1-\bar{\alpha_t} \mathbf{I})
$$

If $\bar{\alpha_t} \rightarrow 0$ as $t \rightarrow \inf$ then $x_t \sim
\mathcal{N}(0, \mathbf{I})$. This is cruicial as when we generate completely new
sample, we need to start with something **that we know** but also is **not part
of the training dataset**. That something is $\mathbf{e} \sim \mathcal{N}(0,
\mathbf{I})$.

The above also justifies the definition of $q(x_t | x_{t-1})$ above.

### Reverse process

If $\beta_t$ is sufficiently small the reverse of $q(x_t | x_{t-1})$, which we
label as $p_\theta$ is also a normal distribution:

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
\begin{aligned}
q(\mathbf{x}_{t-1}|\mathbf{x}_t, \mathbf{x}_0)
&= q(\mathbf{x}_t|\mathbf{x}_{t-1}, \mathbf{x}_0)\frac{q(\mathbf{x}_{t-1}|\mathbf{x}_0)}{q(\mathbf{x}_t|\mathbf{x}_0)} \\
&\propto \exp\left(-\frac{1}{2}\left(\frac{(\mathbf{x}_t-\sqrt{\alpha_t}\mathbf{x}_{t-1})^2}{\beta_t} + \frac{(\mathbf{x}_{t-1}-\sqrt{\bar{\alpha}_{t-1}}\mathbf{x}_0)^2}{1-\bar{\alpha}_{t-1}} - \frac{(\mathbf{x}_t-\sqrt{\bar{\alpha}_t}\mathbf{x}_0)^2}{1-\bar{\alpha}_t}\right)\right) \\
&= \exp\left(-\frac{1}{2}\left(\frac{\mathbf{x}_t^2-2\sqrt{\alpha_t}\mathbf{x}_t\mathbf{x}_{t-1}+\alpha_t\mathbf{x}_{t-1}^2}{\beta_t} + \frac{\mathbf{x}_{t-1}^2-2\sqrt{\bar{\alpha}_{t-1}}\mathbf{x}_{t-1}\mathbf{x}_0+\bar{\alpha}_{t-1}\mathbf{x}_0^2}{1-\bar{\alpha}_{t-1}} + \cdots\right)\right) \\
&= \exp\left(-\frac{1}{2}\left(\left(\frac{\alpha_t}{\beta_t} + \frac{1}{1-\bar{\alpha}_{t-1}}\right)\mathbf{x}_{t-1}^2 - 2\left(\frac{\sqrt{\alpha_t}}{\beta_t}\mathbf{x}_t + \frac{\sqrt{\bar{\alpha}_{t-1}}}{1-\bar{\alpha}_{t-1}}\mathbf{x}_0\right)\mathbf{x}_{t-1} + \cdots\right)\right)
\end{aligned}
$$

From this we derive $ q(\mathbf{x}_{t-1}|\mathbf{x}_t, \mathbf{x}_0) =
\mathcal{N}(x_{t-1}; \mathbf{\mu_\theta}(x_t, x_0), \tilde{\beta_t}\mathbf{I}$,
where:

$$
\begin{aligned}
\tilde{\beta}_t &= 1/\left(\frac{\alpha_t}{\beta_t} + \frac{1}{1-\bar{\alpha}_{t-1}}\right) = 1/\left(\frac{\alpha_t(1-\bar{\alpha}_{t-1})+\beta_t}{\beta_t(1-\bar{\alpha}_{t-1})}\right) = 1/\left(\frac{\alpha_t+\beta_t-\bar{\alpha}_t}{\beta_t(1-\bar{\alpha}_{t-1})}\right) = \frac{1-\bar{\alpha}_{t-1}}{1-\bar{\alpha}_t}\beta_t, \\
\tilde{\mu}_t(\mathbf{x}_t, \mathbf{x}_0) &= \left(\frac{\sqrt{\alpha_t}}{\beta_t}\mathbf{x}_t + \frac{\sqrt{\bar{\alpha}_{t-1}}}{1-\bar{\alpha}_{t-1}}\mathbf{x}_0\right)\frac{1-\bar{\alpha}_{t-1}}{1-\bar{\alpha}_t}\beta_t = \frac{\sqrt{\bar{\alpha}_{t-1}}\beta_t}{1-\bar{\alpha}_t}\mathbf{x}_0 + \frac{\sqrt{\alpha_t}(1-\bar{\alpha}_{t-1})}{1-\bar{\alpha}_t}\mathbf{x}_t.
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

### Training and sampling

TODO: rewrite from slides


