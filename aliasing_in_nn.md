---
tags: [ml, skimmed]
---

# Aliasing in NNs

Aliasing is a phenomena where different signals are after discrete sampling
indistinguishable (aliases of each other). This happens due to small sampling
rate, that fails to represent fast oscillation of the continuous signal.

There are several papers mentioning networks aliasing:
- [StyleGAN3](./stylegan3.md) -- claims anti-aliasing happens due to non-ideal
  upsampling (e.g. strided convolutions) and point wise non-linearities (e.g.
  ReLU)
- [Poor generalization of Convolutional
  networks](./poor_generalization_of_convolutional_networks.md) -- discusses why
  using stride doesn't mean that models are going to be shift invariant
- [Making convolutional networks shift-invariant
  again](https://proceedings.mlr.press/v97/zhang19a/zhang19a.pdf) --
  incorporates common anti-aliasing layers that mitigate improve shift
  invariance without worsening evaluation metrics.

The whole aliasing and anti-aliasing convolutional networks issue seems
to be quite a rabbit hole. Lot's of papers quoting "common fix to aliasing in
convolutions" and so on, all of which is news to me. So I won't go deeper into
this, and just acknowledge that it happens
