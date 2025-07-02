---
tags: [ml, skimmed_over]
---

# StyleGAN 3

StyleGAN3 is a [GAN](./generative_adversial_networks.md) modelling images. The
model was introduced in [Alias-Free Generative Adversial
Networks by Karras et al.
(2021)](https://proceedings.neurips.cc/paper/2021/file/076ccd93ad68be51f23707988e934906-Paper.pdf).

The main contribution of the paper is a theoretical view on NN layers,
guaranteeing invariance to translation and rotation and applying these
theoretical results to real-world network.

## Intro

The motivation behind StyleGAN3 is to get rid of 'texture sticking' problem that
appeared in StyleGAN2. Texture sticking is when a texture in an image **stick**
to given coordinates instead of reacting to the moving background. The problem
is nicely illustrated in [supplementary
video](https://nvlabs-fi-cdn.nvidia.com/stylegan3/videos/video_0_ffhq_cinemagraphs.mp4).

### Positional references

The authors claim that current generative models include 'unintentional
positional references' in the intermediate layers that makes the generation of
pixel value dependent on it's position. The model can infer a pixel's position
through:
- image borders
- per-pixel noise inputs
- [aliasing](./aliasing_in_nn.md)

### Aliasing

According to authors, aliasing in generative networks happens due to
- non-ideal upsampling (e.g. strided convolutions)
- point-wise application of non-linearities (e.g. ReLU)

## Main contributions

### Continuous domain

Instead of considering the discrete pixel grids, the authors look at the
continuous signals from which the pixel grids were sampled. They argue that what
they want to do is achieve invariance in continuous domain, not in discrete
domain and support this by the fact that we cannot translate discrete pixel
grids by half a pixel.

I find this quite strange, but again, I could understand the paper better given
some extra time.

#### Continuous and discrete operations can be equivalent

Later the authors show that a discrete operation $F$ can transformed to an equivalent
continuous operation $f$ by applying:
- sampling operation -- turns the continuous signal into discrete one
- $F$ -- does the discrete operation
- interpolation filter -- turns the discrete output into continuous one

The same thing holds vice-versa, if $f$ (the continuous equivalent of $F$)
doesn't introduce any frequencies beyond band limit given by the sampling
operation ($s/2$).

#### What the authors are trying to do in the continuous domain

What is important for us is that we can reason about the continuous domain, find
and invariant operations there, **ensure they do not introduce any frequencies
beyond the frequency limit** and thanks to the continuous and discrete
operations being equivalent, we can safely deduce the actual architecture of the
network's layers.

#### Equivariance

The authors use equivariance instead of invariance. Here is a definition: an
operation is equivariant to spatial transformations (rotations, translations) if
it commutes with it (we don't care if we translate the image and then do the
operation, or the other way around).

### Operations in continuous domain

The authors look into:
- convolutions
- upsampling and downsampling -- increasing and decreasing dimensionality on
  the time-axis
- non-linearities

They find out that:

### Convolutions are ok

- Skimming through the math, convolutions are equivariant to translations, for
  rotations a special kernel might be needed

### Upsampling is ok

- doesn't modify the signal -> equivariant
- shouldn't introduce any new frequencies -- though IMHO after sampling a
  aliasing issues might occur, notice how [BigVGAN](./bigvgan.md) uses low-pass
  filter after upsampling.

### Downsampling is ok as long as we do a low-pass

- we need to remove frequencies that cannot be represented in the downsampled
  signal
- again rotation equivariance needs special kernel

### Non-linearities are **not ok**

- in continuous domain point-wise non-linearities commute with translation or
  rotations, so they **are equivariant**
- however in continuous domain, **non-linearities may introduce new frequencies
  beyond the band limit**
- therefore discrete operation cannot be implemented in the discrete domain
  itself
- the solution is to enter the continuous domain momentarily, do the continuous
  non-linearity, apply low-pass filter and transition back to discrete domain
- **in practice** authors approximate entering continuous domain by $m\times$
  upsampling, followed by point-wise non-linearity, and $m\times$ downsampling
- **in experiments** authors found $m = 2$ to be sufficient, though the results
  mildly increase if higher $m$ is used

---
Sources:
  - [Blog post](https://codoraven.com/blog/ai/stylegan3-clearly-explained/) --
    wonderful explanation of the model with lots of pictures by
