---
tags: [ml, speech]
---

# Voxpopuli

Voxpopuli dataset contains EU parliament events from 2009 to 2020. It was
introduced by [Wang et al.
(2021)](https://aclanthology.org/2021.acl-long.80.pdf).

## Data

### Source

During events, multiple speakers take the stage, each talking in different
language. Some speeches are transcribed, but most are not. Speeches are
interpretted into 24 EU languages.

### Dataset

Voxpopuli is divided into "transcribed" and "unlabelled" datasets. There is
around 1000x more unlabelled data than transcribed data.

Additionally there is also a dataset that maps the speaker's (*source*) speech
into the interpretations (*target speech*). This dataset is called *S*peech
*T*ranslation dataset.

#### Unlabelled

Unlabelled data are segmented into 15-30s segments with less than 2s consecutive
silence using energy-based thresholding VAD algorithm.

#### Transcribed speech

Transcripts' timestamps are adjusted using pyannote diarization pipeline,
mapping the nearest transcript start/end timestamp of a speaker to the detected
speaker change.

The aligned timestamps are used to extract full speech recordings which are
split using an ASR system (TDS models trained with ASG criterion -- TODO) into
<20s segments. Only segments with less than 20% CER are kept. Test and dev sets
get speakers with shortest speeches to ensure maximum speaker coverage for the
given max number of hours per split.

#### ST dataset

TODO
