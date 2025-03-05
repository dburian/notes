---
tags: [information_theory, ml, generative]
---
# Lossless codelengths

In the context of generative models, lossless codelength specified as bits per
dimension (or *bpd*) is a measure how many [bits of
information](./self_information.md) describe model's output distribution.

It is defined as negative log-likelihood computed with logarithm of base $2$ or:

$$
\mathbb{E}_p - \frac{log p(x)}{log 2}
$$

I couldn't find much information about the metric itself, so I can only guess
why it is named as it is:

- lossless: since the bits fully describe the distribution, we don't lose any
  information
- codelengths: here I think it is assumed it is a **binary** code that describes the
  distribution.
