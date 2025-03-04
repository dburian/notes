---
tags: [ml, text_to_speech]
---

[HifiGAN]: https://proceedings.neurips.cc/paper_files/paper/2020/file/c5d736809766d46260d816d8dbc9eb44-Paper.pdf
[gh_error]: https://github.com/roatienza/efficientspeech/blob/218f62f43cdfe3950961d6d92a1e60c33605c5e1/layers/networks.py#L194-L208

# EfficientSpeech

EfficientSpeech is a small text-to-speech model for edge and mobile devices
introduced by [Atienza (2023)](https://arxiv.org/pdf/2305.13905).

EfficientSpeech is a system composed of:
- [g2p phoneme generator](https://github.com/Kyubyong/g2p)
- the model itself
- [HifiGAN vocoder][HifiGAN]

## Architecture of the model

> I've identified slight error in the image in the paper. According to the image
> the "upsampling" operation done on the output of the first feature encoder
> block, takes in also the upsampled output of the second feature encoder's
> block. I've looked into the source code and this is clearly not the
> case -- the features are 'fused' [independently][gh_error]. So its as the
> paper describes in the paragraphs below the image.

The model is devided into several parts:
1. feature encoder -- processes and contextualizes phonemes
2. acoustic features predictor -- predicts pitch, energy and duration
3. feature fuser and upsampler -- concatenates acoustic features w/
   contextualized phonemes and repeats phonemes according to their duration
4. mel spectrogram predictor -- predicts mel spectrograms from fused mel-frame
   features


### Feature encoder

Feature encoder is composed of two blocks that consist of:
- [Depth-separable convolution](./convolution.md)
- Self-attention layer
- 1D Convolution
- plus some [Layer Norms](./layer_normalization.md) mixed in

Feature encoder follows a [U-network](./u_network.md) style architecture. First
block leaves the sequence dimension alone, but the second block uses convolution
with stride 2, which halves the sequence length. Outputs of each blocks are
further processed by:

- linear layer
- transposed convolution for features of second block, identity for first block

These are then concatenated, and run through a linear layer to produce
contextualized representations of phonemes.

### Acoustic feature predictor

EfficientSpeech follows [FastSpeech 2](./fastspeech2.md) and also predicts
acoustic features including duration per phoneme. Each predictor is trained, and
all three predictions are embedded.

### Feature fuser and upsampler

Feature fuser concatenates the embedded acoustic features with contextualized
phoneme features. Then the phoneme features are turned into mel frame features
by repeating them according to the predicted phonemes' durations.

### Mel spectrogram predictor
To predict the final spectrogram combination of depth-wise separable

convolutions, linear layers, tanh activations and layer norms are used.
