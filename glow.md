---
tags: [ml]
---
# Glow

Glow is a [flow network](./flow_networks.md) introduced by [Kingma et al.
(2018)](https://proceedings.neurips.cc/paper_files/paper/2018/file/d139db6a236200b21cc7f752979132d0-Paper.pdf).

Glow follows up on [RealNVP (2016)](https://arxiv.org/pdf/1605.08803), but
creates two new layers:

- actnorm -- *learned* normalization of activations that replaces Batch Norm
  used in RealNVP
- 1x1 invertible convolution -- layer that replaces the masking in RealNVP

For more info reference the papers, so far I didn't really need more info
🤷.
