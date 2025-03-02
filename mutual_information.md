---
tags: [information_theory]
---
# Mutual Information


Mutual information is a value which describes how much information is shared
between two random variables. It is defined as:

$$
I(X;Y) = D_{KL}\left(P_{(X, Y)} || P_X \cdot P_Y\right) =
  \mathbb{E}_{(x,y) \sim P_{(X,Y)}(x, y)} \left[\log
    \frac{
      P_{(X, Y)}(x, y)
    }{
      P_X(x) \cdot P_Y(y)
    }
  \right],
$$

where $D_{KL}$ is [Kullback-Leibler
divergence](./kullback_leibler_divergence.md).

## Helpful equations

It holds that mutual information equals [entropy](./entropy.md) of one variable
minus [conditional entropy](./conditional_entropy.md) of both:

$$
\begin{align*}
I(X;Y) &= \sum_{x \in \mathcal{X}, y \in \mathcal{Y}} p(x,y) \log \frac{p(x,y)}{p(x) p(y)} \\
&= \sum_{x \in \mathcal{X}, y \in \mathcal{Y}} p(x,y) \log \frac{p(x,y)}{p(x)}
- \sum_{x \in \mathcal{X}, y \in \mathcal{Y}} p(x,y) \log p(y) \\
&= \sum_{x \in \mathcal{X}, y \in \mathcal{Y}} p(y | X = x) p(x) \log p(y | X=x) 
- \sum_{x \in \mathcal{X}, y \in \mathcal{Y}} p(x,y) \log p(y) \\
&= \sum_{x \in \mathcal{X}} p(x) \left( \sum_{y \in \mathcal{Y}} p(y | X=x) \log p(y | X=x) \right) 
- \sum_{y \in \mathcal{Y}} \left( \sum_{x \in \mathcal{X}} p(x,y) \right) \log p(y) \\
&= -\sum_{x \in \mathcal{X}} p(x) H(Y | X = x) - \sum_{y \in \mathcal{Y}} p(y) \log p(y) \\
&= -H(Y | X) + H(Y)
\end{align*}
$$
