---
tags: [math]
---

# Change of variables in integrals

## Indefinite integrals

Imagine an integral with an expression $f(x)$ that you want to substitute for
$y$:

$$
G(x) = \int g(f(x)) dx
$$

What this tells us that if we derivate $G(x)$ we would get $g(f(x))$. We are
free to substitute $x$ for anything else (at both sides of the equation), but we
need to be vary of the chain rule which is more visible if we rewrite the above
as differentiation:

$$
G^\prime(x) = g(f(x))
$$

If we want to get rid of $f(x)$ by substituing it for $y := f(x)$ we essentially
substitute $x := f^{-1}(y)$. If we do that on the LHS, we need to multiply RHS
by $(f^{-1})^\prime(y)$ according to the chain rule, since RHS is
differentiation of LHS:

$$
G^\prime(f^{-1}(y)) = g(f(f^{-1}(y))) \frac{\partial f^{-1}(y)}{\partial y} =
g(y) \frac{1}{f^\prime(x)}
$$

Where the last equation is just rule of differentiating of functions, derived
from derivating $x = f(f^{-1}(x))$ on both sides.

Finally, the last equation in integral form:

$$
G(f^{-1}(y)) = \int g(y) \frac{1}{f^\prime(x)} dy
$$

To summarize:

1. identify the function which we want to get rid of ($f(x)$)
1. realize what we are substituing $x$ for ($x$ for $f^{-1}(y)$)
1. realize we are substituing in the differentiating RHS and we need to follow
   the chain rule (multiply by $(f^{-1})^\prime(y)$ or divide by $f^\prime(x)$)

### Example

Let's say we substitute $y:= 2x^3 + 1$ in:

$$
\int (2x^3 + 1)^7 x^2 dx
$$

Now:

$$
\frac{\partial 2x^3 + 1}{\partial x} = 6x^2
$$

So we need to divide the expression by 6x:

$$
\int (2x^3 + 1)^7 x^2 dx = \int (y)^7 \frac{x^2}{6x^2} dy =
\frac{1}{6} \int y^7 dy
$$

## Definite integral

If we have definite integral, say going from $a$ to $b$ and we substitute $y:=
f(x)$, then we proceed as with indefinite integral, only changing $a$ to $f(a)$
and $b$ to $f(b)$.
