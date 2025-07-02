---
tags: [ml,generative,text_to_speech]
---

# BigVGAN

[Vocoder](./beginners_guide_to_tts.md) introduced by [Lee et al.
(2023)](https://arxiv.org/pdf/2206.04658).

BigVGAN is a scaled-up version of [HifiGAN](./hifigan.md) with further tweaks
that help the model to generalize better to out-of-domain (*OOD*) data. Big
emphasis on the paper is to produce a large vocoder that should do well in a
one-shot settings (i.e. OOD data, no finetuning). The contributions of the paper
are:

1. using [Snake activation](./snake_activation.md) to give a model an [inductive
   bias](./inductive_bias.md)
2. upsampling, then downsampling mels to avoid aliasing issues
3. Scaling of large GAN-based vocoder and the related observations
4. Training a SOTA vocoder
  - SOTA results on OOD test datasets

## Architecture

Even though likelihood-based models (diffusions, autoregressive, or flow-based)
are considered more stable durint training, the authors choose
[GAN-based](./generative_adversial_networks.md) vocoder because:

- autoregressive or [diffusion models](./diffusion_probabilistic_model.md)
  require more interations to sample

- [flow-based models](./normalizing_flow_networks.md) enforce data distr. shape
  due to enforcing bijection through affine layers, may limit the model's output

### Generator

Generator is the same as HifiGAN's generator, but with MRF swapped for
*anti-aliased multi-periodicity composition* (*AMP*) module.

![BigVGAN architecture. Figure 1 in
https://arxiv.org/pdf/2206.04658](./imgs/bigvgan_architecture.png)

Generator is a convolutional neural network composed upsampling blocks:
- a [transposed convolution](./transposed_convolution.md) and
- several residual *Anti-aliased multi-periodicity Composition* (or *AMP*)
  blocks each with different kernel size

*Residual* means that the blocks get the same input, and their output is summed.

### AMP block

AMP block is composed of several residual blocks with different dilatations:
- 2x upsampling w/ low-pass filter
- [Snake activation](./snake_activation.md)
- 2x downsampling w/ low-pass filter
- dilated convolution with given dilatation
- 2x upsampling w/ low-pass filter
- [Snake activation](./snake_activation.md)
- 2x downsampling w/ low-pass filter
- convolution with dilatation=1

#### Snake activations

Snake activation helps when modelling periodic functions. The authors say that
it gives the model *inductive bias for modelling periodic functions*. According
to the authors, typical transformations such as ReLU overfit to training
examples and are not able to generalize (extrapolate).

The Snake activation learn periods ($a$) for each input channel.

#### Up and downsampling

Before and after the snake activations the authors use up- and downsampling
accompanied by a low-pass filter:
1. 2x upsample w/ low-pass filter
2. linear + snake activation
3. 2x downsample w/ low-pass filter

This is similar to a feed-forward layers of
[Transformer](./transformer.md). They too, increase the dimensionality, do a
calculation, then decrease it again. The authors support this decision with:

1. [Nyquist-Shannon theorem](./nyquist_shannon_sampling_sampling_theorem.md) --
   the theorem states that sampled signal can represent frequencies up to half
   of the sampling rate. So after 2x upsampling, the signal can theoretically
   contain frequency equal to its original sample rate.
2. 'Simulating' continuous domain -- this is a bit of a story tale and a bit of
   a deep dive into [equivariant operations](./stylegan3.md).

#### Low-pass filter with upsampling

You might question why there is a low-pass filter after upsampling. This might
be unintuitive but adding 0s between actual discrete samples creates new
frequencies. This is due to the fact that [Discrete Fourier
Transform](./discrete_fourier_transform.md) creates repeating frequencies that
are $1/s$ apart, where $s$ is sampling frequency. When we increase the
frequency, these repeating frequencies are closer together. So when we do
upsampling in the real world, we use low-pass filter, to add samples, but get
rid of the unwanted frequencies which the additional samples create.

Obviously, BigVGAN is a bit different context, but the fact still holds: adding
samples may create new frequencies, which were not there before the upsampling
happend.

### Generator summed

The corner stone of the generator is the AMP block that learns periodic signal
with different kernel size (could be viewed as different limitation for
bandwidth). AMP blocks operate on all upsampling levels, which gives the
generator the opportunity to add global information (beginning) and detailed
information (at the end of the upsampling).

### Discriminator

The tl;dr is that BigVGAN uses same discriminator as [HifiGAN](./hifigan.md),
but replaces MSD with MRD.

The discriminator is composed of two submodules:
- *Multi-Period Discriminator* (or *MPD*) from [HifiGAN](./hifigan.md)
- *Multi-Resolution Discriminator* (or *MRD*)

#### MRD

MRD was introduced as part of UnivNet by [Jang et al.
(2021)](https://arxiv.org/pdf/2106.07889). The discriminator uses several STFT
parameters, to obtain several spectrograms, which are fed through a CNN netowrk,
whose binary output serves as the dicision whether the input waveform was real
or fake.

#### Training losses in detail

For generator we sum (with some weighting):
- adversarial losses: the discriminators' outputs should be "It's Real!"
- feature matching losses: each discriminator's activations should be same for
  real and fake inputs
- mel reconstruction loss: creating mel out of the final waveform should give us
  the input

For discriminator we sum:
- the adversarial loss: the fake and real inputs should be classified correctly
  by all discriminators

## Training

BigVGAN was trained on LibriTTS:
- 24 kHz
- not only clean but also noisy from all kinds of environments -- crucial for
  being able to generalize
- training with `[0, 12]`kHz window, instead of the standard `[0, 8]`kHz

In training the authors experienced an early collapse of the model. This was
mitigated by:
- alleviated with half of learning rate (compared to HifiGAN) -- `1e-4`
- double batch size (compared to HifiGAN) -- `32`
- gradient clipping

BigVGAN was trained for 1M steps in the paper, but checkpoints exist for 3M and
5M.

## Evaluation

BigVGAN was evaluated on
- 5 different automatic metrics
- CMOS (comparative MOS) -- scores similarity of two recordings on `[0,
  5]` scale
  - MOS isn't appropriate since they we need to compare not only the quality of
    the audio, but also how much it corresponds to the gold audio (i.e. taking
    into an account also the speaker identity and environment

### In domain evaluation

- On LibriTTS `dev` and `test` sets -- contains unseen speakers, but seen
  environments
- BigVGAN won on all metrics
- BigVGAN-base outperformed HifiGAN considerably, but all other models as well
  - WaveGlow-256, WaveFlow-128, SC-WaveRNN
- Difference between BigVGAN (base or no-base) and HifiGAN is more significant
  on SMOS than on MOS -- HifiGAN lacks cloning ability

### OOD evaluation:

Only evaluated using SMOS.

#### Speech

- the speech OOD datasets include:
  - speech from low-resource languages
  - multilingual ted-talks with random environemntal noise and
  - speech in korean with non-simulated noise
- the authors only compared BigVGAN (and BigVGAN-base) to HifiGAN, and UnivNet
- the biggest difference was in audios with noise in them, it seemed like
  simulating unknown speech wasn't a big of an issue for UnivNet or HifiGAN
- the difference between BigVGAN-base and BigVGAN is bigger for simulated noise

#### Noises/Music

The authors also used MUSDB18-HQ to test robustness to noises. MUSDB18-HQ
contins splits with:
- `vocals`,
- `drums`,
- `bass`,
- `others`
- `mixture` -- mixture of the above

Here BigVGAN benefited a lot from increased range which was particularly
noticeable for `vocals`, `others` and `mixture`

## Ablation studies


### Architecture ablation studies

Ablation studies were done also on MUSDB18-HQ mentioned above.

Authors ablated the decisions:
- of bigger model (BigVGAN-base vs BigVGAN)
- inclusion of Snake activation
- inclusion of Snake activation with low-pass filters

Learnings:
- The bigger architecture performed significantly better for `others` and
  `mixture`
- Inclusion of low-pass filter is more important than having both low-pass
  filter and snake activations, nevertheless even snake activations are
  responsible for significant improvements in the `others` and `mixture`
  categories of audio
  - this is the data on which the authors found the claim that snake activations
    give the model inductive bias that makes it generalize better to unseen
    domains

### BigVGAN vs large HifiGAN

Big HifiGan or BigVGAN studies:
- **only 58%** chose generated audio with BigVGAN vs with HifiGan
- this is enough for $\text{p-value} < 0.01$ to prove that scaled-up
  BigVGAN-base is better than scaled-up HifiGan

### Large or clean train data

By comparing evaluation scores of BigVGANs trained on clean subset of LibriTTS
vs all of LibriTTS train data, the authors showed that **BigVGAN favors larger
training dataset, despite not being as clean**.


## Appendixes

Only briefly what you can find in there:

- architecture in detail
- training objectives revisited
- practical lessons -- what the authors tried but didn't pan out
- visualizations -- effects of low-pass filter and snake activations in practice
- additional results -- expanded comparion with other models
- evaluation with ASR
- details how they've outsourced MOS and SMOS
