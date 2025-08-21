---
tags: [statistics, ml]
---

# Sizes of splits in ML

Though there are guides such as 80/20 or 70/30, correctly determining the size
of your splits is actually not as easy. The main aim is to **try to minimize
variance of the trained model's results on the test split.**

There are two competing concerns:
- as your train split shrinks your results are worse and have more variance
- as your test split shrinks your results have more variance

So ideally you'd want the largest train split possible, but also the largest
test split possible. So what do you do?

Before going any further, let me emphesize, that even though we care about the
**test** evaluation results, here we experiment with the **evaluation or
development** split in order to avoid overfitting on the test split.

## The subsampling technique

Essentially we are computing a [confidence interval](./confidence_interval.md)
on the validation score results by doing a random subsamples. This will give us
a good estimate of how large should validation split and therefore a test split
be, in order to give us reliable results on the given dataset.

Originally I got this technique from [Stack overflow
answer](https://stackoverflow.com/a/13623707), but did a little research
afterwards.

### 1. Initial split

Use your good guess to split your data into *train*, *dev* and *test* splits.
You can use 80/20 rule to split into rest and *test* and then again 80/20 rule
to split rest into *train* and *dev*. But this largely depends on the size of
your dataset.

**Favor larger _test_ and _dev_ splits** -- you can move samples from test and
dev splits into train, but not the other way around (you'd risk overfitting on a
part of your test split, since you used it as train initially).

### 2. Estimate the variance coming from your model and train splits

First estimate the variance that is caused by your small train split or model.
Subsample your train split a $x$ times for $y$ different sizes. The larger $x$
is going to be, the smaller your confidence interval is going to be. Larger $y$
is going to give you more granularity. You should see that as your subsamples
get larger, 2 things happen:
- the variance of the validation scores decreases
- the validation scores improve

For each subsample size you can compute, mean, $\sigma$. By assuming that the
scores are *far enough* from any lower or upper bound of the metric chosen and
that $x > 30$, we can say that the scores have [Normal
distribution](./normal_distribution.md).

Now that we know

- the distribution of the validation scores,
- their std. deviation and
- mean

we can compute their [confidence intervals](./confidence_interval.md) per each
subsample size.

The resulting confidence interval gives us a range where we expect the
validation (and the test) scores to "live".

### 3. Estimate the variance coming from your dev split

We repeat step 2. but reversed: we keep the same train split (and the same
trained model), but subsample the validation split.

As the subsamples get larger, we should see the same mean score (assuming $x$ is
large enough) but smaller variance. The computed confidence interval let's us
gauge range of scores where similarly performing models should be, given the
validation/test split size.

### Note on subsampling

I've seen [a
paper](https://proceedings.neurips.cc/paper_files/paper/2003/file/e82c4b19b8151ddc25d4d93baf7b908f-Paper.pdf)
that e.g. say using cross-validation **underestimates** the variance, cause it
treats different foldings as independent, which they're not. When subsampling
with *bootstrapping*, each subsample should be independent which should avoid
the underestimation. But I could be wrong.

No matter what, this technique cannot generalize to different datasets. No
matter how we subsample, each subsample is going to have the same underlying
distribution defined by the dataset.
