# Cepstogram

Cepstograms are visualizations of a signal in sound processing. Due to being
kind of inverse [spectograms](./spectogram.md), they were named with first 4
letters reversed (a theme that will be soon apparent).

Cepstograms are a result of further spectral analysis of a spectogram (i.e.
spectral analysis of a spectral analysis). While the original cepstogram used an
inverse of [Discrete Fourier Transform](./discrete_fourier_transform.md),
nowadays [Discrete Cosine Transform](./discrete_cosine_transform.md) is
preferred.

## Cepstogram flavours

### Original cepstograms

The original cepstogram were created as follows:
- take each frame computed by Short-Time DFT
- apply logarithm
- for each frame compute inverse DFT

### Mel cepstogram and Mel-frequency Cepstral Coefficients (MFCCs)

Mel cepstograms come from [Mel spectograms](./spectogram.md). The additional
step is only to apply [Discrete Cosine Transform
(DCT)](./discrete_cosine_transform.md) as if the spectogram frame was a signal.

The core idea is that we'd like to explain the frequency spectrum for a
particular frame of given signal. To explain the frequency spectrum, we
decompose it to *quefrencies* (frequencies in cepstrum). Low quefrencies explain
the repeating patterns in frequency spectrum and are responsible for the main
distinguishing features of a signal. High quefrencies explain the details in
frequency spectrum and are responsible for the sound's detail (or even noise).

Applying DCT to frequency spectrum allows us to find the quefrencies, and filter
out high quefrencies, thereby focusing only on the main sound's identity. Such
filtering is called *liftering* (cepstrum filtering).

---
Sources:
  - [Audio Signal Processing for AI (YouTube playlist)](https://www.youtube.com/playlist?list=PL-wATfeyAMNqIee7cH3q1bh4QJFAaeNv0)
