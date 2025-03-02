---
tags: [ml, generative]
---

# Diffusion Probabilistic Model

Diffusion Probabilistic Model (*DPM*) is a probabilistic generative model. The
model was introduced by [Sohl-Dikstein et al.
(2015)](https://proceedings.mlr.press/v37/sohl-dickstein15.pdf) but the model
was improved multiple times
([DDPM](./denoising_diffusion_probabilistic_model.md), TOOD: more).

## Overview of how DPM work

DPM is a latent generative model. Similarly to
[VAE](./variation_autoencoders.md) it uses latent variables to generate samples.
Only in diffusion models, latents are noised versions of the desired output.
Latents are learned by a fixed (non-trainable) Markovian process, called **the
forward process**, that gradually adds more and more noise to the input. The
model learns an approximation of this process, which is called **the reverse
process**, where the noise is removed from the input.

### Training

There are two processes in training of a DPM:

- *a forward process* $q$ and
- *a reverse process*

Forward process adds noise to a training input. The reverse process tries to
remove the added noise. **The reverse process is modelled by DPM**.

Thanks to some nice math, the reverse process can be explicitly defined, which
gives us the *true* value, which we can contrast with the generated to train a
DPM.

#### Entropy of reverse process

Since the forward process is known, we can define lower and upper bounds on the
[conditional cross-entropy](./conditional_entropy.md) of a step in the reverse
direction.

We label the random variables for step $t$ as $\mathbf{X}_t$ and are interested in lower
and upper bounds for $H_q(\mathbf{X}_{t-1} | \mathbf{X}_t)$, where $H$ is [conditional
cross-entropy](./conditional_entropy.md).

##### Upper bound

To get the upper bound we realize that, we get $X_t$ by noising (increasing
entropy of) $X_{t-1}$:

$$
\begin{aligned}
H_q(\mathbf{X}_t) &\ge H_q(\mathbf{X}_{t-1}) \\
H_q(\mathbf{X}_t) - I(\mathbf{X}_t, \mathbf{X}_{t-1}) &\ge H_q(\mathbf{X}_{t-1}) - I(\mathbf{X}_t, X_{t-1})\\
H_q(\mathbf{X}_t | \mathbf{X}_{t-1}) &\ge H_q(\mathbf{X}_{t-1} | \mathbf{X}_t)\\
\end{aligned}
$$

where $I(\mathbf{X}_t, \mathbf{X}_{t-1})$ is [mutual
information](./mutual_information.md)

##### Lower bound

We arrive for lower bound with similar though process: steps of forward process
lower the amount of information in data and increase their entropy.

$$
\begin{align}
H_q(\mathbf{X}_{0} | \mathbf{X}_{t}) &\geq
  H_q(\mathbf{X}_{0} | \mathbf{X}_{t-1}) \\
H_q(\mathbf{X}_{t-1}) - H_q(\mathbf{X}_{t}) &\geq
  H_q(\mathbf{X}_{0} | \mathbf{X}_{t-1}) + H_q(\mathbf{X}_{t-1}) - H_q(\mathbf{X}_{0} | \mathbf{X}_{t}) - H_q(\mathbf{X}_{t}) \\
H_q(\mathbf{X}_{t-1}) - H_q(\mathbf{X}_{t}) &\geq
  H_q(\mathbf{X}_{0}, \mathbf{X}_{t-1}) - H_q(\mathbf{X}_{0}, \mathbf{X}_{t}) \\
H_q(\mathbf{X}_{t-1}) - H_q(\mathbf{X}_{t}) &\geq
  H_q(\mathbf{X}_{t-1} | \mathbf{X}_{0}) - H_q(\mathbf{X}_{t} | \mathbf{X}_{0}) \\
H_q(\mathbf{X}_{t-1}) - H_q(\mathbf{X}_{t}) + H_q(\mathbf{X}_t | \mathbf{X}_{t-1}) &\geq
  H_q(\mathbf{X}_t | \mathbf{X}_{t-1}) + H_q(\mathbf{X}_{t-1} | \mathbf{X}_{0}) - H_q(\mathbf{X}_{t} | \mathbf{X}_{0}) \\
H_q(\mathbf{X}_{t-1} | \mathbf{X}_{t}) &\geq
  H_q(\mathbf{X}_{t} | \mathbf{X}_{t-1}) + H_q(\mathbf{X}_{t-1} | \mathbf{X}_{0}) - H_q(\mathbf{X}_{t} | \mathbf{X}_{0})\\
H_q(\mathbf{X}_{t-1} | \mathbf{X}_{t}) &\geq
  H_q(\mathbf{X}_{t-1} | \mathbf{X}_t, \mathbf{X}_0)
\end{align}
$$

So the netropy of the reverse process, is lower if we also condition on
$\mathbf{X}_0$. If we condition on $\mathbf{X}_0$ we know the direction, where
we are heading, so naturally there should be less amount of uncertainty in
making the next step.

### Sampling

To sample from a DPM we run a random noise through the model multiple times,
until it removed all the noise.

## Where is the math

The paper includes the math, but I find that
[DDPM](./denoising_diffusion_probabilistic_model.md)'s paper explained it
better, so I include the math there.
