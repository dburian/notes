---
tags: [ml, speech_to_text]
---
# Connectionist Temporal Classification (CTC)

CTC is an objective function for classification of token sequences introduced by
[Graves et al. (2006](https://www.cs.toronto.edu/~graves/icml_2006.pdf). It
addresses the issue of classification of tokens in variable-length sequences.

## The goal

Imagine a sequence of $N$ tokens, in which you have to find $M$ labels (speech
recognition), where $M \leq N$. You don't know where in those $N$ tokens the $M$
labels appear and so you cannot use traditional negative log likelihood for each
token to compute loss.

CTC loss approaches the problem in the following fashion:
- designs decoding algorithm to go from $N$ tokens to $M$ labels
- shows how to compute the probability of a sequence of $M$ labels, so that
  Cross-entropy can be used as a loss

## Decoding

Each of the $N$ tokens is classified into $M + 1$ labels. The extra label being
the blank $-$. Then the decoding is as follows:

- immediate repetition of the same label counts as one
- blanks are tossed

So for $aa-a-a-bb--$ we decode $aaab$. We call the predicted labelling *extended
labelling*, since we extended the true labelling by adding blanks $-$.

However, its not straightforward to obtain model's predicted extended labelling,
since there are exponentially (to $N$) many labellings the model can predicted,
each of those being represented by exponentially many extended labellings. So we
can either use

- crude approximation -- sometimes works ans is fast
- heuristics -- works better but is slower

### Crude approximation -- best path decoding

For each token we take the most probable label. Note that this can lead to
selecting different than the most probable sequence.

### Heuristics -- beam search

As we decode from the beginning we keep the $k$ most probable labellings (not
extended). At the end of the $N$ tokens, we select the most probable sequence
out of the $k$ we remember.

## Probability of a given sequence

To use cross-entropy we would like to compute $p(l|x)$, that is the likelihood
of sequence of $M$ labels $l$ given the sequence of $N$ tokens. As we discussed
above, there are exponential number of extended labellings $l'$ that correspond
to $l$. However, we can use dynamic programming to compute the probability in
$O(MN)$ time.

We imagine an idealised extended labelling $l'$ for $l$ that has blanks at the
beginning, end and between every two characters. Let us then define
$\alpha_t(s)$ as the probability that the first $t$ predicted tokens covers the
first $s$ tokens of $l'$. We set:

$$
\begin{aligned}
  \alpha_1(0) &= y^1_{-} \\
  \alpha_1(1) &= y^1_{l'_1}
\end{aligned}
$$

where $y_k^t$ is the probability of predicting label $k$ at token $t$. We
iterate over $t$. Its more helpful to imagine it like a grid (rows are $s$,
columns are $t$) as in the following image (taken from the original paper):

![Dynamic programming grid to compute
probability of "CAT" in T tokens.](./imgs/ctc_dynamic_programming.png)

The step is:

$$
\begin{aligned}
\alpha_t(s)
&= \left(\alpha_{t-1}(s) + \alpha_{t-1}(s-1)\right)y_-^t &&\;\text{if}\; l_{s-1} = -  \\
&= \left(\alpha_{t-1}(s) + \alpha_{t-1}(s-1)\right)y_{l_s}^t &&\;\text{if}\; l_{s-1} \ne - \land l_{s-2} = l_s \\
&= \left(\alpha_{t-1}(s) + \alpha_{t-1}(s-1) + \alpha_{t-1}(s-2)\right)y_{l_s}^t &&\;\text{if}\; l_{s-1} \ne - \land l_{s-2} \ne l_s \\
\end{aligned}
$$

The above computations correspond to:

1. if $l_s$ is a blank, it can be new one ($\alpha_{t-1}(s-1)$) or a repetition
   of previous one, in which case prediction at $t-1$ needed to represent the
   whole $l_{1:s}$ ($\alpha_{t-1}(s)$)
2. if $l_s$ is a non-blank character and its the same as the previous non-blank
   character, our options are the same as above, since $l_{s-1}$ needs to be
   blank, so we split up $l_s$ and $l_{s-2}$ so that they are not decoded as
   one.
3. if $l_s$ is a non-blank character and its different from the previous
   non-blank character, we can additionally skip the blank
   ($\alpha_{t-1}(s-2)$).


## Implementation

TODO
