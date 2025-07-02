---
tags: [math, signal_processing]
---

# Convolution theorem

The convolution theorem states that [Fourier
transform](./discrete_fourier_transform.md) of a [convolution](./convolution.md)
of two functions is equal to product of their Fourier transform.

Intuitively it does make sense:
- In time domain the signals limit each by convolving together
  - if signals are out-of-phase they cancel-out
  - if the signals are in-phase they multiply
- This is reflected in the frequency domain:
  - if the signals are out-of-phase one of the multiplied frequency will be zero
    or very small -> resulting frequency will be small
  - if the signals are in-phase both multiplied frequencies are going to be high
    -> the resulting frequency will be even higher
