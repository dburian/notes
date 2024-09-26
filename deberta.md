# DeBERTa

DeBERTa -- Decoding-Enhanced BERT with disentangled Attention -- is a model from
Microsoft introduced by [He et al. (2021)](https://arxiv.org/pdf/2006.03654).
The paper introduces three novelties:

- adding absolute position encodings to the *output* of a BERT-like transformer
  to help with MLM prediction
- disentangling computation of embedding and position attention scores
- adversarial training method (not explored)

The paper reports significant performance bumps over similarly sized models
despite training on half of the data. The mentioned benchmarks are MNLI, SQuAD,
RACE and SuperGLUE.

## Disentangled attention computation


