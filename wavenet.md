---
tags: [ml, text_to_speech]
---

[paper]: https://d1wqtxts1xzle7.cloudfront.net/61836013/WAVENET_-_A_GENERATIVE_MODEL_FOR_RAW_AUDIO_-_1609.0349920200120-19152-1e964lf-libre.pdf?1579517363=&response-content-disposition=inline%3B+filename%3DWAVENET_A_GENERATIVE_MODEL_FOR_RAW_AUDIO.pdf&Expires=1731322768&Signature=WvC0jBWdf-WM71WvPF9DbUjfUW~WfPPdTRxF1TGAGSSbP4B-aeK8D-ETKw6djyZU6Xdxi4TDdbD1JQ1BCbYzyReb17ZqTC60sQ0ndEaxhm0xTqz7Z-98bafYgc6gUWfFRgY683GBG1JLnjBRpBS86caHvxYP7RNJtE5OCJKZXf~LQDDWqj6Ichk69ZRl869nBJMN-aP43nKngABL0v-2q8YNcIMpyyaWMzRbx5roGxeG7QZZWiY9RCtqOGdsslpJpKWpXSQtHZParn0b8bzboEH1dgpS88AiA2wUtBvvNllN90WVVt9OwOTXzogLAioIgUMCxqIzqiyrBElFbdSy3w__&Key-Pair-Id=APKAJLOHF5GGSLRBV4ZA

# Wavenet

Wavenet is an autoregressive CNN-based Text-to-Speech (TTS) model introduced by
[van den Oord et al. (2016)][paper].

The paper is very cryptic and hides a lot of details, mainly how are the
individual layers stacked and combined. What we can deduce from the paper is the
following:

#### Wavenet is a CNN model with 1D causal convolutions

This means that layer $l$ combines information from the current and the
previous (in terms of sequence) tokens of the previous layer $l - 1$.

#### Wavenet uses dilated convolutions to increase receptive field

Wavenet's CNN layer $l$ has dilatation $2^l$ up to $l = 9$ with dilatation $2^9
= 512$ and then it starts again from dilatation $2^0$. The reason for the
dilatation is to increase the receptive field. The dilatation is exponentially
increased so that at layer $l$ the model sees exponentially many inputs in the
past.

#### Wavenet quantizes amplitudes and uses softmax

For prediction Wavenet quantizes amplitudes to 256 levels using uniform ranges
in logarithmic scale (as humans perceive loudness). It then uses softmax to
classify particular timestamp to 256 different levels of loudness.

#### Wavenet overall architecture is unclear

Though Wavenet's overall architecture is unclear, there are number of follow-up
articles and implementations:

- [ParallelWavenet (2017)](https://arxiv.org/pdf/1711.10433)
- [Open source implementation](https://r9y9.github.io/wavenet_vocoder/)

One could fairly simply just infer the architecture from the open source
implementation.

