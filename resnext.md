---
tags: [ml, convolution, rough_read]
---

# ResNeXt

ResNeXt was an evolution of [ResNet](./convolution_in_ml.md) introduced by [Xie
et al.
(2017)](https://openaccess.thecvf.com/content_cvpr_2017/papers/Xie_Aggregated_Residual_Transformations_CVPR_2017_paper.pdf).

TL;DR the architecture saved up on parameters by grouping the convolution
filters into segments. Input filter from group $i$ didn't participate on the
computation of output filter $j$ ($i \ne j$), thereby saving on parameters. This
new type of convolution was named **grouped convolution**.

![Equivalent visualizations of a ResNeXt block.
(https://openaccess.thecvf.com/content_cvpr_2017/papers/Xie_Aggregated_Residual_Transformations_CVPR_2017_paper.pdf)](./imgs/resnext_block.png)
