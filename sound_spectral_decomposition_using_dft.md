---
tags: [speech_to_text]
---
# Sound spectral decomposition using DFT

Spectral decomposition of sound signal describes how to split up any sound signal to
pure frequencies the sound is made up of. This is done using [Discrete Fourier
transform (DFT)](./discrete_fourier_transform.md). The result of spectral
decomposition is a [spectrogram](./spectrogram.md).



## Why DFT can do that

A nice intuition is provided by [3Blue1Brown's video
(YouTube)](https://youtu.be/spUNpyF58BY?si=SWHJfzYvdtZksMnZ). The video presents
DFT as a 'winding machine' which windes the periodic signal around a circle at
various frequencies. If the frequency of the winding matches the periodicity of
the function, weird stuff happens and we know we've hit the right spot.

While the above intuition is fun, DFT can be viewed more simply. We are
measuring similarity between two periodic signals: one that we are given and one
we create. We try several frequencies of our own signal, and see which yield
non-zero similarity. Due to the nature of periodic functions, similarities of
functions with non-matching frequencies cancel-out.

## Proof

The proof is illustrative:

1. We consider a pure frequency signals with different phases: $s^k$ and $c^k$,
   where $k$ is the frequency.
2. We then show, that by computing DFT of such signal we get an array of numbers
   $\mathcal{F}(s^k)_j$ (amplitudes per frequency), where only one amplitude $j
   = k$ is non-zero.
3. We don't show it here, but the proof continues to show, that any sound is a
   combination of $s^k$ and $c^k$ for different $k$.
4. Then it can be shown that DFT of any such combination is again array of
   amplitudes per frequency, which are non-zero only if the combination
   contained corresponding pure frequency.

### Projecting pure frequencies through DFT

Take a sine and cosine with normalized frequency $k$ (meaning $k$ periods per
unit of time):
$$
\DeclareMathOperator{\s}{s}
\DeclareMathOperator{\c}{c}
\begin{align}
\s^k = \sin{2{\pi}k} \\
\c^k = \cos{2{\pi}k}
\end{align}
$$

#### Sine DFT

If we sample $s^k$ $n$ times during the one unit in time, and project these
points with DFT, we get the following:

$$
\begin{align}
\mathcal{F}(\s^k)_j
&= \sum_{t = 0}^{n - 1} \sin\left(2{\pi}kt/n\right) \omega^{jt} \\
&= \sum_{t = 0}^{n - 1} \frac{\omega^{kt} - \omega^{-kt}}{2i} \omega^{jt} \\
&= \frac{1}{2i} \left(
  \sum_{t = 0}^{n - 1} \omega^{t(j + k)} -
  \sum_{t = 0}^{n - 1} \omega^{t(j -k)}
\right)
\end{align}
$$

where the second to line is due to expressing $\sin$ using exponentials as
follows from [Eulers formula](./eulers_formula.md). Using sum of gemoetric
series and depending on $j$, we get:

- For $j = k$:
$$
\begin{align}
&= \frac{1}{2i} \left( \frac{(\omega^{j+k})^n - 1}{\omega^{j+k}} - \sum_{t = 0}^{n - 1} \omega^{0} \right) \\
&= \frac{1}{2i} \left( \frac{1^{j+k} - 1}{\omega^{j+k}} - n \right) \\
&= \frac{1}{2i} \left( 0 - n \right) \\
&= -\frac{n}{2i}
\end{align}
$$
- For $j = n - k$:
$$
\begin{align}
&= \frac{1}{2i} \left( \sum_{t = 0}^{n - 1} \omega^{tn} - \frac{(\omega^{n-2k})^n - 1}{\omega^{n -2k}} \right) \\
&= \frac{1}{2i} \left( \sum_{t = 0}^{n - 1} (\omega^{n})^t - \frac{1^{n-2k} - 1}{\omega^{n -2k}} \right) \\
&= \frac{1}{2i} \left( \sum_{t = 0}^{n - 1} 1 - 0 \right) \\
&= \frac{n}{2i}  \\
\end{align}
$$
- For $j \ne k$ and $j \ne n-k$:
$$
\begin{align}
&= \frac{1}{2i} \left( \frac{(\omega^{j+k})^n - 1}{\omega^{j+k}} - \frac{(\omega^{n-2k})^n - 1}{\omega^{n -2k}} \right) \\
&= \frac{1}{2i} \left( 0 - 0 \right) \\
&= 0
\end{align}
$$

This means that only some Fourier coefficients will be non-zero. Specifically
those that used primitive roots of 1 that *correspond to the same period* as
$s^k$. Here is an interpretation of what we did, which might explain the
previous sentence:
- we sampled our function $\s^k$ at $n$ points, equally spaced during one
  unit of time
- we sampled another periodic complex function with period $j$:
  $\omega^{2{\pi}j}$ at $n$ points, equally spaced out during one unit of time
- we multiplied these images together
- when the peridicity of $s^k$ agreed with period of our complex function, we
  get non-zero coefficient, otherwise we get zero

#### Cosine

We could do all of the above for $\cos$ and $\c^k$ as well. We'd get:
- for $j = k$: $\mathcal{F}(\c^k)_k = -1/2$
- for $j = n - k$: $\mathcal{F}(\c^k)_{n-k} = 1/2$
- otherwise: $\mathcal{F}(\c^k)_j = 0$

#### Note about frequency range

Note that the above holds only for $k \in [1, \ldots, n/2 - 1]$. In other words,
DFT on $n$ samples can only capture amplitudes of frequencies that are lower or
equal to $n/2 - 1$.

This is because:

- for $k = 0$, $\s^k = 0$
- for $k = n/2$, $\s^{n/2}_t = \sin\left(\frac{2{\pi}n}{2} \frac{t}{n}\right) = \sin{t\pi} = 0$
- for $k = n/2 + l$:
$$
\begin{align}
\s^{n/2 + l}_t &= \sin\left(2{\pi}  \left(\frac{n}{2} + l\right) \frac{t}{n} \right) \\
&= \sin\left(t\pi + l2\pi \frac{t}{n}\right) \\
&= \pm \sin\left(2l{\pi}\frac{n}{t}\right) \\
&= \pm \s^l_t
\end{align}
$$

So by running $n$ samples though DFT, we get information about $n$ frequencies,
but only half of them are unique. The second half of frequencies, mirrors the
first one.

The same explanation can be drawn for $c^k$.

### Assembling pure frequencies into arbitrary sound

Now that we know how pure sounds manifest in DFT, we can decompose arbitrary
sound into pure sounds. The theorem is as follows: For each sampled signal $x$,
there exist $\alpha_0, \ldots, \alpha_{n/2}$ and $\beta_0, \ldots, \beta_{n/2}$
s.t.:

$$
\begin{align}
\mathcal{F}(x) &= \sum_{k = 0}^{n/2} \alpha_k \c^k + \beta_k \s^k \\
&= \left(a_0 + ib_0, \ldots, a_{n-1} + ib_{n-1}\right),
\end{align}
$$

where
$$
\begin{align*}
\alpha_0 &= a_0 / n, \\
\alpha_j &= 2a_j / n \quad \text{for} \; j = 1, \dots, n/2 - 1, \\
\alpha_{n/2} &= a_{n/2} / n, \\
\beta_0 &= \beta_{n/2} = 0, \\
\beta_j &= -2b_j / n \quad \text{for} \; j = 1, \dots, n/2 - 1.
\end{align*}
$$

The proof applies inverse DFT to both sides, and examines the vectors by the
elements. Since $s^k$ and $c^k$ are going to be non-zero at that position only
for some $k$, we can reverse engineer the $\alpha$s and $\beta$s.

## So which frequencies are there?

We've asserted that we can identify pure frequencies in arbitrary sounds: if
there is some $s^k$ or $c^k$ in the sound mixture, we get non-zero $\beta_k$ and
$\alpha_k$. But what frequency is represented by $s^k$ (or $c^k$)?

We can think of the frequency of $s^k$. It does:
- $k$ iterations over one unit of time
- $k$ iterations over $n$ points, since we sampled each unit of time $n$ times
- $\frac{k}{n}$ iterations over single point
- $\frac{k}{n} * \text{sr}$ over one second of the original signal with sample rate
  $\text{sr}$

And so by running DFT on $n$ points on signal with sample rate $\text{sr}$, we
get an array of amplitudes:

| DFT index ($=k$, $=j$) | Frequency it represents |
| ---------------------- | ----------------------- |
| $0$                    | $0$                     |
| $1$                    | $\frac{\text{sr}}{n}$   |
| ...                    | ...                     |
| $k$                    | $\frac{k}{n} * \text{sr}$ |
| ...                    | ...                     |
| $n/2$                  | $\frac{\text{sr}}{2}$   |
| $n/2 + 1$              | $\frac{\text{sr}}{2} + 1$ |
