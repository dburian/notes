---
tags: [ml]
---

# Generative Adversial Networks

Generative Adversial Networks (or GANs) are generative models introduced by
[Goodfellow et al. (2014)](https://arxiv.org/pdf/1406.2661). GAN is composed
of two models: generator and discriminator. Generator predicts samples, which
discriminator then asses if they are real or fake.

## Two models

Generator generates samples from a random latent variable $z$: $\hat{x} = G(z, \theta_G)$.
Alternatively, we might want to give the generator label information $y$,
specifying which sample to generate. Then we call the model **conditional GAN**.

Discriminator takes in a sample $x$ and classifies it as true (coming from
training data) or false (coming from the generator $G$): $D(x) \in \{0, 1\}$.
For conditional GAN, $D$ would be also given the label $y$ and predicting if
$y$ belongs to $x$ or don't (which includes the case where $x$ is fake).

## Training

The two networks take turns in trainings:

1. First, discriminator $D$ gets true samples, and fake ones. It is trained
   until it gathers some knowledge and is +- reliably (e.g. up to 80%) determine
   if the sample is fake or not. Only **discriminator's** parameters are trained.

$$
\mathcal{L}_D(x, \theta_D) =
-\mathbb{E}_{x \sim P_{\text{data}}} \left[\log D(x)\right]
-\mathbb{E}_{z \sim P(z)} \left[1 - \log D(G(z))\right]
$$

2. Then, generator is trained to generate samples that would fool the trained
   discriminator. Here $D$ works like a hint for $G$. It practically says which
   $G$'s output should be changed and how, so that the output is less
   recognizable.

$$
\mathcal{L}_G(z, \theta_G) = \mathbb{E}_{z \sim P(z)} \left[1 - \log D(G(z))\right]
$$

### Diminishing gradients

Although this was the original formulation of GAN (or *Adversial*) loss, it has
a problem with diminishing gradients. As the generator tries to minimize its
loss, the gradient of, what is essentially a $-\log x$ flattens. Meaning its
harder and harder to make meaningful steps and the training halts. To that end,
oftentimes the losses are formulated with MSE loss:

$$
\begin{align}
\mathcal{L}_D(x, \theta_D) &=
\mathbb{E}_{x \sim P_{\text{data}}} (D(x))^2
+ \mathbb{E}_{z \sim P(z)} (1 - D(G(z)))^2 \\
\mathcal{L}_G(z, \theta_G) &= \mathbb{E}_{z \sim P(z)} (1 - D(G(z)))^2
\end{align}
$$

This formulation was introduced by [Mao et al.
(2017)](https://openaccess.thecvf.com/content_ICCV_2017/papers/Mao_Least_Squares_Generative_ICCV_2017_paper.pdf).

## Problems of GANs

The two problems of GANs are that:

1. SGD is not guaranteed to reach optimum solution. The two trainings can be
   optimizing away (successfully), yet the quality of the generated images might
   not be improving.
2. Generators might suffer from "mode collapse". Its when the generator focuses
   only on one class of samples, the discriminator is particularly bad at. When
   the discriminator catches on, the generator might switch to new class samples
   and the whole cycle repeats. This problem could be partially solved by using
   an ensemble of discriminators, perhaps history of them.

Even though GANs generated higher quality samples than [Variational
Autoencoders](./variation_autoencoders.md), due to these problems VAEs were the
way to go to generate samples. However, eventually, [Diffusion
models](./diffusion_probabilistic_model.md) took over.


