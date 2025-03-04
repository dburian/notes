---
tags: [speech_to_text]
---
# Spectrogram

Spectrogram is a visualization of frequency decomposition of a sound wave in
time. Each sound signal is decomposed of pure frequencies. We can obtain these
frequencies through [spectral
decomposition](./sound_spectral_decomposition_using_dft.md).

## Spectrogram flavours

There are several variations spectrograms.

### Power spectrogram

Power spectrogram is the simplest spectrogram there is. There are two steps:
1. apply short-time Fourier transform (STFT)
2. square to get signal power

#### 1. Apply Short-Time Fourier Transform

Its DFT applied to a [window](./beginners_guite_to_asr.md) of the original
sound. We obtain a 2d array of complex numbers of a shape ($F$, $W$), where:

- $F$ is the number of frequencies (or *frequency bins*) the DFT computed
  - as is described in
  ['Spetral decomposition using
  DFT'](./sound_spectral_decomposition_using_dft.md), it is equal to half of the
  number of sampled points plus one
  - for STFT it means we get $w/2$ frequencies where $w$ is size of STFT window
- $W$ is the number of windows
  - it is equal to $\frac{T - H}{w} + 1$, where
    - $T$ is the total length of the singal
    - $H$ is the hop-length
    - $w$ is size of the window

#### 2. Square to get signal power

We don't care about the [phase](./beginners_guite_to_asr.md) of the frequencies,
only in their amplitude. We can get the magnitude of the signal (corresponding
to its amplitude) for particular frequency bin by squaring the complex number: 

$$
|a + bi|^2 = \sqrt{(a + bi) \cdot \overline{a + bi}}^2 = a^2 + b^2
$$

The result is the simplest spectrogram called **power spectrogram**.

### Log-amplitude spectrogram or log-spectrogram

Power spectrogram visualizes signal magnitude, yet humans don't perceive loudness
in terms of the magnitude of the signal. They perceive loudness logarithmically:
exponential growth of magnitude equals linear increase in subjective loudness.
Loudness is measured in decibels:

$$
\operatorname{db}(s) = 10 * \log_10{\frac{I(s)}{I_TOH}},
$$

where 
- $s$ is a signal
- $I(s)$ is a signal's intensity -- the squared amplitude of the signal
- $I_TOH$ is a reference intensity of a barely audible sound

By transforming power to decibels we get **log-amplitude spectrogram**.

### Log-amplitude, log-frequency spectrogram

Another quirk in human hearing is that we perceive frequency logarithmically:
exponential frequency equals linear increase in perceived pitch. By
logarithmically scaling the frequency axis of a (log-amplitude) spectrogram, we
obtain (log-amplitude) **log-frequency spectrogram**.

### Mel spectrogram

One particular way how to take into an account the logarithmically perceived
pitch is the Mel (from melody) scale. Mel scale is logarithmic scale with some
constants (found by trial and error), constructed such that given a pitch
difference on the Mel scale, the perceived difference in pitch is the same
throughout our hearing range.

However, together with the Mel scale, frequencies are also weighted to bands,
where band is a mix of frequencies that sound similar to human ear and ergo can
be put in one 'bin' (= band).

The complete list of steps going from *power spectrogram* to Mel spectrogram is:
1. choose number of bands -- more bands equals more precision
2. equally sample points in the *Mel scale* going from the lowest frequency of the
  spectrogram to the highest
3. project these points into Hz by applying inverse of Mel scale and **round** to
   nearest frequency bin
4. create triangular filters (usually called *filter bank*) -- each triangular
  filter is centered around one point and goes through:
    1. point of previous filter at weight 0
    2. the center point of current filter at weight 1
    3. the point of the next filter at weight 0
5. apply the filters to the log-amplitude spectrogram
  - we have filters $M$ of shape (#filters, #frequency_bins)
  - we have log-amplitude spectrogram $Y$ of shape (#frequency_bins, time)
  - we multiply the two matrices $MY$ to get **Mel spectrogram**

We can optionally apply dB scaling of the amplitude to obtain *log-amplitude Mel
spectrogram* or **log-mel spectrogram**.

---
Sources:
  - [Audio Signal Processing for AI (YouTube playlist)](https://www.youtube.com/playlist?list=PL-wATfeyAMNqIee7cH3q1bh4QJFAaeNv0)
