---
tags: [math, signal_processing]
---

# Convolution

In mathematics convolution is defined as

$$
(f \star g)(t) := \int_{-\inf}^{\inf} f(\tau)g(t - \tau) d\tau,
$$

where
- $f$, $g$ are two functions

Essentially we:
- mirror $g$
- move $g$, so it has its value at 0 is at $t$
- compute total sum under production of the moved $g$ and $f$

Usually $g$ is zero far from zero.
