---
tags: [ml, tools]
---

# WhisperX

[WhisperX] is a great tool for generating ASR transcription out of a recording.
It generally works very well, though you cannot expect miracles. **Its strength
lies in the ease of use and clever combination of several models**:
- Voice Activity Detection (VAD) -- cuts recording into short segments
- [Whisper](./whisper.md) -- generates ASR transcript for each segment
- [Wav2vec2](./wav2vec.md) -- aligns the transcript with the recording
- Diarization model -- recognizes speaker changes between segments


Though it is just a single repo, it combines all models very well and could be a
great starting point for a more specialized pipeline.

Additionally, WhisperX has a bunch of writers that format the result. `SRTWrite`
outputs the aligned ASR transcription using subtitles.
