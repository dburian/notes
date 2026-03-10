---
tags: [ml, speech]
---

# LibriTTS-R

LibriTTS-R is a restored version of [LibriTTS dataset](./libritts.md). It was
introduced by [Koizumi et al. (2023)](https://arxiv.org/pdf/2305.18802).

While [LibriTTS](./libritts.md) improved the data quality for TTS, its sound
quality isn't at as good as studio-recorded datasets (of smaller scale) like
LJSpeech. The authors argue that this lower quality makes it harder to match
ground-truth MOS scores with models trained on it, despite them being much lower
than LJSpeech's ground-truth MOS scores.

Interestingly the authors mention that noisy TTS datasets are useful for
*"advanced TTS model training"* (TODO: read linked papers). But still see the
value in high-quality TTS dataset.

## Data pipeline

The authors basically just run LibriTTS data through Miipher, which in turn uses
more models. This makes it difficult to see what data it cleans and how it does
it. But it seems that authors **did not filter out any data**.

The authors trained Miipher on multilingual **proprietary** dataset with 2.6k
hours with 670 clean hours. They simulated background noise, and number of
artifacts including room impulse response (RIR).
