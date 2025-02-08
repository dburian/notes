---
tags: [ml,generative]
---

# Normalizing Flow networks

*Normalizing Flow* networks (or *NF*s) are type of Probabilistic Generative
Models (PGM). Compared to other PGMs flow network have intriguing advantages:

- easy to sample from
- easy to evaluate likelihood of a sample (unlike
  [GANs](./generative_adversial_networks.md),
  [VAEs](./variation_autoencoders.md))
- are highly expressive
- produce a latent for **every** sample (not just the generated ones such as [VAEs](./variation_autoencoders.md))
- are easy to train (unlike [GANs](./generative_adversial_networks.md))

## Concept of a flow network

Here we explain the concept of FNs, try to ignore the practicalities as much as
possible, they'll be addressed later.

Normalizing Flow networks, similarly to [diffusion
models](./diffusion_probabilistic_model.md), play with the idea of mapping from data
distribution $p_X$ to some known distribution $p_Z$. Whereas the data
distribution $p_X$ can be arbitrarily complex, the known distribution $p_Z$ is
simple to evaluate and sample from.

Normalizing Flow model $f$ maps $p_X$ into $p_Z$, which, thanks to the [change
of variables](./change_of_variables.md) results in following formula:

$$
p_X(x) = p_Z(f(x)) \|\det{Df(x)}\|\text{,}
$$

where $Df(x)$ is the Jacobian of $f$ at $x$. Normalizing Flows thus *normalize*
the complex distribution into a simple one (often to the simplest
distribution, i.e. $\mathcal{N}(0, \mathbf{I})$).

### Sampling

Major feature of NFs is that they are **invertible**, allowing generation of new
samples:

$$
x = f^{-1}(z)\text{, where}\quad z \sim p_Z
$$

### Composing layers

To build complexity NFs use layers ($f_1, \ldots, f_h$ s.t. $f(x) = f_h(\ldots f_1(x))$) that gradually turn $p_X$ into $p_Z$. Since

$$
\det{\prod_{i = 1}^{h} Df_i(x)} = \prod_{i = 1}^h \det{Df_i(x)}\text{,}
$$

we can write:

