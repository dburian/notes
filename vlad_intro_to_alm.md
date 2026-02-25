# Vlad Intro to Audio Language Modelling

- intro: [VAE](./variation_autoencoders.md)

## Vector quantization

- 'dictionary' matrix of embeddings = *codebook*: $M \ in \mathbb{R}^{K \times D}$
- continuous vector turns into discrete one: $\text{argmax}(xM)$
- gradients are propagated using *Straight-Through Estimator* (*STE*)
  - which doesn't update the codebook entries, the codebook has separate
    loss:
    - codebook loss: entries are moved toward the closest $x$ manually
  - then there is commitment loss: $x$ are penalized from being far away from
    codebook entries

## Vector quantization prior

- VQ-VAE idea: train generative model on the encoded pieces of input
  - e.g. split image to grid, encode each cell, train autoregressive model on
    the series of encodings
- 




