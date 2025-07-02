---
tags: [ml]
---

# Inductive bias

Inductive bias is a set of assumptions that we take to be true when training a
model. E.g. when training image-recognition model, we use
[Convolutions](./convolution_in_ml.md) since we assume that information in images
should be processed independently on *where* in the image the information is,
but dependently on its *context* (the surroudning pixels). In this context, the
model would have such inductive bias -- ignore spatial information, but consider
nearby pixel values, when processing given pixel.
