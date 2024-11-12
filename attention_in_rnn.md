---
tags: [ml]
---

# Attention mechanism in RNN

The attention mechanism in Recurrent Neural Networks (RNNs) was first intriduced
in a text translation paper written by [Bahdanau et al.
(2014)](https://arxiv.org/pdf/1409.0473).


The attention is a component that is put between RNN encoder and RNN decoder.
The encoder produces hidden states $h_j$. We label the decoder's first hidden
states as $s_i$.

> The attention mechanism allows the decoder's hidden states to condition on
> dynamically chosen (and trained) **selection** of the encoder's hidden states.
> Whereas before the attention mechanism was used, all of the encoder's hidden
> states must be condensed into a fixed sized vector.

With the attention mechanism $s_i$ is computed from:
- the previous state $s_{i - 1}$
- previous output $y_{i - 1}$
- and context vector $c_i$

The context vector is the *new thing*. It is computed as weighted average of
hidden states $h_j$, where the weights $\alpha_{ij}$ are computed based on
$h_j$'s resemblance with $s_{i - 1}$:

$$
\DeclareMathOperator{\a}{a}
\begin{aligned}
c_i &= \sum_j \alpha_{ij} h_j\\
\alpha_{ij} &= \frac{\exp(e_{ij})}{\sum_k \exp(e_{ik})}\\
e_{ij} &= \a(s_{i - 1}, h_j)
\end{aligned}
$$

Where $\a$ is a feed-forward neural network that computes the "resemblance"
between its parameters.
