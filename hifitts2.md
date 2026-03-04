---
tags: [ml, speech]
---

# HifiTTS 2

Speech dataset for high-bandwidth TTS. Introduced by [Langman et al.
(2025)](https://arxiv.org/pdf/2506.04152).

- Derived from MLS (derived from LibriVox == open audiobooks) which faces the
  following problems:
  - no punctuation and capitalization (PC) in the original text
  - audios stored as 16kHz even though the audio's sr is different
  - utterances range from 10-20secs
  - text transcriptions may not fit the utterances *exactly*
  - unreliable speaker labels

## Data pipeline

### Text preprocessing

- Fix PC:
  - download original book text, remove PC, match with the dataset transcript
    -> replace transcript with original text
    - Fixes **87%**
  - PC prediction using DistilBERT

- replacement of common html strings as html tags or `nbsp`
- the authors apply NeMo normalization

### Audio processing

- Download original 48kHz audio, downsample to 44kHz
- energy based silence trimming:
  - threshold 50dB
  - at most 0.5s of silence before and after each utterance

- bandwidth estimation:
  - maximal frequency is estimated from power spectrum: the maximum frequency
    with larger dB than -50dB relative to the peak value in the spectrum
  - the maximum frequency must be smaller than [Nyquist
    frequency](./nyquist_shannon_sampling_sampling_theorem.md) = half the
    sampling rate
  - this is fine for 22kHz audio, but 44kHz voice audio often has small power of
    larger frequencies since speech just doesn't have those frequencies
      - ergo the above algorithm would often estimate smaller maximum frequency
        than the true one
      - to address this, the authors check the 13kHz frequency -- should be
        empty for 22kHz, but non-empty for 44kHz (you still hear speech at
        13kHz)
      - the heuristics that audios recorded at sampling rates between 22kHz and
        44kHz (e.g. 37kHz) are very rare. So if it is higher than 22kHz, it is
        44kHz.

### Segmentation

- 10-20s is problematic for TTS models -- we would wish for a more balanced
  (broader) distribution to avoid biases of the model
- authors used NeMo Forced Aligner with Parakeet-TDT-CTC
- the authors try to segment text by sentences:
  - cut is made at each dot followed by 0.08secs of silence (ignores dots for
    abbreviations and others)
  - if more such dots exists in an utterance, authors use the longest (or random
    if they have the same duration)

- this makes the span still below 20s but now the duration distribution looks
  like a balanced bell curve with peak at 8-9s

### Text validation

- each segment was predicted with Parakeet-TDT-CTC and utterances with CER
  higher than 100% were removed
  - the authors state that any lower threshold is a trade-off between volume and
    intelligibility of the training data

### Speaker diarization

- each segment might have multiple speakers, since audio books can be narrated
  by multiple speakers
- the authors use Softformer -- end-to-end neural model for speaker diarization
- the authors annotate the segments with multiple speakers and leave them in the
  dataset


## Experiments

- The authors train KoelTTS on HifiTTS (283h) , LibriTTS (539h), and HifiTTS2
  (30k hours)
  - They filter HifiTTS2 audios to those:
    - with CER equal or smaller than 3%
    - or speaker similarity between context audio (=same speaker label, different
      utterance) larger than 0.6

- training with HifiTTS2 (4.9k speakers) leads to better results in Speaker similarity (using
  Titanet-Small), but +- equal estimated MOS (SQUIM-MOS) than training on
  LibriTTS (2.2k speakers) or HifiTTS (10 speakers)

- aside from the improvement in CER/WER (Parakeet-TDT) of HifiTTS2 (which was
  expected due to similar model used for pre-processing) the biggest improvement
  was in speaker similarity on unseen speakers

- combining LibriTTS, and HifiTTS still suffers from few speakers compared to
  HifiTTS2

- combining all 3 datasets yields similar MOS, but a bit better speaker
  similarity

- IMHO the improvement is not that significant since HifiTTS2 has double the
  speakers and roughly 60x hours than LibriTTS
