---
tags: [ml,text_to_speech]
---

# Beginners guide to Text-to-Speech

TODOs:
- [ ] StyleTTS 2
- [ ] FastPitch
- [ ] Deep Voice 3
- [ ] TransformerTTS
- [ ] WaveNet
- [ ] Tacotron2
- [ ] Mixer-TTS

In text-to-speech (TTS) tasks the model is asked to predict a waveform based on
an input text. Usually this model is actually whole system of models containing:

- grapheme to phoneme transcriber -- converts spelling (letters) to pronunciation symbols
- mel-spectogram predictor -- predicts [mel-spectograms](./spectogram.md) from
  pronunciation symbols, **the main part of TTS systems**
- vocoder -- converts mel-spectogram to waveforms

There are several types of models:

- autoregressive: (ex. Neural speech synthesis with transformer network, Deep voice 3: 2000-speaker neural text-to-speech)
- 

## Usual pipeline

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

## Tools

- [English grapheme to phoneme generator](https://github.com/Kyubyong/g2p)

## Example models

### Vocoders

- [HifiGAN][HifiGAN] -- small dedicated vocoder used by Efficient Speech

[HifiGAN]: https://proceedings.neurips.cc/paper_files/paper/2020/file/c5d736809766d46260d816d8dbc9eb44-Paper.pdf

### TTS models

- [EfficientSpeech (2023)](./efficientspeech.md) -- efficient TTS system for
  mobile and edge devices
- [FastSpeech (2019)](./fastspeech.md) and [FastSpeech 2
  (2020)](./fastspeech2.md) -- non-autoregressive TTS systems with quick
  inference

### Text to speech models

- [EfficientSpeech](./efficientspeech.md)
