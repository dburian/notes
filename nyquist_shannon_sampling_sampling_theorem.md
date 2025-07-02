---
tags: [signal_processing]
---
# Nyquist-Shannon sampling theorem

Nyquist-Shannon sampling theorem says:

> Regularly sampled signal can represent any continuous signal with frequencies
> between 0 and half of the sampling rate.

Let's go through ascending frequencies $f_0, f_1, f_2, \ldots$ and reason about
why sampled signal with sampling rate $r$ can or cannot represent them:

- $f_0 = 0$ -- can be represented by sampled signal, since it is constant. One
  sample suffices to represent such signal.
- $0 < f_1 << \frac{r}{2}$ -- 
  - can bre

## Repetition of frequencies

I think that to truly understand why this theorem holds I need to know why the
following happens:

> Let's say you've sampled an analog signal $x(t)$ with spectrum $X(\omega)$ at
> rate $1/T$ that is high enough to satisfy the sampling theorem. The spectrum
> (i.e. the discrete time Fourier transform (DTFT)) of the sampled signal
> $x_{1,k}$ will be a periodic repetition of $X(\omega)$. The repetition period is
> $1/T$.
