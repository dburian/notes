---
tags: [ml, speech_to_text]
---

# FastSpeech 2

FastSpeech 2 is a second generation of STT model [FastSpeech](./fastspeech.md).
FastSpeech 2 was introduced by [Ren et al.
(2020)](https://arxiv.org/pdf/2006.04558). The second generation improvements
are:

1. The model not-only predicts length, but also pitch and energy.
2. No student-teacher training. Mel-spectograms are learned directly, and
   phoneme duration is extracted using ...

## Architecture

The model architecture is similar as in the previous generation: two stacks of
transformer layers, delimitted by '*Variance Adapter*', which consists of:

1. Duration Predictor: repeats each phoneme according to its predicted
   duration (in mel frames).
2. Pitch Predictor: predicts pitch and adds the results back to the hidden
   representation of the mel frame
3. Energy Predictor: dtto for energy


### Duration predictor

In FastSpeech *2* Duration predictor is trained using ground truth extracted
using [Montreal forced alignment
(MFA)](https://www.isca-archive.org/interspeech_2017/mcauliffe17_interspeech.pdf)
(TODO: how this works?).
