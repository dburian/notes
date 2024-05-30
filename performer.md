[paper]:https://arxiv.org/abs/2009.14794
# Performer

An efficient transformer (but really just the attention is "new") [introduced by
Choromanski et al. in 2022][paper].

## The attention mechanism

The attention mechanism is called FAVOR+:
- **F**ast
- **A**ttention
- **V**ia
- positive
- **O**rthogonal
- **R**andom
- features

There is a quite a lot of math here, and it was not worth for me to go all the
way. So reference the paper if something is missing or unclear.

The central idea is to use an approximation of the attention matrix $A =
\exp(QK^T)$, which is normally part of the [standard self-attention
computation](./transformer_self_attention.md):

$$
Attn(Q, K, V) = D^{-1}AV\qquad
A = \exp(QK^T / \sqrt{d})\qquad
D = \text{diag}(AI_L)
$$

> I feel that for this paper to make sense, one must also look at [Transformer
> Dissection: A Unified Understanding of Transformer’s Attention via the Lens of
> Kernel](https://arxiv.org/abs/1908.11775). There the authors propose to look at
> a classical softmax attention differently and replace the softmax element with
> kernels that for each pair of query, key vectors $q_i$ and $k_j$ assign a
> positive weight. This is essentially what softmax does, but reformulated
> with kernel $K: \mathbb{R}^d \times \mathbb{R}^d \rightarrow \mathbb{R}_+$:
> 
> $$
 \DeclareMathOperator{\softmax}{\textrm{softmax}}
 \softmax(QK^T)_{i,j} =
     \sum_{i = 1}^n \frac{
             \exp(q_i k_j)
         }{
             \sum_{j^\prime = 1}^n \exp(q_i k_{j^\prime})
         }
 = \sum_{i = 1}^n \frac{
             K(q_i, k_j)
         }{
             \sum_{j^\prime = 1}^n K(q_i, k_{j^\prime})
         }
$$
>
> This is the principle of the "kernel". So in this context we can move on to the
> "Performers" paper.

Now it is easy to see that $A$ can be in fact defined by a kernel: $K: K(q_i^T,
k_j^T) = A_{ij}$. The authors suggest the kernel to compute a dot product in
different space $\mathbb{R}^r$ with mapping $\phi(x): \mathbb{R}^d \rightarrow
\mathbb{R}^r_+$. 

For reasons I not yet understand, they used a randomized mapping $\phi$ so the
kernel computes expected value of a dot product:

$$
K(x, y) = \mathbb{E}[\phi(x)^T \phi(y)]
$$

With this you can rewrite the attention defined above with $K^\prime =
\phi(k)$, $Q^\prime = \phi(Q)$, where $\phi$ is applied row-wise.

$$
\widehat{Attn}(Q, K, V) = \widehat{D}^{-1}(Q^\prime((K^\prime)^T V)),
\qquad \widehat{D} = \text{diag}(Q^\prime((K^\prime)^TI_L))
$$

The wide hats hint that this is an **approximation**. The computation then takes
$O(Lrd)$ time and only $(Lr + Ld + rd)$ memory. This is the *FAV* of FAVOR+.
*OR+* is there to define the feature function $\phi$.

Up to now it has been nothing new (at least it seems like it was for the
authors). Because random features would often give negative dot products, the
kernel could return negative weight for some query, key pair. This would lead to
instabilities and this is where this paper comes in and presents cleverly
designed features whose dot product is always positive.

Then there is something about orthogonality ... (TODO: later).
