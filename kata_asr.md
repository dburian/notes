# Katka's presentation on ASR

## Intro -- The Human system
- following how human do it:
  - brain: encoding information into phonemes
  - vocal tract: creating audio
  - channel (air that transmits the sound): adds noise
  - ear: does spectral analysis
  - brain: decodes spectrum into information

### Speaker's brain
- phonemes are defined as being distinct according to human
- phonemes are defined in terms of how we produce the sound in our vocal tract
- prosody is super-segmental -- global modifier of multiple phonemes (which are
  segmental)

### Vocal tract
- prosody defines
  - amplitude
  - durations
  - base frequency
- for TTS prosody was recognized as very important, but in ASR its largely
  ignored
- phone is a realization of a phoneme
- typical duration of a phone is 80ms
- phone depends on context -- e.g. how drunk the speaker is, how our vocal tract
  moves from one phone to other
- phase is assumed to be 'accidental' or 'useless' information -- nobody knows
  what influences it and how

### Noise
- noise can be additive (e.g. additional (constant) noise) or convolutive (e.g.
  echo -- multiplies each other)

### Ear
- ear does spectral analysis -- that is the "motivation" for using FFT when
  processing speech
- frequency sensibility -- humans are more sensitive to the lower frequencies,
  motivation for the mel

### Listener's brain
- the brain is trained to work with damaged signals -- it targets the 'peak' of
  phoneme, the period when phoneme is the clearest
- the brain adapts to new speakers very quickly -- about 2 syllables
- the brain uses higher-order knowledge to classify a phoneme
- the brain uses visual modality to understand language

## ASR classical systems

- are hybrid
- steps:
  - noise detection -- audio preprocessing
  - feature extraction
    - separate signal into chunks (25ms window, 10ms overlap)
    - spectrum analysis of chunks
    - the result is spectra for each chunk -- spectrogram/mel-spectrogram
    - plus first ($\delta$) and second ($\delta\delta$) time-derivatives
    - can be processed further:
      - PCA
      - speaker or domain normalization
      - transformation trained unsupervised by NN
  - acoustic model -- recognizes acoustic units (basically contextualized
    phonemes -- contextualized with neighboring text)
    - composed of 2 sub-models:
      - I became confused about this...
      - classifier: e.g. Gaussian Mixture Model, or NN -- maps features to
        acoustic units
      - aligner: HMM -- models transition between from acoustic features

      We need to model the transition from one acoustic unit to another. For
      that we use HMM -- each acoustic unit is modelled by 3 states
    - we train using Expecation-Maximization:
      - alternating between alignment (HMM) and the ...

    - without constraints HMM can be very big -- we constraint it using
      dictionaries

- issues:
  - complex
  - independence assumptions that don't hold (statistical independence of each
    frame)
  - you need secondary information -- lexicon (maps words to phonemes),
    dictionaries (list of words)
- advantages:
  - alignment as a byproduct

- sometimes used in when you need very small model (on a mobile device), but
  otherwise not


## End-to-End ASR systems

- surpassed hybrid systems in 2016
- view of ASR as sequence-to-sequence -- frame to letter
- makes ASR systems more black-box
  - it's harder to train only on text, add vocabulary manually

- explicit alignment -- predictions on per-frame basis
  - predictions need to post-processed using CTC, RNN-T, RNA (latter is better)
  - not very precise -- 'blank' frame might contain some sound of previous
    symbol
  - streaming of input is possible
- implicit alignment -- models process all the frames at once to predict symbol
  - no streaming possible??
  - stronger than explicit alignment

- explicit alignment models:
  - Listen, Attend, Speak
    - hierarchical BLSTM encoder, LSTM decoder with Attention
  - Improvements:
    - multi-task learning strategy -- CTC and using Attention
    - Transformer (2020)
    - Conformer

## Current

- unsupervised pre-training `wav2vec` speech
- Whisper -- big architectures, tricks

----

# Feedback on the presentation

- dont point, use the presentation w/ pointer
- identify the most difficult slides, ensure everybody understand
