---
tags: [ml]
---
[paper]: https://proceedings.neurips.cc/paper/2016/file/ed265bc903a5a097f61d3ec064d96d2e-Paper.pdf
# Weight Normalization

Weight normalization is a reparametrization of the weight vectors in neural
networks that speeds up convergence. It was introduced by [Salimans et al.
(2016)][paper].

Weight normalization reparametrizes the typical equation of *neuron* output:

$$
y = \mathbf{x} \cdot \mathbf{w} + b
$$

by expressing $w$ as normalized vector with scale:

$$
y = \frac{\mathbf{x} \cdot \mathbf{v}}{||\mathbf{v}||}g + b
$$

## Idea

The idea is to mimic what [Batch normalization (BN)](./batch_normalization.md)
does: standardizing pre-activation outputs to given variance and mean. Assuming
the inputs already have unit variance and 0 mean, weight normalization does
pretty much the same (proof by waving hands) as using BN on an output of normal
layer output.

## Practice

There is a `torch.nn.utils.weight_norm` function that modifies given Parameter,
replacing its 'weight' by 'weight_v' and 'weight_g' parameters, and recomputing
the original weight before every `forward` with a hook.
