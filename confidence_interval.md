---
tags: [statistics]
---

# Confidence interval

Confidence interval is a range of numbers within which you expect your estimate
to be with certain probabilty. A computed confidence interval gives you the
information that *the true value of a statistics lies within a certain interval
from the estimated statistics with a certain probability*.

## When confidence interval is useful

In statistics, you often don't know some parameter of a distribution and you
need to estimate it from a random sample. You could compute [Point
estimate](./rewrite/000026.md) from the sample but this wouldn't tell you how
probable is that you get the same value from another sample.

## How to compute confidence intervals

The TL;DR is:
1. Decide on a confidence interval. Usually you'd say we want to have 95%/99%
   certainty that the true statistics is within the given interval.
1. Compute a [point estimate](./rewrite/000026.md) of the statistics you want to
   estimate with confidence interval. We'd want to use an [unbiased point
   estimate](./rewrite/000024.md).
2. Decide on the point estimate's probability distribution, it's expected value
   and standard deviation.
3. Since the point estimate is unbiased, it's expected value is equal to the
   true statistics. Naturally, we don't know it's expected value, but we've
   proven that given the same context, such point estimates are always unbiased
   (TODO). Ergo we know the point estimate's distribution is centered around the
   true statistics.
4. Knowing the point estimate's distribution, we can compute a probability the
   point estimate lies within any given span of values. We can do also the
   opposite: find an interval centered around the distribution's mean that
   corresponds to the level of certainty we decided on in the first step. Then
   we can say: *the point estimate lies within the given distance from the
   distribution mean (=the true statistics) with 95% probability*.
5. We can also reword the statement from the previous step: *with 95% the true
   statistics lies within the given distance from our computed point estimate*.
   Once you look long enough, you realise these two sentences say the same
   thing.

### Hurdles (TODOs)

- [ ] how do you know a point estimate is unbiased
- [ ] how do you compute the point estimate's distribution and it's standard
  deviation
- [ ] show this on an example
- [ ] rewrite the linked notes into English

