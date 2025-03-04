---
tags: [ml, text_to_speech]
---

# Tacotron 2

RNN-based TTS model introduced by [Shen et al.
(2017)](https://arxiv.org/pdf/1712.05884). Tacotron 2 predicts mel
[spectrogram](./spectrogram.md) frames from characters of the input text. Mel
frames are predicted **autoregressive-ly**, where the prediction of the next mel
frame is dependent on the previous one. Therefore the model can only predict one
frame at a time, which makes inference considerably slow.

In its days the model was very strong benchmark. When it was surpassed it was
still useful, since the trained model provides mapping between input characters
(or phonemes) and mel frames.


## Architecture

The model is composed of an RRN encoder and RNN decoder, with
[attention](./attention_in_rnn.md) between them. Tacotron 2 used slightly
adjusted attention, to make attention weights more continuous throughout the
output sequence.
