---
tags: [ml, text_to_speech, hmm]
---

# Montreal Forced Aligned

*Montreal Forced Aligner* (*MFA*) is a speech tool primarly used for aligning
phonemes to sections of audio. The tool was introduced by [McAuliffe et al.
(2017)](https://www.isca-archive.org/interspeech_2017/mcauliffe17_interspeech.pdf).
The tool has a [docs
website](https://montreal-forced-aligner.readthedocs.io/en/latest/).

## Aligning model

The model that aligns phonemes to audio parts is fairly complicated, and so I
know only how it might approximately work. The model is based on [Hidden Markov
Mohdel (HMM)](./hidden_markov_model.md) with Gausian Mixtures Model (GMM) for
modeling probabilities of audio clips. The model is given a phonetic dictionary
and learns the most probable sequence of phonemes that produced given sounds. My
suspicion is this is learned using Expectation-Maximization algorithm, symilary
how translation systems used to be trained.

## User perspective

MFA is now a set of tools.

### Align a corpus

To align a corpus you need to follow few steps:

1. Get the corpus to the right [format](https://montreal-forced-aligner.readthedocs.io/en/latest/user_guide/corpus_structure.html#specifying-speakers)
2. Download a [dictionary](https://mfa-models.readthedocs.io/en/latest/dictionary/index.html) mapping words to phonemes
3. [Validate](https://montreal-forced-aligner.readthedocs.io/en/latest/user_guide/data_validation.html#mfa-validate) your corpus and dictionary
4. 
