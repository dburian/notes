---
tags: [ml,transformers]
---
# Decoder-only transformer

The vanilla [Transformer](./transformer.md) is composed of two parts: encoder
and decoder. Decoder-only architectures ditch the encoder, leaving only the
decoder part.

## Details

Of course ditching the encoder means also to ditch the encoder-decoder
attention. This leaves us only with masked self-attention.

The decoder has to naturally ingest input, but since there is no encoder, we
essentially train the model to predict the most probable continuation of the
input. This is how GPT style models were trained.

During inference the model can compose the answer from several predictions,
basing the next on its previous one. This is what's called a autoregressive
model.

## Why

Decoder-only models allow to generate sequences with much lower parameter count
than full transformers. Essentially the thinking is that decoder is
"smart-enough" to contextualize tokens at its input and create their
representations without any [encoder](./encoder_only_transformer.md).


