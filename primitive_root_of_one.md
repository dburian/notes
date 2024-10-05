# Primitive root of 1

In complex numbers, 1 has many n-th roots, but not all of them are primitive.
Number $x \in \mathbb{C}$ is primitive n-th root of 1 if $x^n = 1$ and $x^0,
x^1, \ldots, x^{n-1} \ne 1$.

Each complex number has at least 2 N-th primitive roots: $\omega =
e^{2{\pi}i/n}$ and $\overline{\omega} = e^{-2{\pi}i/n}$. These come from [Eulers
formula](./eulers_formula.md). If we imagine complex numbers on 2D plane,
computing their power is like rotating them counter-clockwise (for $\omega$) or
clockwise (for $\overline{\omega}$). We get N-th primitive root, by splitting
the circle in $n$ parts ($2\pi/n$) and taking the first one.

![5-th primitive root of one. From Labyrint algoritmů
(https://pruvodce.ucw.cz/)](./imgs/5_primitive_root_of_one.png)

## Characteristics

N-th primitive roots have some nice properties:

- $\omega^j \ne \omega^k$ for $0 \le j < k < n$
- For even $n$: $\omega^\frac{n}{2} = \sqrt{\omega^n} = \sqrt{1} = -1$ (it
  cannot be 1 since $\frac{n}{2} < n$ and $\omega$ is n-th primitive root of 1)
- For even $n$: $\omega^{\frac{n}{2} + j} = \omega^\frac{n}{2} \cdot \omega^j =
  -\omega^j$
