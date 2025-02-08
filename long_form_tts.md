---
tags: [ml, speech_to_text]
---

# Long form TTS

So far I've found two branches: codecs and Language Models (*LM*s) and latent
diffusion.

## LMs

- [ ] [Vall-e (2023)](https://arxiv.org/pdf/2301.02111)
- [ ] [Bark](https://github.com/suno-ai/bark)
- [ ] [AudioLM paper (2022)](https://arxiv.org/pdf/2209.03143)
- [ ] [Audio LM overview (2024)](https://arxiv.org/pdf/2402.13236)
- [ ] [SoundStorm (2023)](https://arxiv.org/pdf/2305.09636)

## Latent Diffusion

- [ ] [Fast timing-conditioned latent Audio Diffusion
  (2024)](https://arxiv.org/pdf/2402.04825) (Stability AI)
- [ ] [Long-Form music generation with latent diffusion
  (2024)](https://arxiv.org/pdf/2404.10301)
- [ ] [XTTS (2024)](https://arxiv.org/pdf/2406.04904) (Coqui)
- [ ] [Tortoise (2023)](https://arxiv.org/pdf/2305.07243)
- [x] [StyleTTS2](./styletts_2.md), 

## Services similar to ElevenLabs

- [ ] [Apolio](https://github.com/IAHispano/Applio)
- [ ] [ToucanTTS](https://toucantts.com/)

## Long-form audio through post-processing

- [x] [Sentece-joining idea](https://github.com/suno-ai/bark/pull/84#issuecomment-1523433448) and [demo](https://github.com/suno-ai/bark/pull/84#issuecomment-1521063117)
    - the coherency is lost but bark has 'history_prompt': [issue comment](https://github.com/suno-ai/bark/pull/84#issuecomment-1521063117)
    - tortoise doing something [similar?](https://github.com/neonbjb/tortoise-tts/blob/main/tortoise/utils/text.py)
