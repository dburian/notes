# Contrastive loss

Contrastive loss was introduced by [Hadsell, Chopra, LeCun
(2006)](http://yann.lecun.com/exdb/publis/pdf/hadsell-chopra-lecun-06.pdf)

## Context

Contrastive loss was the first loss to train [Metric
learning](./metric_learning.md). As such it is very simple, but for supervised
training it is surpassed. Negatives of contrastive loss:

- converges slowly since it doesn't use the full batch information, unlike
  N-pair loss
- 

## Formula

Contrastive loss penalizes model for distant, but similar pairs, or for close,
distinct pairs:

$$
L(x_1, x_2) = y D(x_1, x_2) - (1 - y)max(0, m - D(x_1, x_2))
$$

where
- $x_1$, $x_2$ are training samples
- $y$ is similarity (1 = similar, 0 = distinct)
- $D$ is distance metric (in the paper it was L2)
- $m$ is margin

The margin $m$ ensures that very distant and distinct points won't be adjusted.
The motivation behind this is that the priority is having similar points close.
Messing with very distant points, may get in the way and so it is ignored.


## Training data

The authors propose quite simple generation of training data: for each sample
generate all positive pairs (neighbors up to a constant similarity) and all
negative pairs. Then sample uniformly.
