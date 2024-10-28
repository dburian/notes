---
tags: [ml]
---
# Autoencoder

An autoencoder is a general term for model that predicts its input. Without
further restrictions, these models would be trivial. And so the architecture
includes a bottleneck which forces the model to represent the input $x$ with a
a **latent variable** $z$ of much smaller dimensionality.

This makes autoencoders' architectures look like hourglasses:
- the large $x$ is reduced to $z$ using an **encoder**
- then $x$ is reconstructed from $z$ by a **decoder**

Sometimes a small noise is added to the input: $x + \epsilon$, where $\epsilon
\sim \mathcal{N}(0, 1)$.
