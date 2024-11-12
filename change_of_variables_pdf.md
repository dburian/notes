---
tags: [math]
---

# Change of variables in probablity density functions

Substitutions inside [Probability Density Functions
(PDFs)](./probability_density_function.md) are similar as substitutions in
integrals.

As before we are effectively substituting $x$ for $x := f^{-1}(y)$. Since its a
PDF, the integral must sum to one with and without the substitution. This means
that the substitution must preserve the domain. If, for example, $f^{-1}$ mapped
both $y_1 \ne y_2$ onto a single $x_1$, then the integral *after the
substitution going over the domain of $y$* would sum to less than one. Ergo
$f^{-1}$ must be **bijective**.

Let's say that $f^{-1}$ is strictly increasing (we'll cover the decreasing case
later), label $p$ as PDF of random variable $X$, and  random variable $Y = f(X)$
and follow the definition of PDF we get:

$$
\begin{align}
P(f(a) \le Y \le f(b)) &= \\
= P(a \le X \le b) &= \int_a^b p(x) dx = \\
&= \int_{f(a)}^{f(b)} p(f^{-1}(y)) \frac{\partial f^{-1}(y}{\partial y} dy
\end{align}
$$

If $f^{-1}$ is decreasing than $f(a)$ is also decreasing (visualizing this in
graph, we just switch the $x$ and the $y$ axes). This means that $f(a) > f(b)$
and the integral's boundaries should be switched -- going from $f(b)$ to $f(a)$.
This will add a minus before the expression. The probability won't be negative,
since $\frac{\partial f^{-1}(y)}{\partial y}$ is also negative ($f^{-1}$ is
decreasing function).

To put the case of decreasing and increasing $f^{-1}$ together, we just add an
absolute value and we get that the PDF of $Y$ is:

$$
q(y) = p(f^{-1}(y)) \left|\frac{\partial f^{-1}(y)}{\partial y}\right|
$$


## The multivariate case

If the PDF is multivariate, i.e. $x \in \mathbb{R}^n$ then the situation becomes
a bit more complicated. I lack a bit of theory (TODO), but intuitively we need
to scale $p$ similarly as in the univariate case.

The scale says how we need to increase the volume of $p$. In univariate case
this was done by looking how our substitution changes $f^{-1}(y)$ as we move with
$y$. This is intuitive as the integral goes through $y$, but the PDF $p$ that
digests the result of our substitution was designed to sum to 1 only when the
integral going through $x$.

In the multivariate case this is done by looking at the determinant of the
Jacobian:

$$
q(y) = p(f^{-1}(y)) \left|\det{\frac{\partial f^{-1}(y)}{\partial y}}\right|
$$

Knowing that [determinant of a linear mapping computes how volume in vector space
changes under the given mapping](./determinant.md), this may be intuitive. The
determinant just tells us how much $f^{-1}$ stretches or shrinks the vector
space in terms of volume.
