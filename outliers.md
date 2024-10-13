---
tags: [statistics]
---

[z_score]: z_score.md

# Outliers filtering

In data outliers are datapoints which are, informally, outside of the expected
distribution for the given statistics. It is often advantegous to remove
outliers since they do not represent the measured statistics, but may be a cause
of measurement error/corrupted data or some artificial handling of the data.

## Normal distributions

When the statistics has a normal distribution, we can use standard deviation as
a measure of how *extreme* the datapoint is. This is thanks to the fact that we
know how big of a volume we are letting-go-of by filtering out datapoints which
are x times standard deviation from the mean. The volume can be found in a
[Z score table][z_score].

https://machinelearningmastery.com/how-to-use-statistics-to-identify-outliers-in-data/

## Interquartile Range (IQR) method

Another method is to take the first quartile (25 quantile) and the third
quartile (75 quantile) and use the distance between them as a measure of how
spread out the distribution is. Such distance is called the Interquartile Range
or IQR.

We can then take IQR multiply it by some constant and move the first and third
quartile away from median (50 quantile) by the given distance. Every datapoint
that is outside of such range is considered to be an outlier.
