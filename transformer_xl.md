---
tags: [transformers,ml]
---
# Transformer-XL


A model combining RNNs and Transformer architectures to arrive at a efficient
transformer able to process longer inputs introduced by [Dai et al.
(2019](https://arxiv.org/abs/1901.02860).

## Architecture

The architecture is based on a primitive idea:
- split the text into segments
- process the segments individually but cache (detached) hidden states at each
  level
- when computing [self-attention](./transformer_self_attention.md) bring-in the
  cached hidden states from the previous level.

To make the above work the authors had to use different positional embedding
than the standard absolute to differentiate between hidden states from the
current and the previous segment. These positional embeddings are relative
pre-computed (not learned). The authors do a good job of explaining how to plug
the new relative embeddings into the classical self-attention.

The result is that the attention span is $O(NL)$, where $N$ is a context length
of each segment and $L$ is a number of layers. Note that gradients do not
"travel" between segments. All the network can do is use the information, but
not adjust it.
