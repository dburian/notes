[field]: field.md

# Matrix norms

There are many, but the most popular is Forbenious norm.

## General form

For a [field][field] $K$ let's suppose:

- $\lVert \cdot \rVert_\alpha$ be a vector norm on $K^n$
- $\lVert \cdot \rVert_\beta$ be a vector norm on $K^m$


## Forbenious norm

Forbenious norm is a special case of a $L_{p, q}$ norm for $p = q = 2$.

$$
||A||_F = \sqrt{\sum_i \sum_j |a_{i, j}|^2} = \sqrt{trace(A^TA)}
$$

## Nuclear norm

Nuclear norm is a variant of Schatten p-norms for $p=1$. For a matrix $A \in
\mathbb{R}^{m \times n}$ and its [singular values](./svd.md) $\sigma_i(A)$ for $i \in
\{1\ldots \min{\{m, n\}}\}$, it is defined as:

$$
||A||_{\ast} = \text{trace}(\sqrt{A^TA}) = \sum_{i}^{\min{\{m, n\}}} \sigma_i(A)
$$
