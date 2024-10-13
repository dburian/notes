---
tags: [statistics]
---

[matrix_norms]: matrix_norms.md
# Canonical Correlation Analysis (CCA)

CCA is a statistical method to find linear projections of two sets of variables
$X$ and $Y$ such that these projections are maximally correlated.

## Definition

- for observations $X \in \mathbb{R}^{n_1 \times m_1}$, $Y \in \mathbb{R}^{n_2
  \times m_2}$,
- we search for weights $p \in \mathbb{R}^m_1$, $q \in \mathbb{R}^m_2$ such
  that:

$$
\begin{align}
 \delta &= max_{p, q} corr(Xp, Yq) \\
        &= max_{p, q} \frac{(Xp)^T Yq}{|Xp| |Yq|} \\
        &= max_{p, q} \frac{p^T \Sigma q}{\sqrt{(Xp)^T \cdot Xp} \sqrt{(Yq)^T \cdot Yq}}
\end{align}
$$

Since correlation is invariant to scaling we additionally constrain the
projections to be unitary:

$$
\begin{align}
    |Xp|^2 &= (Xp)^T \cdot Xp = p^T X^T X p = p^T \Sigma_X p = 1 \\
    |Yq|^2 &= (Yq)^T \cdot Yq = q^T Y^T Y q = q^T \Sigma_Y q = 1
\end{align}
$$

### More projections

When more projections are needed we require that the projections should be
uncorrelated:

$$
\begin{align}
    corr(Xp_1, Xp_2) &= p_1^T \Sigma_X p_2 = 0 \\
    corr(Yq_1, Yq_2) &= q_1^T \Sigma_Y q_2 = 0
\end{align}
$$

We can put the projections into matricies $P = [p_1, p_2, ...]$ and $Q = [q_1,
q_2, ...]$. Then we maximize the sum of all canonical correlations:

$$
\sum_i \delta_i = max_{P, Q} trace(P^TX^TYQ) \\
$$

such that $ P^TX^TXP = I$ and $Q^TY^TYQ = I$.

We can also reformulate the above in terms of minimzalization of a [Forbenius
norm][matrix_norms]:

$$
\begin{align}
    \sum_i \delta_i &= min_{P, Q} ||XP - YQ||_F^2 \\
                    &= min_{P, Q} trace((XP - YQ)^T(XP - YQ)) \\
                    &= min_{P, Q} trace(P^TX^TXP - P^TX^TYQ -Q^TY^TXP + Q^TY^TYQ) \\
                    &= min_{P, Q} trace(- P^TX^TYQ - Q^TY^TXP) \\
                    &= min_{P, Q} -2 trace(P^TX^TYQ) \\
                    &= max_{P, Q} trace(P^TX^TYQ) \\
\end{align}
$$


We can arrange these vectors $p_i$ and $q_i$ as columns into matricies $P$ and
$Q$ respectively.

## Solving CCA

### Sources and references

