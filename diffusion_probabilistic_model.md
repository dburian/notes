---
tags: [ml, generative]
---

# Diffusion Probabilistic Model

Diffusion Probabilistic Model (*DPM*) is a probabilistic generative model. The
model was introduced by [Sohl-Dikstein et al.
(2015)](https://proceedings.mlr.press/v37/sohl-dickstein15.pdf) but the model
was improved multiple times
([DDPM](./denoising_diffusion_probabilistic_model.md), TOOD: more).

## High level intuition of DPM

From a high level, diffusion probabilistic models attempt to capture *'noise' in
a sample* (e.g. image), such that if we subtract the predicted noise from the
input, we should move closer to the distribution in the training data.

### Training

There are two processes in training of a DPM:

- *a forward process* and
- *a reverse process*

Forward process adds noise to a training input. The reverse process tries to
remove the added noise. **The reverse process is modelled by DPM**.

Thanks to some nice math, the reverse process can be explicitly defined, which
gives us the *true* value, which we can contrast with the generated to train a
DPM.

### Sampling

To sample from a DPM we run a random noise through the model multiple times,
until it removed all the noise.

## Where is the math

The paper includes the math, but I find that
[DDPM](./denoising_diffusion_probabilistic_model.md)'s paper explained it
better.
