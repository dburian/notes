---
tags: [ml]
---
# Transposed convolution

Transposed convolution is a sort-of reversed [regular
convolution](./convolution.md) that can upscale the input. We can arrive at its
definition by inspecting differentiation of regular convolution w.r.t. its
inputs. Regular convolution is defined as so:

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
  \partial I_{i^\prime, j^\prime, c}
}{
  \partial \left(K \star I\right)_{i, j, o}
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
  L
}{
\partial I_{i^\prime, j^\prime, c}
} &=
\sum_{i, j, o}
\sum_{
  \substack{m, n\\i \cdot S + m = i^\prime, j \cdot S + n = j^\prime}
} G_{i, j, o} K_{m, n, c, o} \\
&=
\sum_{m, n, o}
G_{\frac{i^\prime - m}{S}, \frac{j^\prime - n}{S}, o} K_{m, n, c, o} \\
\end{aligned}
$$

Note that the derivation in the last formula closely resembles convolution
defined above. There are two differences:

- kernels are applied for different positions -- instead of neatly placing
  kernels above the image, we need to 'find' the correct $m, n$ to use
- the loss goes over the kernel's output channels $o$ instead of its input
  channels $c$. Essentially, the kernel's last dimensions are swapped, i.e. the
  kernel is **transposed**.

This is how a transposed convolution came to be. There are nice illustrations on
[vdumoulin repo](https://github.com/vdumoulin/conv_arithmetic).

## Paddings

Naming of padding is based on the regular convolution. For example, if we went
from $w^\prime \times h^\prime$ image to a $w \times h$ output with regular
convolution and '*valid*' padding, then transposed convolution with '*valid*'
padding must go from $w \times h$ to $w^\prime \times h^\prime$. This
means that an input transposed convolution is usually heavily padded.
