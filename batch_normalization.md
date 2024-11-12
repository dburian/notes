---
tags: [ml]
---
# Batch Normalization

Batch normalization (*BN*) is a layer in neural networks that standartizes the
pre-activation outputs of previous layer to have given variance and mean. It was
introduced by [Ioffe et al. (2015)](https://arxiv.org/pdf/1502.03167).

## Definition

The layer itself is defined as follows:

$$
o = \frac{x - \mu}{\sigma} * \gamma + \beta
$$

where
- $x$ is the input and $o$ is the output of batch normalization
- $\mu$ and $\sigma$ are expected mean and standard deviation of the inputs
- $\gamma$ and $\beta$ are trainable parameters that scale the normalized input

$\mu$ and $\sigma$ are trained during training using exponential moving
averages, and kept fixed for inference.

## Why it helps

The problem with training deep neural network is that the inputs to the deeper
levels continously change as weights of previous levels are adjusted. This makes
it harder for the deeper layers to be trained as they have to also adapt to the
weight adjustments of the previous layers.

The idea of BN is to make the output of each layer more stable during training,
thereby allowing the deeper levels not only to react to how previous layers
changed, but also to train itself.

BN also regularizes the network since it forces the outputs to have certain
distribution.

Both effects of batch normalization are usually valued.

## Be ware of small batches

The drawback of BN is that it requires large batch sizes to accurately estimate
the mean and the variance of the inputs. With small batch sizes, BN can
significantly hurt the training by bringing in noise.
