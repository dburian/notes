---
tags: [ml, text_to_speech]
---

# WaveGlow

WaveGlow is a TTS vocoder introduced by [Prenger et al.
(2018)](https://arxiv.org/pdf/1811.00002).

The model is a combination of [Wavenet](./wavenet.md) and [Glow](./glow.md).
It's a [flow-based network](./flow_networks.md) utilizing

- Glow's 1x1 invertible convolutions and
- Wavenet's architecture to generate waveform from mel-spectogram

## Architecture

WaveGlow is in core a flow-based network which repeats the following layers:

- 

