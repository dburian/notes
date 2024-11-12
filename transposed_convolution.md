---
tags: [ml]
---
# Transposed convolution

Transposed convolution is a sort-of reversed [regular
convolution](./convolution.md) that can upscale the input.


## Derivation of the formula

We can arrive at its definition by inspecting differentiation of regular
convolution w.r.t. its inputs. Regular convolution is defined as so:

$$
\left(K \star I\right)_{i, j, o} =
\sum_{m, n, c} I_{i\cdot S + m, j \cdot S + n, c} K_{m, n, c, o}
$$

For every output position $i, j$ and every output channel $o$ we apply kernel
$K$. When we derivate w.r.t. some input position $i^\prime, j^\prime, c$, we are
essentially asking which output positions $i, j$ and kernel placements $m,
n$ used the input $I_{i^\prime, j^\prime}$.

$$
\frac{
  \partial \left(K \star I\right)_{i, j, o}
}{
  \partial I_{i^\prime, j^\prime, c}
} =
\sum_{
  \substack{m, n\\i^\prime \cdot S + m = i, j^\prime \cdot S + n = j}
} K_{m, n, c, o}
$$

If we define $G_{i, j, o} = \frac{\partial L}{\partial \left(K \star
I\right)_{i, j, o}}$ as the derivation of loss $L$ w.r.t. the convolution
outputs, we can then write:

$$
\begin{aligned}
\frac{
  \partial L
}{
  \partial I_{i^\prime, j^\prime, c}
} &=
\sum_{i, j, o}
\sum_{
  \substack{m, n\\i \cdot S + m = i^\prime, j \cdot S + n = j^\prime}
} G_{i, j, o} K_{m, n, c, o} \\
\end{aligned}
$$

Note that the derivation in the last formula closely resembles convolution
defined above. There are two differences:

- kernels are applied for different positions -- instead of neatly placing
  kernels above the image, we need to 'find' the correct $m, n$ to use
- the loss goes over the kernel's output channels $o$ instead of its input
  channels $c$. Essentially, the kernel's last dimensions are swapped, i.e. the
  kernel is **transposed**.

### The definition

For 2D input $I$ transposed convolution is defined as:

$$
\left(K \star^T I\right)_{i, j, c} =
\sum_{
  \substack{m, n, o\\i^\prime \cdot S + m = i, j^\prime \cdot S + n = j}
} I_{i^\prime, j^\prime, o} K_{m, n, c, o} \\
$$


## Intuition

Its hard to get intuition from formulas. Visualizations are much better suited
for this, e.g. on [vdumoulin
repo](https://github.com/vdumoulin/conv_arithmetic).

Looking at the visualization, we can see that transposed convolution is just a
convolution with minor adjustments:

- no padding really equals $k - 1$ zeros around sides (more on that later)
- more padding really equals less zeros around sides
- strides add $S$ zeros between each two columns and rows
- kernel's last two dimensions are swapped ($o, c = c, o$)
- kernel's first two dimensions are flipped (e.g. $m = -m$)

## Paddings

Naming of padding is based on the regular convolution. For example, if we went
from $w^\prime \times h^\prime$ image to a $w \times h$ output with regular
convolution and '*valid*' padding, then transposed convolution with '*valid*'
padding must go from $w \times h$ to $w^\prime \times h^\prime$. This
means that an input transposed convolution is usually heavily padded.
