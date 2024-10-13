---
tags: [math]
---
# Taylor series

The formula that evaluates a function using a sum of its derivatives.

$$
\mathcal{T}^{f, a}_n(x) =
    \sum_{n = 0}^\infty \frac{f^{(n)}(a)}{n!} (x - a)^n,
$$

where:

- $f$ is the function which we want to calculate,
- $a$ is the point at which $f$ is infinitely differentiable,
- $x$ is the variable of the Taylor series.

With any $n < \infty$ Taylor series is only an approximation that is closest
to the original function at point $x = a$.

## Series for popular functions at $a = 0$

Exponential function:

$$
e^x = \sum_{n = 0}^\infty \frac{x^n}{n!}
$$

$\sin$:

$$
\sin{x} = \sum_{n=0}^\infty \frac{(-1)^n}{(2n + 1)!} x^{2n + 1}
$$


$\cos$:

$$
\cos{x} = \sum_{n=0}^\infty \frac{(-1)^n}{(2n)!} x^{2n}
$$
