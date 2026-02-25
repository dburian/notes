---
tags: [ml, speech]
---

# Granary speech dataset

Granary is large open-source speech dataset containing around 600k filtered
hours of 25 European (+Russian, Ukranian) languages. It was introduced along
with the filtering pipeline by [Koluguri et al.
(2025)](https://arxiv.org/pdf/2505.13404).

Granary contains both ASR and automatic speech translation (AST) datasets.

## Pipeline implementation

The pipeline implementation is called
[NeMo-speech-data-processor](https://github.com/NVIDIA/NeMo-speech-data-processor/tree/main).

In the NeMo framework, the main **the** dataset format are `.jsonl` manifests
with paths to audio files.

The pipeline consists of *processors*. Each processor is given
- an input manifest path and
- an output manifest path.

When the previous processor is finished, the new one is called to:
1. take samples from the input manifest (the output manifest of the previous processor)
2. process them
3. save them to the output manifest (the input manifest of the following
   processor)

It's really simple as that.

### Granary cleaning pipeline implementation

Granary's pipeline is just a
[configuration](https://github.com/NVIDIA/NeMo-speech-data-processor/blob/main/dataset_configs/multilingual/granary/config.yaml)
of the NeMo Speech Data Processor.

## Source datasets

Granary is just a filtered concatenation of 3 datasets with CC license: YODAS,
YouTube-Commons, MOSEL. Until I write dedicated note about these datasets
(TODO), here is a brief summary of them:

- YODAS -- 500k hours, 100+languages, annotations from YouTube subtitles, noisy
- YTC -- similar to YODAS, skewed toward English (70%)
- MOSEL -- comprises VoxPopuli and LibriLight, pseudo-labeled using Whisper

## Filtering

All audio was re-sampled to 16kHz and converted to mono-channel. The target
length of individual segments was 40 seconds.

Though the paper describes the pipeline, I found it best to reference the
pipeline configuration mentioned above.

ASR pipeline steps (high-level):
1. convert to 16kHz, 1 channel
2. Whisper prediction -- obtaining language ids (LIDs) per audio
3. filtering out audios with different predicted LIDs than assumed or with low
   LID prediction probablity
4. first Whisper ASR inference -- obtaining segment timestamps
5. splitting into segments
6. second Whisper ASR inference -- obtaining word-level timestamps and ASR
   final transcript
7. Whisper hallucination detection and filtering
8. punctuation restoration using Qwen 2.5
9. filtering by character rates -- eliminating samples with abnormally low or
   high character ratios

AST pipeline:
1. The ASR pipeline
2. word count filtering -- eliminate the sample if the ratio between source and
   target language words is too high/low
3. character histograms checks -- compares character histograms to pre-trained
   character histograms per language and eliminates outlier samples
4. FastText language detection and filtering -- another LID filtering, now with
   different model
5. machine translation quality estimation -- a model that is able to estimate the
   quality of the translation without a reference is used to filter out "bad"
   translations

### Alignment/Segmentation

Authors tested segmentation using 2 ASR models: Whisper, and Parakeet. The
authors found that:
- Whisper performed worse on word-level alignment, necessitating 2 passes:
    - first identifying the speech/non-speech boundaries,
    - second timestamping the words inside speech segments
- Parakeet worked well (but was only used for LibriLight dataset for some
  reason, and isn't part of the config implementation)

The authors also mention using VAD, but I didn't find it in the configuration.

## Notes

- Whisper has reduced accuracy for low-resource languages and is prone to
  hallucinations.
- Whisper turbo is more prone to hallucinations than its larger variant.
- Whisper struggles with noise and non-speech segments.
