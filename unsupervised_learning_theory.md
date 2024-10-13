---
tags: [ml]
---
# Theory of unsupervised learning

in progress

found a vid by Ilya Sustskever stating that
- there is no theory saying unsupervised learning will lead to better
  understanding of the inputs
- Ilya connects Kolmogorov complexity, compression and maximizing likelihood of
  next token prediction by waving hands
- showing that unsupervised training makes sense and could be the least regret
  way of learning from an unlaballed dataset

There are two *oh* moments: 

## Compression in context of unsupervised learning

In unsupervised learning we're trying to learn the distribution of the dataset.
That is exactly what we'd expect a good compressor to do. To leverage the
knowledge about the distribution to achieve higher compression ratio.

## Kolmogorov complexity

Kolmogorov for a dataset is the smallest program that outputs given dataset.

Kolmogorov complexity is **undecidable**.
