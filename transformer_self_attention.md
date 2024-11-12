---
tags: [transformers, ml]
---
# Transformers' self-attention

Transformers' self-attention comes from an [attention mechanism used in
RNNs](./attention_in_rnn.md), that is slightly adapted:

$$
Attention(Q, K , V) = softmax(\frac{QK^T}{\sqrt d_z}) V
$$

, where

- $Q \in \mathcal{R}^{n\times d_z}$ is a matrix of queries
- $K \in \mathcal{R}^{n\times d_z}$ is a matrix of keys
- $V \in \mathcal{R}^{n\times d_z}$ is a matrix of values
- $n$ is a number of input tokens
- $d_z$ is the dimension of query, key vectors

Normally the self-attention *block* also includes one fully connected layer:

$$
AttentionBlock(x) = Attention(xW_Q, xW_K, xW_V) W_O,
$$

where

- $W_Q, W_K$ are weight matrices $d \times d_z$ for query and key vectors
- $W_V$ is a weight matrix $d \times d$ for value vectors
- $W_O$ is a weight matrix $d \times d$ for the self-attention output

## Quadratic complexity

Self-attention is quadratic both in memory and in time in $N$. It is due to the
$QK^T$ matrix multiplication which results in a $N\times N$ matrix, and due to
the next multiplication of the previously mentioned matrix by $V$. First
multiplication takes $O(N^2)$ space and $O(N^2E)$ time, the second $O(NE)$ space
and $O(N^2E)$ time.

## Multi-head self-attention

In a normal settings the self-attention computation is done several times to
allow the attention to model several sets of token dependencies. These sets are
called heads. Each head computes its own values, queries and keys.

However, this is implemented simply as computing #head-times queries, values and
keys from the full embeddings with #head-times smaller dimensions. With clever
transpositions we achieve the same effect as if we were doing the computation
#head-times:

```python
def per_head(z: torch.Tensor) -> torch.Tensor:
  """Recieves [batch_size, seq_length, dim], outputs [batch_size, num_heads,
  seq_length, dim // num_heads]"""

  batch_size, seq_length, dim = z.shape
  z = torch.reshape(z, [batch_size, seq_length, num_heads, -1])
  return torch.permute(z, (0, 2, 1, 3))

keys = per_head(linear_keys(x))
queries = per_head(linear_queries(x))
values = per_head(linear_values(x))

attention = queries @ torch.transpose(keys, (-2, -1))
d_z = dim // num_heads
results_per_head = torch.nn.softmax(attention/torch.sqrt(d_z)) @ values

return torch.reshape(
  torch.permute(results_per_head, (0, 2, 1, 3)),
  (batch_size, seq_length, -1),
)
```

---
For some visualizations visit [Illustrated
transformer](http://jalammar.github.io/illustrated-transformer/).
