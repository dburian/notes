---
tags: [ml, skimmed_over]
---

# Poor generalizations of convolutional networks

This note summarizes main idea of ["Why do deep convolutional networks
generalize so poorly to small image transformations?" by Azulay and Weiss
(2019)](https://arxiv.org/pdf/1805.12177).

The authors measure and identify why convolutional networks are not invariant
to small transformations of input images despite:
- convolutions' inductive bias -- [Convolutions](./convolution_in_ml.md) were
  designed to be invariant to pixel positions (i.e. to translations of features
  in input image)
- being trained on images with small augmentations that include translations and
  rotations

The authors claim that the lack of invariance is caused by convolutions ignoring
the [sampling theorem](./nyquist_shannon_sampling_sampling_theorem.md). To be
precise, the authors point out that the classical subsampling is not ideal in
convolutions, where it's approximated by stride. Thus we cannot expect a network
to have the same outputs, unless we shift the input image by a factor of the
network's total subsampling factor=product of all of it's strides.
