[paper]: https://arxiv.org/abs/1803.02155

# Relative position embeddings in self-attention

Described in detail in a [paper from 2018, by Shaw, Uszkoreit and
Vaswani][paper].

The idea is to add information about the relative positions of two tokens to the
computation of self-attention. So when we compute the self-attention value for
i-th output token from j-th input token, we would say to the self-attention
computation "hey you are computing attention between tokens that are j-i apart".

## Computation

### Classical self-attention and labelling

Classical self-attention computes:

$$
Attention(X) = softmax(\frac{XW_Q (XW_K)^T}{\sqrt{d}}) XW_V,
$$

where
- $W_? \in \mathbb{R}^{d\times d}$ are weight matrices,
- $X \in \mathbb{R}^{n\times d}$ are inputs,
- $d$ is the hidden dimension (or dimension of self-attention if they differ)

### Attention with relative embeddings

For this part it is better to think about self-attention in terms of one input
and one output token as such:

$$
Attention(x_i) = \sum_{j = 1}^n \frac{
    \text{exp}\, e_{ij}
}{
    \sum_{k=1}^n \text{exp}\, e_{ik}
} (x_j W_V)
$$

$$
e_{ij} = \frac{x_iW_Q (x_jW_K)^T}{\sqrt{d}}
$$

We can add relative information to the computation of keys and values as so:

$$
Attention(x_i) = dtto (x_j W_V + a^{ij}_V)
$$

$$
e_{ij} = \frac{x_iW_Q (x_jW_K + a^{ij}_K)^T}{\sqrt{d}}
$$

where $a^{ij}_?$ would be the relative information in form of learned embeddings
for the relative shift $j-i$.

- Learned positional embeddings are shared across layers and attention heads.
- Authors found there is a limit on maximum  relative shift (=$k$) after which
  the relative positional embeddings don't get more useful. In other words if
  $j-i > k$ or $j - i < -k$ the positional shift would get embedded as if it was
  just k tokens apart in positive or negative direction respectively.

### Implementation

While the classical self-attention is normally computed in matrix form and ergo
can be parallelized across GPU cores, attention with relative embeddings cannot.
To compute it efficiently computation is split into two parts -- the classical +
positional extra:

$$
e_{ij} = \frac{x_iW_Q (x_jW_K)^T + x_iW_Q (a^{ij}_K)^T}{\sqrt{d}}
$$

The classical part can be computed normally with parallelization. We get a matrix
$E$. The other part must be computed for each $i$ separately in a loop, but
since all heads share the positional embeddings it can be parallelized across
batch and attention heads.

## Performance

The authors say that **replacing** absolute embeddings with positional ones
increased scores in translation tasks while increasing the training time by 7%.

In ablation study the authors show that by omitting $a_V$ the model doesn't
decrease in performance suggesting that adding $a_V$ is not required.
