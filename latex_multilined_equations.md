---
tags: []
---

# Latex multilined equations

There are multiple $\LaTeX$ environments that can split equations into multiple
lines. The ones I have experience with are the following:

## `align`

Aligns equations on one (`&`) or more (n-times `&`) points with line splits
(`\\`) in between.

### Single aligned equation, spanning multiple lines

There are scenarios where the single aligned equation doesn't fit into one line.
Then we can split it by using `mathtools`'s environment `multlined`:

```tex
\begin{align}
& q(\mathbf{x}_{t-1}|\mathbf{x}_t, \mathbf{x}_0) =\\
&\begin{multlined}[t]
  \propto \exp \bigg(-\frac{1}{2}\bigg[
      \frac{
        (\mathbf{x}_t-\sqrt{\alpha_t}\mathbf{x}_{t-1})^2
      }{
        \beta_t
      } +
      \frac{
        (\mathbf{x}_{t-1}-\sqrt{\bar{\alpha}_{t-1}}\mathbf{x}_0)^2
      }{
        1-\bar{\alpha}_{t-1}
      } - \\
      \frac{
        (\mathbf{x}_t-\sqrt{\bar{\alpha}_t}\mathbf{x}_0)^2
      }{
        1-\bar{\alpha}_t
      }
    \bigg]\bigg)
\end{multlined}
\end{align}
```

Which gives:

$$
\begin{align}
& q(\mathbf{x}_{t-1}|\mathbf{x}_t, \mathbf{x}_0) =\\
&\begin{multlined}[t]
  \propto \exp \bigg(-\frac{1}{2}\bigg[
      \frac{
        (\mathbf{x}_t-\sqrt{\alpha_t}\mathbf{x}_{t-1})^2
      }{
        \beta_t
      } +
      \frac{
        (\mathbf{x}_{t-1}-\sqrt{\bar{\alpha}_{t-1}}\mathbf{x}_0)^2
      }{
        1-\bar{\alpha}_{t-1}
      } - \\
      \frac{
        (\mathbf{x}_t-\sqrt{\bar{\alpha}_t}\mathbf{x}_0)^2
      }{
        1-\bar{\alpha}_t
      }
    \bigg]\bigg)
\end{multlined}
\end{align}
$$
