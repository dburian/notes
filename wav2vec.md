# Wav2vec

Wav2vec is a pre-trained speech recognition model introduced by [Baevski et al.
(2020)](https://arxiv.org/pdf/2006.11477).

Wav2vec is composed of several parts. CNN acts as a feature encoder and encodes
the audio signal to fixed-sized representations which are fed to Transformer
that contextualizes it. Similarly to MLM the representations are masked-out,
leaving the Transformer to predict their *quantized forms*.

The model is then fine-tuned on a labelled dataset using [CTC loss](./ctc.md).
The authors showed that thanks to the pre-training, even with small amount of
supervised data the model is able to surpass the competition

TODO:
- quantization
- CNN structure
- Gumbel Softmax
- Using CNN instead of absolute embeddings for the transformer
- Contrastive loss
- diversity loss for the masked prediction
- usage of SpecAugment during fine-tuning
