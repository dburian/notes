---
tags: [ml]
---

# U-network

U-network is a fully convolutional network, where higher-order features captured
by deeper levels are combined with fine detail obtained by shallower levels.

Though this idea appeared in a paper by [Long et al.
(2015)](https://openaccess.thecvf.com/content_cvpr_2015/html/Long_Fully_Convolutional_Networks_2015_CVPR_paper.html),
it was later explicitely described by [Ronneberger et al.
(2015)](https://arxiv.org/abs/1505.04597).

## Architecture

![U-net architecture
(https://arxiv.org/abs/1505.04597)](./imgs/unet_architecture.png)

The network uses only convolutions. There are two directions:
- *downsampling*
  - uses convolutions and pooling operations to process the input image by
    down-scaling its spatial resolution (width and height), while increasing the
    number of channels
  - as a result gives many higher-level features drawn a large context
- *upsampling*
  - uses convolutions and transposed convolutions to upsample the higher-order
    features and combine them with intermediate outputs of the downsampling
    direction
  - as a result it propagates higher-order features into the finer image details

The two directions combined form a U-like schema, hence the name. Note that the
upsampling direction can have different resolution than the corresponding
intermediate output of the downsampling direction. This can be solved by
- cropping -- to get the same width and height
- 1x1 convolutions -- to get the same number of channels

