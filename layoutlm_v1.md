---
tags: [transformers,ml]
---
# LayoutLMv1

First version of LayoutLM introduced by [Xu et al.
(2019)](https://arxiv.org/abs/1912.13318). The first version combines only text
and layout information.

### Architecture

LayoutLM's v1 architecture is based on BERT. The only difference to a
traditional text-based BERT is the addition of layout embeddings to each token.
Each token receives a bounding box embedding (left upper corner, and right lower
corner of the token in image). This information is supplied by Optical Character
Recognition (OCR).

Additionally v1's architecture also uses Faster R-CNN model to generate features
for each token's bounding box. These features are combined with the
Transformer's outputs (token embeddings) in downstream tasks.

![LayoutLMv1 architecture. source:
https://arxiv.org/abs/1912.13318](./imgs/layoutlm.png "LayoutLMv1 architecture")

### Pretraining

The model is pretrained using two loses. First Masked Language Modelling (MLM)
loss is applied to the token embeddings, where only the embeddings of the text
is masked out (2D layout embeddings and 1D position embeddings are left
unmasked). Authors call this loss Masked Visual-Language Modeling (MVLM).
According to the authors this should motivate the model to connect the token
information with the bounding box embeddings.

The second loss is Multi-label Document Classification (MDC), which is applied
only if the documents have tags. The loss scores multi-label classification of
the input based on `[CLS]` token embedding. In testing the MVLM+MDC training
was only slightly better than just MVLM training, so MDC is expendable.


