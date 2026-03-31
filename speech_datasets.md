---
tags: [ml, speech]
---

# Speech datasets

This is a summary of large datasets for speech:

1. \[ASR\] LibriSpeech -- 1k hours of english audiobooks

1. \[ASR\] Mozilla Common Voice -- crowd-sourced multi-lingual dataset

1. \[TTS\] LJSpeech -- single-speaker speaker reading

1. \[TTS\] CSTR VSTK Corpus -- multi-speaker, english, same sentence in various
   accents

1. \[ASR\] TED-LIUM (v3) -- TED talks

1. \[ASR\] VoxCeleb -- extracted YT 1M utterances of celebrities

1. \[ASR\] GigaSpeech -- 10k+ hours collected from audiobooks, podcasts, YouTube

1. \[ASR\] People's Speech -- 30k of transcribed speech with commercial license

1. \[ASR\] CHiME -- noisy ASR datasets from people's recordings

1. \[ASR\] Switchboard-1 -- collection of 2.4k telephone conversations,
   disfluencies (hesitations)

1. \[ASR\] Multi-lingual LibriSpeech -- extension of LibriSpeech for 8 languages
   with 50k hours

1. \[ASR\] MUSAN -- huge library of music, speech, noises

1. \[phonetic ASR/TTS\] TIMIT -- small time-aligned phonetic dataset

1. \[TTS\] [LibriTTS](./libritts.md) / [LibriTTS-R](./libritts_r.md) --
   multi-speaker, english, from LibriSpeech filtered with AI to clean up the
   audio

1. \[foundation TTS/ASR\] [VoxPopuli](./voxpopuli.md) -- 400k unlabeled hours,
   1.8k labeled, multi-lingual, european parliament

1. \[test TTS\] FLEURS (Few-shot Learning Evaluation of Universal
   Representations) -- small testing multilingual dataset with the same set of
   sentences

1. \[multi-speaker ASR\] AMI Meeting Corpus -- 100h of business meetings,
   multi-speaker

1. \[conversational ASR\] Fisher English Training Speech -- collection of
   telephone conversatinons

1. \[ASR translation\] CoVoST 2 -- translations of english speech

1. \[ASR\] [Granary](./granary.md) -- 1Mh across 25 European languages

1. \[pre-training foundational ASR/TTS\] Libri-light -- 60k hours of unlabelled
   speech

1. \[TTS\] M-AILABS Speech Dataset -- 1k+ hours of european languages

1. \[low-resource ASR\] BABEL (IARPA) -- low-resource languages

1. \[TTS\] CSS10 -- 10 languages, single-speaker (multilingual euquivalent of
   LJSpeech)

1. \[ASR\] DiPCo (Dinner Party Corpus) -- multi-speaker speech during dinner
   party, lots of background noise

1. \[ASR translation\] GigaST -- english speech to chinese/german text

1. \[global ASR coverage\] Unsupervised People's Speech -- 1Mh across 89+
   languages, very inclusive (includes low-resource languages)

1. \[quality TTS\] HifiTTS -- high bandwidth dataset, 10 speakers, 292 hours

1. \[quality TTS\] [HifiTTS2](./hifitts2.md) -- continuation of HifiTTS,
   enlarged to 5k speakers and 36k hours

1. \[global TTS/ASR\] Massively Multilingual Speech -- 1000k languages focused
   on low-resource languages, few speakers, formal speech

1. \[ASR\] MLS -- large scale multilingual dataset for speech research (derived
   from LibriVox)


## Filtered

Datasets that have been filtered:
- [HifiTTS2](./hifitts2.md) -- CER/WER, bandwidth estimation, silence trimming,
  segmentation
- [LibriTTS](./libritts.md) -- SNR, filter out segments with a lot of
  reverbaration
- GigaSpeech -- multi-filtering (?), ASR confidence scores, character to
  duration ratios, forced alignment learned from other datasets, iterative
  training process
- LibriSpeech -- alignment using acoustic models
- VoxCeleb -- alignment w/ video, video-oriented tools like face detection and
  lip sync
- People's Speech -- ASR alignment by threshold
- [VoxPopuli](./voxpopuli.md) -- ASR CER threshold
- [Granary](./granary.md) -- Whisper hallucination detection, WER, language id filtering


### Filtering tools

- Miipher -- generative denoiser
  - LibriTTS
- Whisper -- ASR model
  - Gigaspeech, People's Speech, VoxPopuli
- Kaldi -- phonetic precise alignment
  - LibriSpeech
- VAD -- for silence detection
  - VoxPopuli


TODO:
- https://arxiv.org/pdf/2409.03283
