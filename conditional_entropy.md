---
tags: [information_theory]
---

# Conditional entropy

Conditional entropy is a special form of [cross entropy](./cross_entropy.md),
where we fix one random variable in their joint distribution. It is defined as:

$$
H(X|Y) = -\mathbb{E}_{(x, y) \sim P(x, y)} \left[\log P(x|y) \right]
$$

