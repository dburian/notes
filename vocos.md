---
tags: [ml,speech_to_text]
---
# Vocos

[Vocoder](./beginners_guide_to_tts.md) introduced by [Siuzdak
(2024)](https://arxiv.org/pdf/2306.00814). Compared to other vocoders such as
[BigVGAN](./bigvgan.md) or [HifiGAN](./hifigan.md), which try to upsample
mel-spectrograms in the time domain, Vocos chooses a rudimentary different
approach of trying to estimate [Short-Time-Fourier-Transform (STFT)
Coefficients](./spectrogram.md) and computing the waveform via inverse STFT.
This approach requires significantly less parameters, while matching the
performance of BigVGAN.

## Model
