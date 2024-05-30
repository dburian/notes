# Transformers' self-attention

Transformers self-attention computes as so:

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

---
For some visualizations visit [Illustrated
transformer](http://jalammar.github.io/illustrated-transformer/).
