# Beginners guide to ASR

I write this, since I'm a beginner in speech ML (TTS, ASR/STT). This is what has
helped me get started.

## Common libraries

- loading files: `librosa`
- processing input data: `spark`

## Common values

- sampling rate: 16k

## Common metrics

- Word Error Rate (WER) -- word edit distance over the number of true words:

$$
\operatorname{WER}(y_\text{pred}, y_\text{true}) = \frac{
  \operatorname{ed}_w(y_\text{pred}, y_\text{true})
  }{
    |y_\text{true}|
  }
$$

## Glossary

### Frame

Part of the signal in time in between two timestamps. 

### Window, Hop length

Some functions are **windowed**, meaning they are applied repeatedly for different
frames. *Windows are part of frames*, where window size doesn't have to be equal,
but often is to size of frame. The distance between consecutive windows is
called **hop length**, i.e. the overlap of two consecutive windows.


### Frequency, Phase, Amplitude

If we take *pure sound* we can study its 3 basic properties:

$$
a * \sin(2\pi * (ft - \phi),
$$

where

- $a$ is **amplitude** -- height of signal's peak (loudness of it)
- $f$ is **frequency** of the sound -- the number of cycles per second
- $t$ is time in seconds
- $\phi \in [0, 1)$ is **phase** of the signal -- horizontal shift of the sine
  wave


### Amplitude envelope

The max of waveform for each frame. Gives an idea of loudness of a signal.
Sensitive to outliers.

$$
\text{AE}_t = \max_{i \in t} s(i)
$$

where

- $t$ - frame index
- $i$ - timestamp index in a frame
- $s(i)$ - amplitude at timestamp $i$

### Root-mean-square energy

The root of mean of square amplitude within each frame. Describes mean loudness
within a frame, with bias towards high amplitude (loudnesses)

$$
\text{RMS}_t = \sqrt{\sum_{i \in t} s(i)^2}
$$

where

- $t$ - frame index
- $i$ - timestamp index in a frame
- $s(i)$ - amplitude at timestamp $i$

### Zero-crossing rate

Number of times signal crossed `y=0` line (going from positive to negative or
vice-versa). ZCR can tell us about the sound's:

- percussiveness vs pitch -- both have high ZCR, but percussive sound's ZCR
  varies more
- presence/absence of voice -- voiced sounds tend to have higher ZCR
- pitch -- assuming monotonic pitch, ZCR would correspond to frequency

$$
\text{ZCR}_t = \frac{1}{2}
  \sum_{i \in t} |
    \operatorname{sgn}\left(s(i)\right) -
    \operatorname{sgn}\left(s(i+1)\right)
  |
$$

where

- $t$ - frame index
- $i$ - timestamp index in a frame
- $s(i)$ - amplitude at timestamp $i$

## Common approaches

The common approach is to
1. visualize the signal
2. apply CNN to windows of the visualization and
3. contextualize CNN representations of the windows either by RNN or Transformer

### Visualization of signal

Traditionally signal was visualized using [Mel-Frequency Cepstral Coefficients
(MFCCs)](./cepstogram.md), which filters out sound's details and leaves only the
main identification points. However, its a heavy feature processing which leaves
out detail, that more powerful architectures could use. And so, recently, there
are models that use pure sound wave such as [Wav2vec](./wav2vec.md) or
[HuBERT](./hubert.md).

## Good sources of info

- [Audio Signal Processing for ML
  (YouTube)](https://www.youtube.com/playlist?list=PL-wATfeyAMNqIee7cH3q1bh4QJFAaeNv0)
