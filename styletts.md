---
tags: [ml, speech_to_text]
---

# StyleTTS

Style TTS is a text-to-speech system introduced by [Li et al.
(2022)](https://arxiv.org/pdf/2205.15439). StyleTTS allows synthesis of speech
acording to *style vector* -- characterisation of the prosody. This allows to
generate artificial speech that matches a reference audio.

## Architecture

Style TTS is a system of 8 modules:

1. text encoder $T(t) \rightarrow h_\text{text}$ -- encodes input phonemes $t$
   into representations
2. style encoder $E(x) \rightarrow s$ -- extracts style vector from an audio
   sample
3. speech decoder $G(h_\text{text} \cdot a_\text{pred}, E(\tilde{x}),
   \hat{p}_\tilde{x}, \hat{n}_\tilde{x}) \rightarrow x_\text{pred}$ -- predicts mel
   spectogram given
    - aligned phonemes representations,
    - style vector,
    - desired pitch and
    - desired energy
4. duration predictor $S(h_\text{text}, s) \rightarrow d_\text{pred}$ -- predicts
   duration from phoneme representation and style vector, predicted duration is
   then used to generate alignment $a_\text{pred}$.
5. prosody predictor $P(h_\text{text}, s) \rightarrow \hat{p}_x, \hat{n}_x$ --
   predicts energy and pitch from phoneme representations and style vector
6. discriminator $D$ -- is used to further train $G$
7. text aligner $A(x, t) \rightarrow a_\text{algn}$-- extracts true alignment
   from an audio input
8. pitch extractor $F(x) \rightarrow p_x$ -- extracts true pitch from an audio
   sample

Modules 1.-3. compose speech generation system, 4. and 5. are prosody
predictors, 6.-8. utility modules used for training $G$.


### Training

During training several moduels are trained.

**Speech decoder** is trained to match the original mel spectogram, along with
other losses that take into an account the output of the discriminator. For
example there is the adversarial loss that operates on true input $x$ and
predicted mel spectogram $\hat{x}$ similar as in
[GANs](./generative_adversial_networks.md).

**Style encoder** is trained jointly with speech decoder.

**Duration predictor** is trained to reflect extracted alignment $a_\text{algn}$
from the input.

**Prosody predictor** is trained to match output of pitch extractor $F$ and the
true energy $n_x = ||x||$.

**Text aligner** is trained on ASR task with annotated phoneme sequences per
audio sample.

### Inference

During inference the prosody and duration predictor's outputs are used as an
input to speech decoder, along with a style vector extracted from a reference
audio.


