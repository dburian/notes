# HuBERT

Hidden-unit BERT (HuBERT) is a pre-trained speech recognition model introduced
by [Hsu et al. (2021)](https://arxiv.org/pdf/2106.07447). HuBERT closely
resembles [wav2vec](./wav2vec.md) in that it pre-trains using self-supervised
learning by classifying quantized input features.

## Architecture

HuBERT is a model composed of CNN feature encoder, Transformer that
contextualizes features and a quantization model. The authors use k-means
clustering that assigns each span of speech a cluster. To make the cluster
labels more robust, there are ensembles of $k$ clusterings with different
parameters for each span. The loss takes multi-label form:
$$
L_m(f; X, \{Z^{(k)}\}_k, M) =
\sum_{t \in M} \sum_k \log p_f^{(k)}\left(z_t^{(k)} \mid \tilde{X}, t\right)
$$

In later stages of training the clusters are derived from the trained
representations of the transformer, rather than from the raw input. Using
clustering metrics, the authors show that clustering based on Transformer
features yields better clusters.

## Interesting comparisons to wav2vec

The authors point out that they quantize the input directly, not quantizing the
features extracted by CNN. They argue that since CNN encoding is lossy,
quantization has less information to quantize the input and therefore is not as
good.

## Ablation studies

The authors experiment with convex combination of losses on masked and unmasked
input spans. They show that with training only on masked inputs, the model is
more resilient to bad clustering. However, with good clustering, training on
unmasked spans is favorable.

TODO:
- *freeze step* parameter during fine-tuning (as in wav2vec)
- What exactly is MFCC
- What are dev-other sets? Is there some other dev set? dev-clean, dev-other,
  test-clean, test-other


