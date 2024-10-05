# Discrete Fourier transform

Discrete Fourier transform (DFT) is a function $\mathcal{F}: \mathbb{C}^n
\rightarrow \mathbb{C}^n$:

$$
\mathcal{F}(x)_j = \sum_{k = 0}^{n -1} x_k \cdot \omega_n^{jk},
\quad j \in \{0..n-1\},
$$

where $\omega_n$ is the [n-th primitive root of 1](./primitive_root_of_one.md).

## What it is

DFT is essentially evaluating polynomial $(x_0, x_1, \ldots, x_{n-1})$ at points
$(\omega_n^{0}, \omega_n^{1}, \ldots, \omega_n^{n-1})$. The reason why we chose
$\omega_n^j$, is due to the characteristics of n-th primitive roots of one,
thanks to which we can evaluate DFT in $O(n\log n)$ time using algorithm called
[Fast Fourier Transform](./fft.md).

## There is an inverse too

Since DFT is a linear projection:
$$
\mathcal{F}(\mathbf{x}) = \mathbf{\Omega} \mathbf{x}, \;\text{s.t.}\;
\mathbf{\Omega}_jk = \omega^{jk}
$$


There is also an inverse:
$$
\mathbf{\Omega^{-1}} = \frac{\overline{\mathbf{\Omega}}}{n}
$$


## Why is it useful

DFT is useful for many things. Here is some short list to give you an idea.

### Multiplying polynomials

For a naive multiplication of two polynomials of size $n$, we would need
quadratic time. Since polynomial of size $n$ is defined by arbitrary, but different
$n$ points. We can do the following:
1. Evaluate each polynomial in $n$ points using DFT -- $O(n\log n)$
2. Multiply corresponding pair of coefficients together -- $O(n)$
3. Do inverse of DFT -- $O(n\log n)$

### Spectral analysis of a signal

For a signal (e.g. sound) we can use DFT to find pure frequencies the signal is
made up of. This is somehow more involved and there is a separate note dedicated
to [Spectral analysis of sound](./spectral_analysis_of_sound.md).
