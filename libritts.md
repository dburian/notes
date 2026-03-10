---
tags: [ml, speech]
---

# LibriTTS

LibriTTS is a speech dataset derived from LibriVox corpus, similarly as
LibriSpeech. Compared to LibriSpeech which is more oriented towards ASR,
LibriTTS has stricter filtering conditions and is designed for TTS.

The dataset was introduced by [Zen et al.
(2019)](https://arxiv.org/pdf/1904.02882).

Although LibriSpeech was originally designed for ASR, it was used for TTS as
well, despite its drawbacks:
- 16kHz downsampled audio
- segmented by silences, which limits learning sentence-level prosody
- lack of capitalization and punctuation
- paragraph-level context of segments is unknown -- we don't know what was
  before the segment or after it, which limits inter-sentence prosody learning
- background noise -- *"clean"* audios were picked by speakers that had low
  WERs, not on single utterance basis

## Data pipeline

The paper is by Google so there is lot of proprietary software.

### Text pre-processing

- Book texts are downloaded from the original source and split into
  paragraphs
- Each paragraph was split into sentences using proprietary algorithm
- Non-standard words were normalized by normalizer (TODO: paper)
  - Non-standard words include:
    - abbreviations
    - numbers
    - currency
    - token sequences that represent entities -- measure phrases, addresses,
      dates, ...

### Audio download

- Audios were downloaded from the original source (1 audio = 1 book chapter) and
  downsampled to 24kHz
- Google's ASR ran on chapter audios
- Each audio was mapped onto the downloaded texts by matching the ASR transcript
  with the text

### Aligning audio

- The authors use a combination of acoustic model and tri-gram language model
- Sentences that don't match the normalized text exactly are filtered out
- timestamps for sentence starts and ends are extracted from the model

### Post-processing

- Filter out sentences that are too long -- possible splitting error
- Filter out *utterances* with large average word duration -- possible severe
  audio-text mismatch
- Normalize audio polarity -- if the wav's mean is negative, multiply by -1
  - Humans are not sensitive to a switch in audio's polarity, but models
    might/are
  - I don't know why exactly this was necessary at all -- I don't know if there
    are many audios with switched polarity, or whether it was just their random
    experiment

- remove begin and end silence (using "silence end-pointer"?)
- compute SNR (using WADA), filter out audios with SNR above threshold

## Dataset stats

- LibriTTS has character count per segment distribution similar to the original
  source -- LibriVox -- , whereas LibriSpeech has much higher peak.
- LibriTTS filtering causes the distribution of audio length per speaker to be
  shorter. Also, LibriTTS's distribution has wider variance.




