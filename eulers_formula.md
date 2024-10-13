---
tags: [math]
---

# Euler's formula

The formula in complex analysis, that combines complex exponential with
trigonometric functions:

$$
e^{ix} = \cos{x} + i\sin{x}
$$

## Consequences

Through Euler's formula we can define cosine or sine only by complex
exponential.

**Cosine:**
$$
\begin{aligned}
e^{ix} + e^{-ix} &= \cos{x} + i\sin{x} + \cos{x} - i\sin{x} = 2\cos{x} \\
\cos{x} &= \frac{e^{ix} + e^{-ix}}{2}
\end{aligned}
$$

**Sine:**
$$
\begin{aligned}
e^{ix} - e^{-ix} &= \cos{x} + i\sin{x} - \cos{x} + i\sin{x} = 2i\sin{x} \\
\sin{x} &= \frac{e^{ix} - e^{-ix}}{2i}
\end{aligned}
$$

## Proofs

### Through Taylor series (original proof)

Even though there are other proofs, let's go over the 'original' one through
which the relationship was discovered.

The formula comes from the [Taylor series](./taylor_series.md). If we compare the
series for $\cos$, $\sin$ and $e$ we get:

$$
\begin{aligned}
e^{ix} &= \sum_{n = 0}^\infty \frac{(ix)^n}{n!} \\

\color{teal}
\cos{x} &\textcolor{teal}{=}
    \color{teal} \sum_{n=0}^\infty \frac{(-1)^n}{(2n)!} x^{2n} \\

\color{purple}
i\sin{x} &\textcolor{purple}{=}
    \color{purple} \sum_{n=0}^\infty \frac{i(-1)^n}{(2n + 1)!} x^{2n + 1}\\
\end{aligned}
$$

Now if we expand the series of $e^{ix}$ we get:

$$
\begin{alignat}{7}
e^{ix} &=
\textcolor{teal}{\frac{1}{1}} &&+
\textcolor{purple}{\frac{ix}{1}} &&+
\textcolor{teal}{\frac{-x^2}{2!}} &&+
\textcolor{purple}{\frac{-ix^3}{3!}} &&+
\textcolor{teal}{\frac{x^4}{4!}} &&+
\textcolor{purple}{\frac{ix^5}{5!}} &&+
\ldots \\
&=
\textcolor{teal}{\cos_{n=0}} &&+
\textcolor{purple}{i\sin_{n=0}} &&+
\textcolor{teal}{\cos_{n=1}} &&+
\textcolor{purple}{i\sin_{n=1}} &&+
\textcolor{teal}{\cos_{n=2}} &&+
\textcolor{purple}{i\sin_{n=2}} &&+
\ldots \\
e^{ix} &= \textcolor{teal}{\cos{x}} &&+ \textcolor{purple}{i\sin{x}}
\end{alignat}
$$

## As a rotation

Frequently euler's formula is used in the context of 2D rotation. Here a linear
isomorphism from $\mathbb{R}^2$ to $\mathbb{C}$ is assumed s.t. $(a, b) \rightarrow a + ib$.

Essentially the $i$ is taken as a mark of second dimention. So multiplying by
the following matrix:

$$
\begin{pmatrix}
\cos{x} & -\sin{x} \\
\sin{x} & \cos{x}
\end{pmatrix}
$$

corresponds to multiplying a complex number by $e^{ix} = \cos{x} + i\sin{x}$.

