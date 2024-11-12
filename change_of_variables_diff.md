---
tags: [math]
---

# Change of variables in differentiation

When differentiating we can use the chain rule to find the correct substitution.
Consider $f^\prime(g(x))$ and let's say we substitute $y := g(x)$, then:

$$
\begin{aligned}
\frac{\partial f(g(x))}{\partial x} &=
\frac{\partial f(g(x))}{\partial g(x)} \frac{\partial g(x)}{\partial x} \\
&= \frac{\partial f(y)}{\partial y} \frac{\partial y}{\partial x}
\end{aligned}
$$

The $\partial y/\partial x$ is the '*extra*' part we have to add for the
substitution to be valid.

## Example

For example, for $y:= x^2$:

$$
\frac{\partial \sin x^2}{\partial x} = \frac{\partial \sin y}{\partial y} 2x =
(\cos y) 2x = 2x \cos x^2
$$


