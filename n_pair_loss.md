---
tags: [metric_learning]
---
# N-pair loss

N-pair loss is loss for supervised [metric learning](./metric_learning.md)
introduced by [Sohn
(2016)](https://proceedings.neurips.cc/paper/2016/file/6b180037abbebea991d8b1232f8a8ca9-Paper.pdf).
It is a natural progression of the [Triple loss](./triplet_loss.md), but trying
to extract more information from a given batch.

## Loss formula

Notation:
- $x$ -- anchor input
- $x^{+}$ -- positive input
- $x_i$ -- negative input to $x$
- $f$ -- normalized representation of $x$
- $f^{+}$ -- normalized representation of $x^{+}$
- $f_i$ -- normalized representation of $x_i$

Then the loss for $x$ is:

$$
\mathcal{L}\left(\{x, x^{+}, \{x_i\}_{i=1}^{N-1}\}; f\right)
  = \log\left(
    1 + \sum_{i=1}^{N-1} \exp\left(f^\top f_i - f^\top f^{+}\right)
  \right)
$$

Note that this is identical to classical softmax loss:

$$
\begin{aligned}
\log\left(
    1 + \sum_{i=1}^{N-1} \exp\left(f^\top f_i - f^\top f^{+}\right)
  \right) &= 
-1 \log\left(
  \frac{f^\top f^{+}}{f^\top f^{+}} + \frac{\sum_{i=1}^{N-1}\exp(f^\top f_i)}{f^\top f^{+}}
  \right)^{-1} \\
&=-\log\left(
  \frac{f^\top f^{+}}{
    f^\top f^{+} + \sum_{i=1}^{N-1}\exp(f^\top f_i)
  }
\right)
\end{aligned}
$$

## Efficient batching

Instead of computing a single loss for each batch of N+1 inputs (N-1 negatives,
1 positive, 1 anchor), the authors propose to create a batch of 2N inputs
composed of N pairs from different classes (whose embeddings should the loss
pull apart). From the 2N inputs we can compute N losses:

- take $j$-th anchor and positive as $x$ and $x^{+}$, other N-1 positives as
   $x_i$

## Hard class mining

Triplet loss relies on mining of hard instances to speed up convergence. Authors
of N-pair loss propose to mine classes instead of instances:

1. Choose large number of classes C with 2 randomly sampled instances from each
2. Get the sampled instances' embeddings
3. Greedily create a batch of N classes by:
    1. Randomly taking a class $i$ (a random instance of the randomly chosen class
      $i$)
    2. Choose $j$-th class next if it violates the triplet constraint the most
      w.r.t. the selected classes.
    3. Repeat

## Results

The authors report better results than
- triplet loss w/ hard class mining (no hard instance mining)
- classical softmax loss

However, often the most performing version of N-pair loss is using hard-class
mining, which is undoubtadly the most costly loss out of all that were
evaluated.
