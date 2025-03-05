---
tags: [information_theory]
---

# Entropy

Entropy of a random variable $P$ is defined as:

$$
H(P) = \mathbb{E}_{x \sim P(x)} \left[I(x)\right] =
  - \mathbb{E}_{x \sim P(x)} \left[\log P(x)\right],
$$

where $I(x) = -\log P(x)$ is [*self-information*](./self_information.md).
Entropy tells us the expected amount of self-information or surprise in a
distribution.
