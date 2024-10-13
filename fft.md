---
tags: [informatics,math]
---
# Fast Fourier transform

Fast Fourier transform (FFT) computes [Discrete Fourier
transform (DFT)](./discrete_fourier_transform.md) in $O(n\log n)$ time.

Naive evaluation would run in $O(n^2)$ time since we need to evaluate polynomial
$x_0, \ldots, x_{n-1}$ in $n$ points $(P(z_0), \ldots, P(z_{n-1}))$.

## Idea of FFT

The idea of FFT is pretty simple:

- We assume: $n = 2^l$ for some $l$, if $n$ is smaller, we pad it with zeros.

- divide the polynomial $P$ into even and odd powers:

$$
\begin{align}
P(z) &=
\left(
  x_0 z^0 +
  x_2 z^2 +
  x_4 z^4 +
  \ldots +
  x_{n - 2} z^{n - 2}
\right) \\
&\quad +
z \left(
  x_1 z^0 +
  x_3 z^2 +
  x_5 z^4 +
  \ldots +
  x_{n - 1} z^{n - 2}
\right) \\
&= P_{e}(z^2) + z P_o(z^2)
\end{align}
$$

- now we have to evaluate two polynomials that are half the original size at $n$
  points

- Further, we strategically pick the $n$ points, s.t. they are paired:

$$
z_k = - z_j, \,\text{for some}\, i,j \in {0, \ldots, n-1}
$$

- now we only have to evaluate two polynomials that are half the original size
  at half the points since $z_k^2 = z_j^2$:

$$
\begin{align}
P(z_k) &= P_e(z_k^2) + z_k P_o(z_k^2) \\
P(z_j) &= P_e(z_k^2) - z_j P_o(z_k^2) \\
\end{align}
$$

- We apply the same idea recursively, which gives us $\log_2 n$ iterations. The
  number of $z$s to evaluate also decreases by a factor of 2 ($z$s are
  paired), which means at level $l$ we are going to be computing $\frac{1}{2^l}$
  $z$s. However the number of branches also grows exponentially which mean
  at each depth, we are going to be doing $n$ computations. This means $O(n\log
  n)$ time in total.

The only hiccup is that after the first iteration, the $z$s are all going
to be positive and we will not be able to find $k, j$ s.t. $z^2_j = -
z^2_k$. This is at least true for **real numbers**.

For complex numbers there is a convenient option for $z_k$ to be the [n-th
root of 1](./primitive_root_of_one.md) at the power of $k$. This means that we
define $z$ as:

$$
z_k = \omega^k,
$$

where $\omega$ is n-th primitive root of one. Thanks to the characteristics of
primitive roots, the following are true:

- $\omega^j \ne \omega^k$ for $0 \le j < k < n$ -- ergo we will get value of $P$
  at $n$ different points
- For even $n$: $\omega^{\frac{n}{2} + j} = \omega^\frac{n}{2} \cdot \omega^j =
  -\omega^j$ -- ergo we can find $j, k$ such that $z_k$ and $z_j$ are paired
- For even $n$: $\omega^2$ is $\frac{n}{2}$-th primitive root of 1
  - all powers are unique and less then one $(\omega^2)^0 < (\omega^2)^1 <
    \ldots < (\omega^2)^{\frac{n}{2} - 1} < (\omega^2)^\frac{n}{2} = 1
  - meaning we can find pairing on every depth as long as $n = 2^l$

### Summary

The reason why the algorithm is as fast is thanks to the fact that we can
divide the problem to 2 parts, each of which is four times easier than the
original solution (half the polynomial size, half of the evaluation points).

## Algorithm

```python
def fft(x: list[float], n: int, w: complex) -> list[complex]:
  """Computes fft.

  Parameters
  ----------
  x
    Vector of n numbers.
  n
    n = 2**l for some l
  w
    Primitive n-th root of 1

  Returns
  -------
  Vector of n complex Fourier coefficients.
  """
  if n == 1:
    return x[0]

  evens = fft(x[::2], n/2, w**2)
  odds = fft(x[1::2], n/2, w**2)

  coefs = [0 for _ in range(n)]
  w_to_i = 1
  for i in range(n/2):
    coefs[i] = evens[i] + w_to_i * odds[i]
    # w^{n/2 + i} = -w^i
    coefs[n/2 + i] = evens[i] - w_to_i * odds[i]
    w_to_i *= w

  return coefs
```
