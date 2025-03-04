---
tags: [ml,text_to_speech]
---

# StyleTTS 2

StyleTTS is a second generation of [StyleTTS](./styletts.md). The
second version was introduced by [Li et al.
(2023)](https://arxiv.org/pdf/2306.07691).

There are 4 features that differentate StyleTTS 2 from its previous version:

- Style Diffusion -- style vector is generated with diffusion from input
  phonemes, meaning the system can produce speech without reference audio

- StyleTTS 2 predicts waveform, not just mel-spectrogram. Meaning its an
  *end-to-end* system, that **can be trained all at once**.

- To make the end-to-end training possible, the authors designed a
  differentiable duration modelling that allows the gradient to backpropagate.

- Adversial Training through Speech Language Models (*SLM*s) -- the synthesized
  audio is transcribed into tokens, fed into SLM and a discriminative head
  judges if the waveform is real or fake. The loss signal then travels to the TTS
  model. This is similar to [GANs](./generative_adversial_networks.md).


## Long-form audio generation

To allow for longer audios with fluency, the authors propose to concatenate
waveforms generated with running mean of the style vector. Meaning the
algorithm goes as follows:

```python
wavs = []
old_style_vector = None
for chunk in text:
  style_vector = style_model(chunk)
  if old_style_vector is not None:
    style_vector = alpha * style_vector + (1 - alpha) * old_style_vector

  wavs.append(generate_audio(chunk, style_vector))
  old_style_vector = style_vector

return torch.concat(wavs)
```

The reason behind using a combination of the previous and current style vector
is that
- using only *the current style vector* would lead to inconsistent, non-fluent
  speech while
- using only *the first style vector* would sound monotonic and robotic.

The authors found a good value for `alpha` to be `0.7`. There is an example
[here](https://styletts2.github.io/#long).





