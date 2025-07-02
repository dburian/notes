---
tags: [ml, text_to_speech]
---
[pitch_extract_paper]: https://www.researchgate.net/profile/Paul-Boersma-2/publication/2326829_Accurate_Short-Term_Analysis_Of_The_Fundamental_Frequency_And_The_Harmonics-To-Noise_Ratio_Of_A_Sampled_Sound/links/0fcfd511668e1a0fb2000000/Accurate-Short-Term-Analysis-Of-The-Fundamental-Frequency-And-The-Harmonics-To-Noise-Ratio-Of-A-Sampled-Sound.pdf

# Fastpitch

Parallel TTS model introduced by [Łańcucki
(2020)](https://arxiv.org/pdf/2006.06873). Similar and slightly simpler than
[Fastspeech](./fastspeech.md), Fastpitch generates [mel
spectrograms](./spectrogram.md) from input
text. The spectrograms can be then synthesized to waveform, the authors used
[WaveGlow](./waveglow.md).

## Architecture

The model is composed of two main [Transformer](./transformer.md) blocks with
*pitch predictor* and *duration predictor* in between.

The input characters $\mathbf{x} \in \mathbb{R}^{n}$ are embedded and
contextualized with the first transformer block $\mathbf{h} \in \mathbb{R}^{n
\times d}$. From the hidden representations of characters, the pitch and duration
predictor predict pitch $\mathbf{\hat{p}}$ and duration $\mathbf{\hat{d}}$ for
each characters. Pitches are then projected to the hidden representation's
dimension $\mathbf{h}_{p}$ and added with them $\mathbf{g} = \mathbf{h} +
\mathbf{h}_{p}$. These sums are repeated according to durations. These
repetitions are contextualized again to predict mel spectrogram frames
$\mathbf{\hat{y}}$.

![FastPitch architecture. Figure 1. in
https://arxiv.org/pdf/2006.06873](./imgs/fastpitch_architecture.png)

### Transformer layers

Transformer layers were taken over from [FastSpeech](./fastspeech.md):

- Self-attention block
    - Multi-head [self-attention](./transformer_self_attention.md)
- Add w/ residual
- [Layer Norm](./layer_normalization.md)
- FFN block
    - 1D 3k [Convolution](./convolution_in_ml.md)
    - ReLU
    - 1D 3k [Convolution](./convolution_in_ml.md)
- Add w/ residual
- [Layer Norm](./layer_normalization.md)

Surprisingly in a comparative study, the authors found that **the smaller the
number of attention heads, the better**. So, the end model has only one
attention head and 6 layers.

### Pitch and duration predictors

Both the pitch and duration predictors are:

- 1D Conv
- 1D Conv
- Fully Connected

The pitches are then projected to the hidden representation's dimension with 1D
Conv.

## Training

There are 3 losses:
- mel spectrogram loss
- pitch loss
- duration loss

All are trained with MSE:

$$
\mathcal{L} =
  ||\mathbf{y} - \mathbf{\hat{y}}||^2_2 +
  ||\mathbf{p} - \mathbf{\hat{p}}||^2_2 +
  ||\mathbf{d} - \mathbf{\hat{d}}||^2_2
$$

### Obtaining true pitches

To obtain true pitches authors used a method based on autocorrelation of signal
described in [separate paper][pitch_extract_paper]. The idea of it is:

1. Apply windows to obtain signals per each mel spectrogram frame.
2. On each windowed signal $x$ apply autocorrelation function:

$$
\DeclareMathOperator{\r}{r}
\DeclareMathOperator{\x}{x}
\r(\tau) = \int \x(t) \x(t + \tau) dt
$$

It shouldn't be hard to see that $\r(\tau)$ corresponds to strength of a signal
with period $1/\tau$ contained in $x$. We can also look at normalized
autocorrelation:

$$
\r^\prime(\tau) = \frac{\r(\tau)}{\r(0)}
$$

which gives us relative strength of the frequency $1/\tau$.

3. From the local maxima of normalized autocorrelation derive a list of possible
   candidate frequencies.
4. With [Viterbi algorithm](./viterbi_algorithm.md) find the most probable path
   through frames' all candidate frequencies.

The above mentioned paper includes some details, mainly:
- applying a window is not as straightforward
- problems with sampling of the signal -- signals are discrete, not continuous
- exact definition of 'probabilities' (expressed as costs) that define how 'bad'
  is it to jump from one frequency to another.

### Obtaining true durations

The true durations were extracted from the attention weights of [Tacotron
2](./tacotron_2.md).


## Experiments

The authors trained on *graphemes* (characters), not *phonemes* as training w/
phonemes didn't show any improvement.

The author also used [LAMB optimizer](./lamb_optimizer.md), instead of
[Adam](./adam.md)-related.

The experiments showed that Fastpitch + [WaveGlow](./waveglow.md) is slightly
better in terms of [Mean Opinion Score](./beginners_guide_to_tts.md) than
[Tacotron 2](./tacotron_2.md) + WaveGlow. Both in the single speaker and in
multi-speaker scenarios. Additionally, the authors show that Fastpitch produces
mel spectrograms significantly faster.

### Multiple speakers extension

The authors also showed how Fastpitch can be extended for multi-speaker
scenario, although only 2 speakers were trained. For each input embedding an
additional trainable embedding $s$ was added representing the speaker.
