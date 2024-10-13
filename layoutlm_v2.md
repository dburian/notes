---
tags: [transformers,ml]
---
# LayoutLMv2

Second version of LayoutLM model that combines text, layout and visual
information. Introduced by [Xu et al. (2020)](https://arxiv.org/abs/2012.14740).

Compared to the [first version](./layoutlm_v1.md) the Transformer now also
processes image.

### Architecture

The v2 version differs from standard Transformer in the used embeddings and in
the self-attention mechanism.

#### Used embeddings

The image is downscaled to create a low-dimensional feature map and then
processed by a visual encoder (originally ResNeXt-FPN), which is trained
simultaneously with the transformer. The visual encoder outputs a fixed sized
embeddings for different parts of the image. These image embeddings are resized
to have the same dimensionality as token embeddings. All tokens (image and text)
receive additional embeddings:

1. Segment embedding -- different segments for image, text (text is further
   split to two segments if the input's format is question and answer).
2. 1D embedding -- visual and text tokens get different ordering embeddings
3. 2D embedding -- each image and text token gets a bounding box embedding

All embeddings are summed up.

#### Self-attention

The standard [self-attention](./transformer_self_attention.md) models the
attention scores of token $t_i$ and $t_j$ as follows:

$$
\alpha_{ij} = \frac{1}{\sqrt{d_{\text{head}}}}
  (t_i\mathbf{W}^Q)
  (t_j\mathbf{W}^K)^T
$$

LayoutLMv2 adds relative positional biases (1D and 2D). Assuming $t_i$'s
top-left corner of its bounding box has coordinates $(x_i, y_i)$, the
self-attention scores are adjusted as so:

$$
\alpha_{ij}^\prime =
  \alpha_{ij} +
  b^{(1D)}_{j-i} +
  b^{(2D_x)}_{x_j - x_i} +
  b^{(2D_y)}_{y_j - y_i}
$$

![LayoutLMv2 architecture. source:
https://arxiv.org/abs/2012.14740](./imgs/layoutlmv2.png "LayoutLMv2
architecture")

The authors called this adjusted self-attention Spatial-Aware Self-Attention
Mechanism (SASAM). SASAM boosts the performance of the model quite
significantly.

### Pretraining

The v2 model pretraines using 3 losses: adjusted MVLM, TIA, TIM.

MVLM from v1 remains, but the image region corresponding to the masked text
token is also masked to avoid spilling information from visual to text and force
the Transformer to look at neighboring tokens.

Text-Image alignment (TIA) loss covers lines in the image (since covering words
in the low-resolution feature map would be too noisy). Each token is then
classified as being covered or not, trained with Cross-Entropy loss. TIA should
boost the model's capacity to spatially align the text and the visual
information.

Text-Image matching (TIM) loss uses `[CLS]` token embedding and does binary
classification based on whether the input text is from the input image. TIM
should motivate the model to connect the image and textual information in a more
coarse fashion.

The results of combinations of losses are as follows:
- MVLM + TIA is better than MVLM + TIM, which barely improves just MVLM
- MVLM + TIA + TIM is slightly better than MVLM + TIA

