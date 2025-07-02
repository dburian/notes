---
tags: [ml,speech_to_text]
---

# Beginners guide to ASR

I write this, since I'm a beginner in speech ML (TTS, ASR/STT). This is what has
helped me get started.

## Some nice-looking models (as of Q1 2025)

### Overview


| Model | Params | Average WER in HF leaderboard | RTFX | Comercial license | Czech in training data | Training data size | Training steps | Training GPUs |
| ----- | ------ | ----------------------------- | -----| ----------------- | ---------------------- | ------------------ | ------------- | ------------- |
| [Whisper v3 large](https://huggingface.co/openai/whisper-large-v3) | 1550 M | 7.44 | 145 | ✔️ | 192h asr + 401h trans. | 1Mh of weakly labelled, 4Mh of pseudo-labelled | 1406 M | ? |
| [Whisper v3 turbo](https://huggingface.co/openai/whisper-large-v3-turbo) | 809 M | 7.83 | 200 | ✔️ | 192h asr + 401h trans. | 1Mh of weakly labelled, 4Mh of pseudo-labelled | 1406M + ~900M finetune | ? |
| [Canary 1b flash](https://huggingface.co/nvidia/canary-1b-flash) | 883M | 6.35 | 1045.75 | ✔️ | ❌| 85kh of data | 200K steps, optimized | 128 A100 80GBs |
| [Canary 1b](https://huggingface.co/nvidia/canary-1b) | 1B | 6.5 | 235.34 | ❌ | ❌| 85kh of data | 150k steps | 128 A100 80GBs |
| [Canary 180M flash](https://huggingface.co/nvidia/canary-180m-flash) | 180M | 7.12 | 1233 | ✔️ | ❌| 85kh of data | 219k steps, optimized | 32 A100 80GBs |
| [Parakeet TDT 1b](https://huggingface.co/nvidia/parakeet-tdt-1.1b) | 1.1B | 7.01 | 2390 | ✔️ | cv | 64kh | 360M steps? | ? |


### One-by-one

- [Whisper (2022)](https://github.com/openai/whisper)
  - v3 large is the latest, greatest
    - multilingual, 1.5B
  - v3 turbo is the latest, smallest
    - multilingual, 800M
    - faster inference than v3, at the cost of some quality
  - [How to finetune Whisper models blog post by HF](https://huggingface.co/blog/fine-tune-whisper)
  - 💰comercial license
- [Distil-Whisper](https://huggingface.co/distil-whisper/distil-large-v3.5)
  - [paper](https://arxiv.org/abs/2311.00430)
  - faster than v3 turbo, with different balance
  - 💰comercial license
- [Nvidia canary
  (2024)](https://www.isca-archive.org/interspeech_2024/puvvada24_interspeech.pdf)
  - 1B sits on top of HF Open ASR leaderboard
    - 💰non-comercial license
  - there is also 180M model that does pretty well
  - there are also flash versions
- [Parakeet TDT 1.1B](https://huggingface.co/nvidia/parakeet-tdt-1.1b)
  - top of HF leaderboard
  - composition of two concepts:
    - [Fast Conformer](https://arxiv.org/pdf/2305.05084)
    - [Something called TDT](https://arxiv.org/abs/2304.06795)
  - 💰comercial license
- [FireRedASR (2025)](https://github.com/FireRedTeam/FireRedASR)
- [Wav2vec (2020)](https://huggingface.co/docs/transformers/model_doc/wav2vec2)
  - worse than Whisper on some random benchmark I found online, considerably so
    its probably true
- [Phi-4 multimodal (2025)](https://arxiv.org/pdf/2503.01743)
  - from microsoft
  - multimodal-multi task model, 3.8B
  - its instruct version is SOTA in HF ASR leaderboard
  - 💰comercial license
- [CrisperWhisper](https://huggingface.co/nyrahealth/CrisperWhisper)
  - v3 large Whisper finetuned to produced more verbatim text
  - 👎 non-comercial license

## Benchmarks

- [HF Open ASR
  leaderboard](https://huggingface.co/spaces/hf-audio/open_asr_leaderboard)

## Metrics

- Inverse of Real Time Factor (RTFX): number of seconds of audio processed /
  processing time
- Word Error Rate (WER) -- word edit distance over the number of true words:

$$
\operatorname{WER}(y_\text{pred}, y_\text{true}) = \frac{
  \operatorname{ed}_w(y_\text{pred}, y_\text{true})
  }{
    |y_\text{true}|
  }
$$

- Character Error Rate(CER) -- edit distance over divided by the number of true
  characters


---

## Common libraries

- loading files: `librosa`

## Common values

### Sampling rate

For basic speach 16kHz is enough. But latest models go from 22kHz to 44kHz.

## Common metrics


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
