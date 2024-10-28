---
tags: [information_theory]
---

# Kullback–Leibler divergence

Kullback-Leibler divergence (or KL divergence or relative entropy) is defined
for two random variables $P$ and $Q$ as:

$$
D_{KL}(P || Q) = H(P, Q) - H(P) = \mathbb{E}_{x \sim P(x)} \left[\log
\frac{P(x)}{Q(x)} \right],
$$

where $H$ is a [cross-entropy](./cross_entropy.md). KL divergence tells us how
much surprise we **add** if we asses the surprise of an event according to $Q$,
instead of $P$.
