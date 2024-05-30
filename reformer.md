[reversible_layers_paper]: https://arxiv.org/abs/1707.04585
[paper]: https://arxiv.org/abs/2001.04451
# Reformer

A memory efficient transformer with attention suited for larger sequences
[introduced by Kitaev et al. in 2020][paper].

The transformer uses [Reversible layers introduced by Gomez et al. in
2017][reversible_layers_paper] along with new self-attention based on
locality-sensitive hashing (LSH).

## LSH self-attention

The idea behind LSH attention is to use locality-sensitive hashing as an
approximation of dot-product. For similar vectors dot-product is large. After
key and query vectors are compared using dot product, the softmax is applied
which is dominated by large numbers ([Read more about classical Transformer
self-attention](./transformer_self_attention.md)). To sum up, even the classical
self-attention computes with value vectors only of very close key and query
vector pairs.

This is the motivation of LSH attention: use a hashing function on query and key
vectors such that the buckets are aproximately of equal size and similar vectors
end up in the same bucket. Then use full attention on each bucket. Essentially
it is "smart" grouping of tokens into chunks.

Approximately equally sized buckets are important to enable batching operations.
However in practice it is hard to achieve it. So instead tokens are sorted
according to buckets and this sequences is then split into several chunks.

On each chunk a full attention is computed with the addition that tokens also
attend to the previous chunk as well. This is because one bucket can be split
between chunks.

Using this scheme the attention limits the amount of dot product, but also lets
the model to decide (to a certain degree) which dot products to compute.

Some implementation details:

- Key and query vectors don't need to be different -- The authors run an
  ablation study and it turns out they can be the same (i.e. $W_Q = W_K$).

## Reversible layers

Reversible layers pretty much mean that activations don't need to be stored in
memory to be used by backward pass later. This cuts down on memory usage as
standard transformer needs $O(bldn)$ memory whereas with reversible layers you
only need $O(bld)$ for $b$ batches, $l$ input tokens, $d$ transformer dimension
and $n$ layers.

## Results

The authors claim the model is on par with the original transformer, while also
being much more memory and time efficient. Sounds like a model worth to
experiment with.


