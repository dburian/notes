---
tags: [information_theory]
---

# Entropy

Entropy of a random variable $P$ is defined as:

$$
H(P) = \mathbb{E}_{x \sim P(x)} \left[I(x)\right] =
  - \mathbb{E}_{x \sim P(x)} \left[\log P(x)\right],
$$

where $I(x) = -\log P(x)$ is *self-information*. Self-information describes the
amount of *surprise* in an event. The more sure the event is going to be, the
less surprising it is.

So, entropy tells us the expected amount of surprise in a distribution.
