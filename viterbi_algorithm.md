---
tags: [hmm]
---

# Viterbi algorithm

Viterbi algorithm is a dynamic algorithm to find the most probable sequence of
states that produced given observations for a given [Hidden Markov
Model (HMM)](./hidden_markov_model.md).

## Algorithm

Let's assume:

- a sequence of observations: $o_1, \ldots, o_T$
- probability of transitioning from state $s_i$ to $s_j$: $P_{s_i,s_j}$
- probability of producing observation $o_i$ at state $s_j$: $O_{s_j,o_i}$
- probability of initial state $s_i$: $P_{s_i}$

then we dynamically compute:

- the maximum probability of being at state $s_i$ and producing $o_t$:
  $\pi_{t,s_i}$
- the most probable previous state $s_j$ that lead to this probability:
  $\gamma_{t, s_i}$

Now:
$$
\begin{align}
\DeclareMathOperator*{\argmax}{argmax}
\pi_{0, s_i} &= P_{s_i} \cdot O_{s_i, o_0} \\
\pi_{t, s_i} &= \max_{j}\{\pi_{t-1, s_j} \cdot P_{s_j, s_i} \cdot O_{s_i, o_t} \} \\
\gamma_{t, s_i} &= \argmax_{j}\{\pi_{t-1, s_j} \cdot P_{s_j, s_i} \cdot O_{s_i, o_t} \}
\end{align}
$$

The algorithm computes $\pi$ and $\gamma$ going from $t = 0, \ldots, T$ and them
chooses the maximum of last states: $i^* = \argmax_{i} \pi_{T, s_i}$ and
follows back the trail made by $\gamma_{T, s_{i^*}}$.

