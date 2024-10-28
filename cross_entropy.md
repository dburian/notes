---
tags: [information_theory]
---

# Cross-entropy

Cross-entropy is defined for two random variables $P$ and $Q$ as:

$$
H(P, Q) = -\mathbb{E}_{x \sim P(x)} \left[\log Q(x)\right]
$$

It holds that
- $H(P, Q) \geq H(P)$, where $H(P)$ is an [entropy](./entropy.md) of $P$ and
- $H(P, Q) == H(P) \Leftrightarrow P = Q$.

Cross-entropy is a measure of how distributions are different. In other words,
it tells us how surprising are events sampled from $P$ according to $Q$.
