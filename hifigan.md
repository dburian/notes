---
tags: [ml, text_to_speech]
---
# HiFi-GAN

HiFi-GAN is a [GAN](./generative_adversial_networks.md) vocoder introduced by
[Kong et al.
(2020)](https://arxiv.org/pdf/2010.05646).

## Architecture

As a GAN, HiFi-GAN is composed of 2 models: generator and discriminator.
Generator generates waveforms from [mel spectrograms](./spectrogram.md) and
discriminator classifies waveforms as true (coming from training data) or fake
(coming from the generator).

### Generator

The main task of the generator is to upsample mel-spectrogram, which is
understood as **1d input with several channels**, to match the resolution of the
waveform.

There are two 'manipulating' layers that are needed so the number of channels
checks out. First a Convolution layer prepends the entire generator to increase
the number of channels to the desired initial hidden size. Second, another
Convolution layer is added after the last upsampling layer to bring the number
of channels to 1. This means that re-training the HiFi-GAN for different number
of mel-channels could be as simple as training a single layer.

#### Upsampling layer

The generator is composed of sequentially repeated upsampling layers. Each
upsampling layer is a [transposed convolution](./transposed_convolution.md) with
a *Multi-Receptive Field Fusion* (*MRF*) block. The transposed convolution
halves the number of channels while increasing the length of the input by
$k/2$-times, where $k$ is the size of the kernel.

#### MRF block

The MRF block has several parallel *ResBlocks*, whose output is then summed as
the output of the MRF block.

#### ResBlock

ResBlock is a sequential CNN composed repeating the following 2 layers:

- Leaky ReLU
- Dilated 1d Convolution

with a residual connections going around several of such repetitions. Each
convolution in each ResBlock can have different dilatation. But all convolutions
inside one ResBlock have the same kernel size. ResBlocks keep the dimension of
the input, regardless of the dilatation.

#### As a whole

The generator then gradually upsamples the mel spectrogram, each time processing
it through several CNN networks each with different kernel. Therefore each CNN
network has different notion of locality. However, all of them have increased
receptive field through repeated dilated convolutions.

#### Typical hyperparameters

In practice the biggest model had:

- the initial number of channels was 512
- upscaling layers have kernels:
    - 16, (8x upsample)
    - 16, (8x upsample)
    - 4, (2x upsample)
    - 4, (2x upsample)
    - in total this means 256x upsampling, which corresponds to having 256
      tokens
    - meaning the hop size of the generated mel-spectrogram must be 256
- 3 ResBlocks all with the same dilatations, but different kernel sizes
    - kernel sizes were 3, 7, 11
- 6 repetitions of activation + Convolution inside each ResBlock
    - two non-dilatating convolutions to begin with
    - two dilatating convolutions
    - after each dilatating convolution, one normal non-dilatating convolution

### Discriminator

The discriminator is in fact multiple independent discriminators. There are two
types:

- *Multi-Period Discriminator* (*MPD*) -- assesses periodic timestamps
- *Multi-Scale Discriminator* (*MSD*) -- asseses the signal as a continual
  stream of timestamps

There are multiple instances of each type with different setting and each
instance is on its own i.e. its *not an ensamble*.

#### MPD

The idea of MPD is to analyze waveforms in terms of sinusoids -- periodically
repeating signals. So, each MPD gets the waveform that has been reshaped to 2D:

- columns are periods
- rows are different phases.

Applying this reshape to a signal of length $T$ with period 5 gets as a $5
\times T/5$ matrix., where the original indices of samples in the first row
would be $0, 4, 8, \ldots$, and the second row $1, 5, 9, \ldots$, and so on.

Each MPD has different period. Then the 2d matrix is processed by CNN network of
strided convolutions (doubling \# channels each time) with leaky ReLU
activations. The twist is that the convolutions have 1d kernel, forcing them to
**process each phase independently**. The output is projected into single
scalar.

The authors apply [weight normalization](./weight_normalization.md) to all MPDs.

#### MSD

Multi-Scale Discriminators are taken from from
[MelGAN](https://proceedings.neurips.cc/paper/2019/file/6804c9bca0a615bdb9374d00a9fcba59-Paper.pdf).
MSDs are necessary so that discriminators can see the continuous stream of
samples. There are three instances of MSDs gettings inputs pooled 3 different
ways:

- no pooling
- 2x average pooling
- 4x average pooling

MSDs are CNNs composed of strided and [grouped convolutions](./convolution.md)
with leaky ReLU activations. All except the first MSDs are weight normalized.
The first MSD is *spectral normalized* -- a normalization developed to keep
GAN's training stable introduced by [Miyato et al.
(2018)](https://arxiv.org/pdf/1802.05957). There is a lot of math, but at a
first glance it seems that the goal is to have maximum singluar value of the
weight matrix equal to 1.

## Training

The generator and all the discriminators are trained using several losses:

- adversary loss $\mathcal{L}_{Adv}$ -- discriminators try to beat generator and
  the other way around
- mel-spectrum loss $\mathcal{L}_{Mel}$ -- forces generator to return
  mel-spectrogram-equivalent waveform
- feature matching loss $\mathcal{L}_{FM}$ -- additional signal to generator

I define the losses for one discriminator only, the total loss, shown at the
end, sums the respective losses for all discriminators.

### Adversary loss

The adversary loss is the standard way how GANs are trained. To avoid [vanishing
gradients](./generative_adversial_networks.md), the authors formulate the loss
with MSE:

$$
\begin{align}
\mathcal{L}_{Adv}(D; G) &= \mathbb{E}_{(x,s)}\left[(D(x) - 1)^2 + (D(G(s)))^2\right] \\
\mathcal{L}_{Adv}(G; D) &= \mathbb{E}_{s}\left[(D(G(s)) - 1)^2\right]
\end{align}
$$

where $x$ is the ground truth audio, $s$ is the mel-spectrogram of $x$.

### Mel-spectrum loss

At the begining, it helps to stabilize the training of GANs with additional goal
like: *mel spectrograms of the outputs should be equal to the input*. This is
called a *reconstruction loss* and was shown to benefit also the quality of the
generated solutions.

$$
\mathcal{L}_{Mel}(G) = \mathbb{E}_{(x, s)} \big[ \|\phi(x) - \phi(G(x))\|_1 \big]
$$

where $\phi(x)$ is function that transforms a waveform into a mel-spectrogram.
As I mention below, the authors use fairly large weight, suggesting
**mel-spectrum loss is important for convergence**.

### Feature matching loss

Feature matching loss is an additional loss that gives the generator guidence.
It was shown by previous works to contribute to the successful training of
speech synthesis model, so the authors use it as well.

$$
\mathcal{L}_{FM}(G; D) = \mathbb{E}_{(x,s)} \left[
  \sum_{i=1}^T\frac{1}{N_i}\|D^i(x) - D^i(G(s))\|_1
\right]
$$

where $T$ is the number of discriminator layers, $D^i$ and $N_i$ stand for the
features and the number of them in the $i$-th layer.


### Total loss

For all losses, and all discriminators $D_i$ the total loss equals to:

$$
\begin{align}
\mathcal{L}_G &= \sum_{k = 1}^K \big[
  \mathcal{L}_{Adv}(G;D_k) + \lambda_{fm} \mathcal{L}_{FM}(G; D_k)
  \big] + \lambda_{mel} \mathcal{L}_{Mel} \\
\mathcal{L}_D &= \sum_{k = 1}^K \mathcal{L}_{Adv}(D_k;G)
\end{align}
$$

The authors used the following weights:
- $\lambda_{fm} = 2$
- $\lambda_{mel} = 45$

## Experiments

All scores are given as [MOS scores](./beginners_guide_to_tts.md). In all
experiments samples generated with HiFiGAN are just below the ground truth in
terms of MOS.

### Mel-spectrogram synthesis evaluation on unseen speekers

Experiments shown Hifi-GAN is superior to MelGAN, [WaveGlow](./waveglow.md) and
[WaveNet](./wavenet.md) when evaluating on unseen speekers. The evaluation was
done by generating the audio samples to mel-spectrograms and judging the
synthesized waveforms.

### End-to-end evaluation with Tacotron 2

The authors also compared HifiGAN in an end-to-end training. For mel-spectrogram
generation they used [Tacotron 2](./tacotron_2.md) and showed that in this
setting HiFiGAN is superior to [WaveGlow](./waveglow.md).

## Ablation studies

The authors did two ablation studies. First the authors showed the model can be
made slimmer (and bit worser in terms of results) by:

- decreasing the initial number of channels of generator (e.g. to 128)
- less upscaling layers with more dramatic upscaling (4x instead of two 2x)
- less layers in each ResBlocks
- smaller kernels for ResBlocks
- to save on parameters while maximizing receptive field, the ResBlocks with
  larger kernels got more dilated convolutions

The second ablation study is on all the major decisions. The resulting MOS
scores judge **the quality of the generated audio** and indicate
the imporance of various parts of the model. Going from the most important to
the least:

1. MPD
1. Mel loss
1. MSD
1. minimum overlap in MPD's different periods
1. having multiple ResBlocks with different kerenels helped