$$
\begin{aligned}
p_X(x) &= p_Z(f(x)) \|\det{Df(x)}\| = p_Z(f_1 \circle \ldots \circle 
\end{aligned}
$$


More math

## asdfs

Flow networks are neural networks for generation of a multi-variate
distribution. The advantages of flow networks compared to other generational
networks such as [VAE](./variation_autoencoders.md) or
[GAN](./generative_adversial_networks.md) are:

1. Flow networks give exact mapping to latent space. To elaborate, each
   sample from the modelled distribution will have deterministic value of all
   latent variables, that represents it. The latents will always reproduce the
   original sample. Compared to VAEs, which map a sample to a **distribution**
   in latent space, not a one latent. GANs do not map samples to latent space at
   all.
2. Flow networks allow to compute exact log-likelihood of a sample. VAEs compute
   log-likelihood's upper bound. This allows to compute and compare likelihoods
   of different samples.
3. ... there are more, but I don't understand them yet, more in
   [Glow](./glow.md)'s paper.

Flow networks were introduced gradually with the following papers:

1. [Dinh et al. (2015)](https://www.gwylab.com/pdf/nice.pdf) -- first
   introduction to flow-based networks, shows they work practically
2. [Dinh et al. (2016)](https://arxiv.org/pdf/1605.08803) -- 


## The idea (math here)

The principle of flow networks is based on the same idea as
[VAEs](./variation_autoencoders.md):

> Choose a simple latent distribution and learn a function mapping it to the
> distribution of data.

The trick of flow networks is that they are **invertible**. This means that once
we learn mapping in one direction we can use it in the opposite one as well. Due
to their invertibility, flow networks actually model mapping **from the data, to
the latent distribution**.

Let's have a latent distribution $h \sim p_H$ (usually $\mathcal{N}(0, I)$).
Then we learn mapping $h := f(x)$, where $x$ comes from the data distribution:
$x \sim p_X$. $f$ will be some neural network composed of several layers:

$$
f(\mathbf{x}) =
(f_{n-1} \circ \ldots \circ f_0)(\mathbf{x}) =
f_{n-1}(\ldots f_0(\mathbf{x})\ldots)
$$

### Loss function

We could look at how would the PDF of $x$, how our model $p_\theta$ models it.
We do so by substituting $h:= f(x)$ in $p_H(h)$. Due to the [change of variable
rule](./change_of_variables_pdf.md) we end up with:

$$
\begin{aligned}
p_X(x) &= p_H(f(x)) \left|\det \frac{\partial f(x)}{\partial x}\right| \\
&= p_H(f_{n-1}(\ldots f_0(x) \ldots)) \left|\det\left(
  \frac{
    \partial f_{n-1}(\ldots f_0(x)\ldots)
  }{
    \partial f_{n-2}(\ldots f_0(x)\ldots)
  }
  \right)\right| \\
&= p_H(f(x)) \prod_{i = 1}^{n - 1} \left|\det\frac{
  \partial f_i(\ldots f_0(x) \ldots)
  }{
  \partial f_{i - 1}(\ldots f_0(x) \ldots)
  }
  \right|
\end{aligned}
$$

If we label $f^x_d := f_d(f_{d-1}(\ldots f_0(x) \ldots ))$ then the
log-likelihood of the data can be expressed like so:

$$
\log p_X(x) = \log p_H(f^x_{n-1}) + \sum_{i = 1}^{n-1}
  \left|\det \frac{\partial f^x_i}{\partial f^x_{i-1}}\right|
$$

This gives flow networks exact loss to minimize. The only potentionally scary
thing here is the Jacobian $\partial f^x_d/\partial f^x_{d-1}$.

### Sampling

Since $f$ is invertible, we can simply sample $h \sim p_H$ and compute its
inverse: $f^{-1}(h)$. This means that, in contrast to VAEs, flow networks create
an exact mapping between the latent space and the data space.

### Summary

So we know how to compute the loss, and therefore how to train flow networks. We
also know how to sample. Now we have to construct $f_i$ such that:

- its Jacobian is easy to compute 
- is invertible

## Layers

TODO: split layers into the papers they were introduced in.
TODO: add multiscale architecture of RealNVP
TODO: add 1x1 invertible convolution of Glow

### Coupling layer

The simplest layer is the *coupling layer*. Coupling layer splits the input into
two parts and maps each part differently. Let $x \in \mathbb{R}^n$, then we
split $x$ to: $x_1 = x_{0:j}$ and $x_2 = x_{j+1:n}$ and compute:

$$
\begin{aligned}
y_1 &= x_1 \\
y_2 &= g(x_2, m(x_1))
\end{aligned}
$$

where $m$ is some neural network mapping, and $g$ is a invertible mapping with
respect to its first argument given its second. Most often $g$ is just an
**addition** (so $y_2 = x_2 + m(x_1)$), which is something we'll go with from
now on. The beauty of this mapping is that its **invertible**

$$
\begin{aligned}
x_1 &= y_1 \\
x_2 &= y_2 - m(x_1) = y_2 - m(y_1)
\end{aligned}
$$

and has **unit determinant of the Jacobian** (thanks to the Jacobian being
lower-triangular matrix with unit diagonal):

$$
\frac{\partial y}{\partial x} = \begin{pmatrix}
I & 0 \\
\frac{\partial m(x_1)}{\partial x_1} & \frac{\partial y_2}{\partial x_2} = I \\
\end{pmatrix}
$$

#### Combining coupling layers

Coupling layers only change what is in the second part of the dimensions. To be
able to change also the first dimensions we have to stack multiple coupling
layers on top of each other, flipping the 'parts' of the dimensions.

The question is then: *How many layers?* If we label the overall inputs as $x_1$
and $x_2$ then after:

1. Layer 1: $x_1$ gets unconditionally transformed
2. Layer 2: $x_2$ gets transformed based on $x_2$ and $x_1$
3. Layer 3: both $x_1$ and $x_2$ get transformed based on $x_2$ and $x_1$

So we need 3 layers so that each output dimension can be conditioned on each
input dimension.

### Scaling layer

Since the coupling layer has unit determinant of the Jacobian, it doesn't scale
the input as a whole. To allow for scaling we can multiply each dimension by
learnable parameter. This is both reversible (just divide) and has easily
computable Jacobian: its just a diagonal matrix, with the learnable parameters.

---

#### Extra materials

- [Introductory YT video](https://youtu.be/8XufsgG066A?si=xs4XiwVaLmJlURS3)

