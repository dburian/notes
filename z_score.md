# Z score and Z score tables

Z score is simply a normalized distance to the mean:

$$
z = \frac{x - \mu}{\sigma}
$$

It is often used to compute the area under the normal distribution between two
points $a$ and $b$:

$$
\int_{a}^{b} \frac{1}{\sigma \sqrt{2\pi}} e^{-\frac{1}{2}(\frac{x - \mu}{\sigma})^2} \,dx
$$

It turns out that there isn't any function comprised of elementary functions
that is an antiderivative to the above. I.e. there is no analytical solution.
We can approximate it with numericall methods, but we would need to precompute
it for each $\mu$ and $\sigma$. This is where z-scores come in. For some
z-scores we can pre-compute the percentage of volume under the curve and write
it in a table. Such table would be called a z-score table.

