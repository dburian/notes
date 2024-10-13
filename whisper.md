---
tags: [ml,speech_to_text]
---

# Whisper

Whisper is a pre-trained speech to text model introduced by [Radford et al.,
2020](https://arxiv.org/pdf/2212.04356). Compared to other STT models like
[HuBERT](./hubert.md) or [wav2vec](./wav2vec.md), Whisper aims to be ready out
of the box for zero-shot transcription.

Instead of training an encoder, authors chose to train speech transcriber
end-to-end. This means that the model cannot be used for other speech-encoding
tasks. The authors argue that pre-trained encoders always lack equally trained
decoders and that finetuning decoders only on dedicated datasets decreases the
system's robustness.

## Architecture

Whisper is [Transformer encoder-decoder](./transformer.md) model that digests
audio via fairly small 2-layer convolution layers. The model is trained to do
multiple tasks:
1. multilingual transcription (focusing on English though)
2. Any language to English translation
3. Speech detection (Is somebody speaking or not?)

Additionally the model is trained to detect the language of the input, on some
inputs even detect quantized timestamps for each token. To control these
predictions the decoder receives a rather complicated sequence of tokens with
several special ones (apart from the utterance). The special tokens give the
model place to predict
- if somebody is speaking,
- the language,
and instruct it if it should
- translate,
- predict timestamps or not.
