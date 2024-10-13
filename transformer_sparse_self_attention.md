---
tags: [ml,transformers]
---
# Transformer's sparse self-attention

Sparse attentions are [self-attentions](./transformer_self_attention.md) that
don't compute some dot-products in the attention matrix and thus save on
resources while sacrificing performance. The goal is to lower the amount of
required memory, which is especially important when processing longer
sequences.

There are several versions of sparse attentions, though usually *the* cited
paper is of [Sparse transformer](./sparse_transformer.md) as it was one of the
first if not *the* first model to implement sparse attention. Other models
include [Longformer](./longformer.md), [BigBird](./bigbird.md) and many others.
There is a nice paper that summarizes efficient transformer models (not only
with sparse attentions) written by [Tay et al.
(2022)](https://dl.acm.org/doi/pdf/10.1145/3530811).

## Mechanisms

Sparse attentions usually have several patterns:

- *local pattern* -- it has been highlighted by many (e.g. [by Clark et al.
  (2019)](https://arxiv.org/abs/1906.04341)) that the classical full
  self-attention looks primarly on neighbouring tokens. To reflect this, sparse
  attentions have a pattern where query token attends to few (e.g. up to 512)
  neighbouring tokens.
- *field-of-view incresing pattern* -- As sparse attentions limit the amount of
  dot-products in which tokens can "exchange" informations, it is critical to
  not limit them too much. Distant tokens could have great dificulty exchanging
  information wich would significantly lower the model's performance. To
  mitigate this problem, models usually have some patterns that increase the
  "field-of-view" of tokens. Such as [Longformer's dilated
  window](./longformer.md) or [BigBird's random pattern](./bigbird.md).
- *global pattern* -- To have at least some tokens that "know the whole
  sequence", some tokens are selected which attent to all other tokens and vice
  versa.

## The theory of implementation

Usually the implementation of sparse attention (if a custom GPU-kernel is not
used) is based on blockifying both the query and key matricies and computing
full attention on given blocks (which can be overlapping). A nice visualization
is found in the appendix of [BigBird's](./bigbird.md) paper.

## Libraries

Recently PyTorch's [`scaled_dot_product_attention`
function](https://pytorch.org/docs/2.2/generated/torch.nn.functional.scaled_dot_product_attention.html)
automatically uses the best kernel there is, including [Flash
Attention](./flash_attention.md). For more selection of self-attentions look at
[xFormers](https://github.com/facebookresearch/xformers) library.

There might be some problems with masking out sequences in a single batch.
Nested tensors might be the way, but in time everything should smooth out. Check
the latest versions.
