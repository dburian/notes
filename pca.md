# Principal Components Analysis

Dimensionality reduction technique that projects a set of points in a
$d$-dimensional space to $k$-dimensional such that the coordintes of the target
space explain as much variance of the given points as possible.

## Computations
### Using Covariance matrix

Given a set of points $X \in \mathbb{R}^{n \times d}$ we are looking for linear
projection $w_1 \in \mathbb{R}^d$ such that the variance $Var(X_{i\star}w_1)$
for $i \in \{1\dots n\}$ is maximized. Let's treat $x \in \mathbb{R}^d$ as
random vector sampled from the distribution of input points (one of the rows of
$X$).

$$
\begin{align}
Var(x^Tw_1) &= E[(x^Tw_1 - E[x^Tw_1])^2]\\
&= E[(x^Tw_1 - \mu^Tw_1)^2]\\
&= E[(x^Tw_1 - \mu^Tw_1)(x^Tw_1 - \mu^Tw_1)]\\
&= E[(x^Tw_1 - \mu^Tw_1)^T(x^Tw_1 - \mu^Tw_1)]\\
&= E[(w_1^Tx - w_1^T\mu)(x^Tw_1 - \mu^Tw_1)]\\
&= E[w_1^T(x - \mu)(x - \mu)^Tw_1]\\
&= w_1^TE[(x - \mu)(x - \mu)^T]w_1\\
&= w_1^T\Sigma w_1
\end{align}
$$

where $\Sigma$ is covariance matrix of $x$: $\Sigma = cov(x, x) = E[(x − \mu)(x −
\mu)^T]$.

We contrain $w_1$ to have unit length and use Lagrange multiplier method (TODO),
which gives us:

$$
\Sigma w_1 = \lambda w_1
$$

Therefore $w_1$ must be an eigenvector of covariance matrix $\Sigma$. If we want
maximize $w_1^T\Sigma w_1 = \lambda w_1^T w_1$ we must choose the eigenvector
with the largest eigenvalue.

#### Construction

We iteratively apply the steps described above with the additional constraint
that new $w_i$ must be orthogonal to $w_j$ for all $j<i$. We get eigenvectors of
the covarience matrix, sorted by their corresponding eigenvalues.

When using covariance matricies to come up with principal components, you do not
need to center your data before hand as the covariance of the centered data is
the same as for non-centered. This, however, does not apply for other techniques
how to compute principal components.

### Other techniques

Note that PCA can be done using other techniques such as Singular Value
Decomposition (SVD) TODO.

TODO: explained variance

## Choosing the number of principal components

We usually stop when Proportion of Variance (PoV) explained:

$$
\frac{\lambda_1 + \lambda_2 + \ldots + \lambda_k}{\lambda_1 + \lambda_2 + \ldots
+ \lambda_k + \ldots + \lambda_d}
$$

is above 0.9.
