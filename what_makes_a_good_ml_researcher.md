# What makes a good ML researcher

Without a doubt an ML researcher's long-term and the *absolute* goal is to get
results.

> *Result* is a broad term, but I understand each research task as to asking a
> question: Can we increase accuracy? Will more data help us create a better
> model? How we can improve our predictions?. In this sense, *result* is just an
> answer, founded on data and experiments.

I compiled a list of recommendations that stem from my experience and
which increase the chance of getting results faster.

## Do lots of experiments

The more experiments you do the more you'll know.

## Keeping an up-to-date high-level plan

Having a plan helps to
- assign priorities/deadlines/man-days to subtasks
- get a clear overview of the direction the research is heading
- spot patterns in experiment results

## Supporting tools

In the ideal world, a researcher would just define what needs to be done, and
someone else would do it for him. Juggling running training on cluster, evaluation
results, notes, training failures, technical errors and high-level plan can lead
to a big mess, unless handled properly. Having *tools* that make each task as
streamline as possible certainly helps:
- automatic logging of configs
- logging of metrics
- good folder hierarchy
- automatic running of cheap evaluation and its logging

## Writing concise documentation

You must make sense of high volume of information coming at you. You cannot hold
that much information in your brain. You need to document it. Writing concise
notes, will help you to
- spot patterns
- see what you've missed -- maybe something wasn't founded on data, maybe we
  chose bad evaluation dataset, ...
- write your final answer -- why we did this 3 weeks back?
- have more time to do run more experiments
