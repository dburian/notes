---
tags: [ml,transformers]
---
# Encoder-only Transformer

The vanilla [Transformer](./transformer.md) is composed of two parts: encoder
and decoder. However, for some applications decoder is unnecessary and is
tossed. What is left is an *encoder* Transformer, sometimes refered to as
encoder-only architecture.

## When it is useful

Encoders are useful when we just want to process the sequence. We sometimes
refer to this processing as *contextualization* of the input sequence, since an
output for a particular token depends on its surrounding tokens (its context).
These usecases include:

- classification of tokens
- classification of whole sequence
- embedding of tokens
- embedding of the whole sequence
- ...
