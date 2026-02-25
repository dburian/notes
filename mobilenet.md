---
tags: [ml, convolution, rough_read]
---

# MobileNet

[MobileNet
(2017)](https://www.researchgate.net/profile/Marco-Andreetto/publication/316184205_MobileNets_Efficient_Convolutional_Neural_Networks_for_Mobile_Vision_Applications/links/5f8b50c9299bf1b53e2f1419/MobileNets-Efficient-Convolutional-Neural-Networks-for-Mobile-Vision-Applications.pdf)
together with [Xception (2016)](https://arxiv.org/pdf/1610.02357)) popularized
depht-wise convolution and depth-wise separable blocks. These new types of
architectures saved up on parameter count.

## Depth-wise convolution

Depth-wise convolution is a step further from [grouped
convolution](./resnext.md) where \# of groups equals \# of input channels.
Basically input and output channels are not mixed and number of input channels
is equal to number of output channels.

$$
\left(K \star_{DW} I\right)_{i, j, c} =
\sum_{m, n} I_{i\cdot S + m, j \cdot S + n, c} K_{m, n, c}
$$

## Depth-wise separable block

Depth-wise convolution's lack of ability to mix channels between each other is
so limiting, that it is useful to add another convolution that operates on the
channels only i.e., using kernel size $1\times 1$.

Together these two convolutions are called **depth-wise separable blocks**:

1. Depth-wise convolution operating on channels separately
2. Point-wise convolution -- 1x1 regular convolution operating equally for all
  positions, but mixing input and output channels

Importantly, using depth-wise convolution always with point-wise means that we
don't loose the ability to change both channels and dimensions of the input.

