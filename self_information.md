---
tags: [information_theory]
---

# Self-information

Self-information $I(x)$ is measure of surprise in an event x:.

$$
  I(x) = - \log P(x)
$$

The more sure the event is going to be, the less surprising it is.

## Bits of information

If we use logarithm with base $2$, we might also say that self-information
specifies bits of information that are needed to describe the distribution. The
more deterministic $P(x)$ is, the smaller $I(x)$ will be and the less bits we
need, since a lot of the information is *implicitly* given (since $P$ is
deterministic). The less deterministic $P$ is, the more bits we need to define
individual events, since there are more of them that are likely.

