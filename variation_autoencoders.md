---
tags: [ml, generative_models]
---

# Variation AutoEncoders

Variation AutoEncoder (VAE) is a generative neural model introduced by [Kingma
et al. (2013)](https://arxiv.org/pdf/1312.6114).

VAEs are [autoencoders](./autoencoder.md). There are two key ideas of VAEs:

- having the encoder predict a **distribution** of the latent variable, rather than its
  concrete value and
- is to force a known distribution $P(z)$ of the latent variable $z$.

In standard autoencoders, there is a latent $z$ for every input $x$ (the encoder
always produces a value). In VAE's, with the latent distribution known, each $z
\sim P(z)$ represents a training input (the decoder always produces meaningful input).

This aspect of VAEs makes them appealing as we can indefinitely sample from the
**known** distribution of $z$ and get different, but meaningful prediction. When
I say *meaningful* I mean a prediction that faithfully reflects training data's
distribution.

## Problems

There are quite a few problems that have to be solved to make the idea of VAEs
work.

### Loss function

The typical function to maximize during an autoencoder training, looks like so:

$$
\log P_\theta(x) =
\log \sum_z P(z) P_\theta(x | z) =
\log \mathbb{E}_{z \sim P(z)} P_\theta(x | z)
$$

We want to maximize the log likelihood of producing input x, based on our latent
variable $z$. Since $z$ is produced by an encoder $Q_\varphi$ we get:

$$
\begin{aligned}
  \log P_\theta (x) &= \log \mathbb{E}_{P(z)} \left[ P_\theta (x|z) \right] \\
  &= \log \mathbb{E}_{Q_\varphi (z|x)} \left[ P_\theta (x|z) \cdot \frac{P(z)}{Q_\varphi (z|x)} \right] \\
  &\geq \mathbb{E}_{Q_\varphi (z|x)} \left[ \log P_\theta (x|z) + \log \frac{P(z)}{Q_\varphi (z|x)} \right] \\
\end{aligned}
$$

where the inequality stems from the fact that logarithm is convex function and therefore [Jensen's
inequality](https://en.wikipedia.org/wiki/Jensen's_inequality) holds. This
allows us to move the expectation before everything else and approximate the
expression with a single training sample/mini-batch.

We can rearrange the probabilities and formulate the final loss function
$-\mathcal{L}(\varphi, \theta, x)$ as:

$$
\begin{aligned}
\mathcal{L}(\theta, \varphi; \mathbf{x})
&= \mathbb{E}_{Q_\varphi (z|x)} \left[ \log P_\theta (x|z) + \log P(z) - \log Q_\varphi (z|x) \right] \\
&= \mathbb{E}_{Q_\varphi (z|x)} \left[ \log P_\theta (x|z) \right] - D_{KL} \left( Q_\varphi (z|x) \| P(z) \right),
\end{aligned}
$$

where $D_{KL}$ is [Kullback-Leibler
divergence](./kullback_leibler_divergence.md). This divides VAE's loss to two
components:

1. the reconstruction loss (the expectation) -- enforcing high likelihood of
   predicting input $x$
2. the latent loss (the KL divergence) -- enforcing encoder to produce the same
   distribution as we've pre-determined

### Choosing the known $P(z)$

Don't forget the encoder in VAE predicts distribution, not a concrete $z$. This
means we need to define the known distribution $P(z)$ such that:

1. we can parametrize it.
1. we can compute its KL divergence,
1. we can sample from it effectively,
1. we can sample from it in a differentiable way,

Leaving the last requirement for later, we can use normal distribution. We can
parametrize it by letting the encoder predict $\sigma^2$ and $\mu$, the sampling
is effective and KL divergence between two normal distributions can be defined
as:

$$
D_{KL}(P, Q) = \log \frac{\sigma_2}{\sigma_1} + \frac{\sigma_1^2 + (\mu_1 - \mu_2)^2}{2 \sigma_2^2} - \frac{1}{2}
$$


### The reparametrization trick

The authors of the mentioned paper also came with a novel approach that let's
sample normal distribution in a differentiable way. Instead of sampling as so:

$$
z \sim \mathcal{N}(\mu, \sigma^2)
$$

We sample as so:
$$
z \sim \mu + \sigma \cdot \mathcal{N}(0, 1)
$$

The result will be differentiable w.r.t. both $\mu$ and $\sigma^2$.

## Conditional VAEs

Often we'd like to say what samples to generate. The solution is to pass the
sample class both to the encoder and the decoder. We then maximize the
log likelihood $\log P_\theta(x|c)$, for class $c$:

$$
\mathcal{L}(\theta, \varphi; \mathbf{x}, c) =
\mathbb{E}_{Q_\varphi (z|x)} \left[
  \log P_\theta (x|z)
\right] - D_{KL} \left(
  Q_\varphi (z|x) \| P(z|c)
\right),
$$

---
More resources:

- [In-depth theoretical look at conditional VAEs](https://beckham.nz/2023/04/27/conditional-vaes.html)
- [Overview of generative models, with focus on VAEs](https://www.youtube.com/watch?v=iL1c1KmYPM0)

