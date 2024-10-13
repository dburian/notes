---
tags: [ml]
---
# Guide to training neural nets

This note is gist of [Andrej Karpathy's
post](https://karpathy.github.io/2019/04/25/recipe/) about training neural
networks. There are quite a few good tips, but the blog post is dense and long,
so I'm wiriting down my take to fully grasp all the mentioned ideas.

Deep learning is not typical software development. Neural networks differ in two
important ways:

#### Training neural networks cannot be abstracted away completely

Although many libraries try to make training neural networks as simple as 2
lines of code, training neural networks cannot be abstracted completely. There
will be always quirks that you cannot debug without the full knowledge of what
is going on: from [tokenization](./tokenization_gotchas.md) and data to loss
function and backpropagation algorithm.

#### Training of neural networks fails silently

Normally when software has a bug there is a big red error popping up, more often
than not describing what exactly went wrong. Training NN isn't like that. E.g.
neural networks can learn to fix issues in input data without giving you any
indication that it is doing that, except that its performance would be slightly
lower.

For the two above reasons, you should go slowly, increasing the complexity
slowly, not in a big steps. If you add too much complexity at once, debugging
could get out of hand quite quickly.

## Recipe

### Study the data

### Implement end-to-end training and evaluation pipeline

### Overfit

### Regularize


