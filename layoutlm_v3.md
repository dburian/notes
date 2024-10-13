---
tags: [transformers,ml]
---
# LayoutLMv3

Third version of LayoutLM introduced by [Huang et
al.](https://arxiv.org/abs/2204.08387). Compared to [the second
version](./layoutlm_v2.md), v3 simplifies the model architecture to a single
Transformer avoiding the need of CNN-based visual encoder.

## Architecture

### Embeddings

The Transformer receives text and image tokens. Text tokens are those obtained
by OCR accompanied by **1D positional embeddings and 2D layout embeddings**,
where the layout embeddings refer to **the bounding box of a text segment**, where
the token was found. In other words, tokens in the same segment have the same 2D
layout embeddings.

Image tokens are flattened-out parts of resized image processed by a linear
layer. This comes from Visual Transformer (ViT) and (ViLT). (TODO)

### Self-attention

V3 uses the same [spatial-aware self-attention as did
v2](./layoutlm_v2.md#self-attention).

## Pretraining

The model is pretrained using 3 losses: MLM, MIM, WPA.

Adjusted Masked Language Modelling (MLM) loss masks out the semantic parts of
**text** embedding (1D and 2D embeddings are left unmasked). The goal is to predict
masked text tokens.

Masked Image Modelling (MIM) is a mirror image of MLM but for image. The loss
masks out some image tokens and trains the model to predict lower-dimensional
representation of the masked out token. (TODO: connection to DALLE through the
[mentioned
paper](https://proceedings.mlr.press/v139/ramesh21a.html?ref=journey-matters))

Word-Patch Alignment (WPA) loss forces the model to align the two modalities.
The model's goal is to predict for unmasked text tokens if their corresponding
image patch is masked by MIM.

