---
tags: [ml,text_to_speech]
---

# Beginners guide to Text-to-Speech

TODOs:
- [ ] flow-based models, e.g.
    - VITS (2021)
    - Flowtron (2020)
    - Glow-tts (2020)
- [ ] StyleTTS 2 (2023)
- [ ] Deep Voice 3 (2017)
- [ ] TransformerTTS
- [ ] Mixer-TTS (2021)
- [ ] Neturalspeech (2022)

In text-to-speech (TTS) tasks the model is asked to predict a waveform based on
an input text. Usually this model is actually whole system of models containing:

- grapheme to phoneme transcriber -- converts spelling (letters) to
  pronunciation symbols

- mel-spectogram predictor -- predicts [mel-spectograms](./spectogram.md) from
  pronunciation symbols, **the main part of TTS systems**

- vocoder -- converts mel-spectogram to waveforms

## Types of mel-spectogram generators

There are several types of models for mel-spectogram generation.

### Autoregressive

> Predicting next mel frame based on the previous

- Classical approach to speech synthesis which, initially, gave the best
  results
- RNNs with attentions or causal CNN architectures
- the big drawback is **slow inference**, which is why these models are not
  really used anymore
- examples: [Tacotron 2](./tacotron_2.md) or [WaveNet](./wavenet.md)

### Parallel

> Predicting all frames at the same time.

- Typically [Transformer](./transformer.md)-based models producing all
  frames in a single go.
- They need to resolve the problem of '*How many frames should I predict?*'
    - Typically they use
        - attention from a pre-trained autoregressive model such as Tacotron 2 or
        - [Montreal Forced Aligner](./https://www.isca-archive.org/interspeech_2017/mcauliffe17_interspeech.pdf)
- examples: [FastSpeech](./fastspeech.md), [FastSpeech 2](./fastspeech2.md),
  [EfficientSpeech](./efficientspeech.md), [FastPitch](./fastpitch.md)

## Types of vocoders

Vocoders need to predict very high-definition output: second of speech is
typically between 16k and 22k scalars. Ergo this is a realm of generative
models.

### GANs

- Vocoders based on [Generative Adversial Networks or GANs](./generative_adversial_networks.md)
- examples: [HifiGAN](./hifigan.md)

### Flow-based

- Vocoders based on [Flow networks](./flow_networks.md)
- examples: [WaveGlow](./waveglow.md)


## Glossary

- phoneme: smallest speech sound of a particular language (pronunciation)
- grapheme: smallest unit of writing system (letters)

> ### grapheme vs phoneme
> graphemes:
>   - the quick brown
>   - fox jumps over
>   - the lazy dog
>
> phonemes:
>   - DH AH0 K W IH1 K B R AW1 N
>   - F AA1 K S JH AH1 M P S OW1 V ER0
>   - DH AH0 L EY1 Z IY0 D AO1 G

- prosody: properties of larger phonetic segments such as rhythm, stress and
  intonation
- vocoder: turns mel-spectograms into waveforms

## Evaluation metrics

### Mean Opinion Score (MOS)

Mean Opinion Score (MOS) is the mean of opinion scores given by annotators. The
annotators can score a recording on a **categorical** scale from 1-5, 5 being
'*excelent*', 1 being '*bad*'. To compare model's generated audio, the authors
should also disclose what was the MOS of ground truth (typically $\sim$4.5).

## Tools

- [English grapheme to phoneme generator](https://github.com/Kyubyong/g2p)

### TTS models

- [StyleTTS (2022)](./styletts.md) and [StyleTTS 2 (2023)](./styletts_2.md) --
  SOTA models incorporating 'style vector' into the waveform generation

