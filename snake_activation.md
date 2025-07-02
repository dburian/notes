---
tags: [ml,skimmed_over]
---
# Snake activation

Snake activation was introduced by [Ziyin et al.
(2020)](https://proceedings.neurips.cc/paper/2020/file/1160453108d3e537255e9f7b931f4e90-Paper.pdf).

Snake activation function was invented to help neural networks generalize
(extrapolate) periodic functions.

## Activation

Snake activation is defined as:

$$
\text{Snake}(x) = x + \frac{1}{a} \sin^2{(ax)},
$$

where parameter $a$ defines the period. Although, the authors don't heavily
experiment with $a$ being learnable, it is possible to train $a$. $\frac{1}{a}$
is needed to keep Snake activation monotonic.

### Why not some other periodic function?

The $\text{Snake}$ activation benefits from several good qualities:

- is monotonic (same as $x + \sin{x}$ or $x + \cos{x}$) -- non-monotonic
  functions can be hard to optimize since there exist many (potentially
  infinitely many for $\sin{x}$) inputs for which the output is the same.
  Local-based [gradient descent](./rewrite/00002b.md) cannot know the global
  behaviour of the output.
- has positive first non-linear term -- rewriting it using [Taylor
  series](./taylor_series.md) the first non-linear term is $x^2$, whereas for
  $x + \sin{x}$ it's $-x^3$. This first non-linear term drives the non-linearity.
  For $x + \sin{x}$ the first non-linear term goes against $x$ and 'vanishes'
  the output. This doesn't happen for $\text{Snake}$ with $x^2$ as the first
  non-linear term.
- behaves good around $0$ -- although we could mitigate this with different
  initialization, the authors note that **initializing weights before
  $\text{Snake}$ to Uniform** around $0$ with some given variance should be
  fine.