The following is mainly from an exert from [Encyclopedia of Social Network
Analysis and
Mining](https://link.springer.com/referenceworkentry/10.1007/978-1-4939-7131-2_110191).
Though it is not available to public for free, I've found this to be best source
in terms of explanation. As an alternative there is [a paper on CCA hosted on
Standford.edu](https://graphics.stanford.edu/courses/cs233-21-spring/ReferencedPapers/CCA_Weenik.pdf)
or a paper that introduces [Deep
CCA](http://proceedings.mlr.press/v28/andrew13.pdf) which also mentions how to
solve CCA from the standpoint of implementation.

Deriving the solution may be lengthy, but in the end we find that there is only
a few computations one needs to do. For a reference implementation have a look at
[use of CCA as a loss in Deep CCA
implementation](https://github.com/Michaelvll/DeepCCA/blob/master/objectives.py).

### Lagrange multiplier

We start by writing the optimization problem in Lagrangian form:

$$
L(\alpha, \beta, p, q) = p^T \Sigma q
    - \frac{\alpha}{2} (p^T\Sigma_X p -1)
    - \frac{\beta}{2} (q^T\Sigma_Y q -1)
$$

Computing the derivation of the above with respect to $p$ and $q$, and setting
the result equal to 0 we get:

$$
\begin{aligned}
    \frac{\partial L}{\partial p} &= \Sigma q - \alpha\Sigma_X p = 0 \\
    \frac{\partial L}{\partial q} &= \Sigma^T p - \beta\Sigma_Y q = 0
\end{aligned}
$$

We can pre-multiply the equations by $p^T$ and $q^T$ respectively:

$$
\begin{aligned}
    p^T \Sigma q &= \alpha p^T \Sigma_X p = \alpha \\
    q^T \Sigma^T p &= \beta q^T \Sigma_Y q = \beta
\end{aligned}
$$

since the projections $Xp$ have unitary length (viz. equation in definition).
Because the left hand sides are a scalar, we can write

$$
p^T \Sigma q = (p^T \Sigma q)^T = q^T \Sigma p
$$

and hence $\alpha = \beta$. According to definition of $\delta$ and due to how
Lagrangian optimization works we can also say that $\alpha = \beta = \delta$.

### Formulating the optimization as eigenvalue problem

By post-multiplying by $\Sigma_X^{-1}$ and $\Sigma_Y^{-1}$ equations with
partial derivations of $p$ and $q$ respectively we get:

$$
\begin{aligned}
    \Sigma_X^{-1}\Sigma q &= \delta p \\
    \Sigma_Y^{-1}\Sigma^T p &= \delta q
\end{aligned}
$$

By substituting $p$, and $q$ from the other equation we get:

$$
\begin{aligned}
    \Sigma_X^{-1} \Sigma \Sigma_Y^{-1} \Sigma^T p &= \delta^2 p \\
    \Sigma_Y^{-1} \Sigma \Sigma_X^{-1} \Sigma q &= \delta^2 q
\end{aligned}
$$

We see that $p$ is an eigenvector of the matrix $\Sigma_X^{-1} \Sigma
\Sigma_Y^{-1} \Sigma^T$ with eigenvalue $\delta^2$.

### More projections and matrix form

Usually we'd like to compute more projections than just one. In this case we add
the, already mentioned, constraint that the projections are uncorrelated (or in
other words orthogonal):

$$
\begin{aligned}
    P^T X^T X P &= P^T \Sigma_X P = 1 \\
    Q^T Y^T Y Q &= Q^T \Sigma_Y Q = 1
\end{aligned}
$$

The eigenvalue problems can now be rewritten in matrix form:

$$
\begin{aligned}
    \Sigma_X^{-1} \Sigma \Sigma_Y^{-1} \Sigma^T P &= P \Delta^2 \\
    \Sigma_Y^{-1} \Sigma \Sigma_X^{-1} \Sigma Q &= Q \Delta^2
\end{aligned}
$$

where $\Delta$ is a diagonal matrix with $\Delta_{ii} = \delta_i$ for
projections $p_i$ and $q_i$.

### Eigendecomposition

By substituting $P$ for $\tilde{P} = \Sigma_X^{-\frac{1}{2}}P$ we can get an
eigendecomposition:

$$
\begin{aligned}
    \Sigma_X^{-1} \Sigma \Sigma_Y^{-1} \Sigma^T P &= P \Delta^2 \\
    \Sigma_X^{-1} \Sigma \Sigma_Y^{-1} \Sigma^T \Sigma_X^{-\frac{1}{2}} \tilde{P} &= \Sigma_X^{-\frac{1}{2}} \tilde{P} \Delta^2 \\
    \Sigma_X^{\frac{1}{2}} \Sigma_X^{-1} \Sigma \Sigma_Y^{-1} \Sigma^T \Sigma_X^{-\frac{1}{2}} \tilde{P} &= \tilde{P} \Delta^2 \\
\end{aligned}
$$

From the above we can see that $\tilde{P}$ stores eigenvectors with eigenvalues
$\Delta^2$ for the symmetric matrix: $\Sigma_X^{\frac{1}{2}} \Sigma_X^{-1}
\Sigma \Sigma_Y^{-1} \Sigma^T \Sigma_X^{-\frac{1}{2}} \tilde{P}$. Since the
matrix is symmetric, the eigenvectors are orthogonal and therefore we get an
eigenvalue decomposition:


$$
\Sigma_X^{\frac{1}{2}} \Sigma_X^{-1} \Sigma \Sigma_Y^{-1}
\Sigma^T \Sigma_X^{-\frac{1}{2}} =
\tilde{P} \Delta^2 \tilde{P}^T \\
$$

With the same arguments we can derive that the same holds for $\tilde{Q} =
\Sigma_Y^{-\frac{1}{2}}$:

$$
\Sigma_Y^{\frac{1}{2}} \Sigma_Y^{-1} \Sigma^T \Sigma_X^{-1}
\Sigma \Sigma_Y^{-\frac{1}{2}} =
\tilde{Q} \Delta^2 \tilde{Q}^T \\
$$

### Correlation of all views

If we care only about the total correlation we can stop here since trace of a
matrix is the sum of all its eigenvalues. Ergo to get the sum of all $\delta_i$
we can do:

$$
\sum_i \delta_i = trace(\sqrt{
    \Sigma_Y^{\frac{1}{2}} \Sigma_Y^{-1} \Sigma^T \Sigma_X^{-1}
    \Sigma \Sigma_Y^{-\frac{1}{2}}
})
$$

### Singular Value Decomposition

Since the decomposed matrix is symmetric, its singular values are equal to its
eigenvalues.

> The below may not be completely true. Nevertheless it should hold that by
> doing SVD on the decomposed matrix we get the same thing as if we were doing
> the eigendecomposition as in the previous section.

Also for symmetric matrix $A$, its SVD is in form: $BDC^T$, where $C = B$:

$$
BD^2 B^T = AA^T = A^2 = A^T A = CD^2C^T
$$

Ergo

$$
\Sigma_Y^{\frac{1}{2}} \Sigma_Y^{-1} \Sigma^T \Sigma_X^{-1}
\Sigma \Sigma_Y^{-\frac{1}{2}}
= U \Delta^2 U^T
$$

So performing SVD gives us all correlations squared.
