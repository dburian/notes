[cca]: cca.md

# Singular Value Decomposition (SVD)

SVD is a concept that appears in a lots of places (TODO: links) and therefore it
is important to master.

## Definition

Any $m\times n$ matrix $A$ with $rank(X) = r$ can be decomposed as
$$
A = U\Sigma V^T
$$

, where columns of $U \in \mathbb{R}^{m \times m}$'s and $V$'s columns are
orthonormal bases of A's column space and row space respectively.

## Intuition and geometric meaning

We can view SVD as a decomposition of linear transformation, defined by matrix
$A$ into:

1. $U$: rotation of space into new set of orthogonal coordinates
2. $\Sigma$: scaling new coordinates
3. $V^T$: rotation of scaled space, back to the original space

## Computation

TODO:
From [MIT](https://math.mit.edu/classes/18.095/2016IAP/lec2/SVD_Notes.pdf) see
the proof of SVD. Has connections to [CCA][cca].


## Connection to eigenvalues

Singular values $\sigma_i = \Sigma_{i, i}$ are square roots of eigenvalues of
$A^TA$:

$$
A^TA = (U\Sigma V^T)^T(U\Sigma V^T) = V\Sigma U^T U \Sigma V^T = V\Sigma^2 V^T
$$

Source: [Matrix Based Introduction to Multivariate Data
Analysis](https://link.springer.com/chapter/10.1007/978-981-10-2341-5_14)

