---
tags: [hmm]
---

# Hidden Markov Model

Hidden Markov Model (HMM) models a scenario where we have an unknown (hidden)
Markov process $X$ and observable random variable $Y$ which depends on $X$. The
process $X$ gives us a sequence of *states*, which lead to a sequence of
*observations*.

## Discrete definition

The pair $(X_n, Y_n)$ is called a Hidden Markov Model if:
- $X_n$ is a Markov Process: $P(X_n | X_1 = x_1, \ldots, X_{n-1} = x_{n - 1}) =
  P(X_n | X_{n-1} = x_{n-1})$
- $X_n$ is not observable
- $Y_n$ is observable and depends only on $X_n$: $P(Y_n | X_1 = x_1, \ldots,
  X_{n-1} = x_{n - 1}) = P(Y_n |X_{n-1} = x_{n-1})$
