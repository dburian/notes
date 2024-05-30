[paper]: https://arxiv.org/abs/1604.06174
[blog_post]: https://medium.com/tensorflow/fitting-larger-networks-into-memory-583e3c758ff9
# Gradient checkpointing

When training large models, the amount of available memory becomes limiting.
Gradient checkpointing is a technique to reduce memory load during training,
while slightly increasing training time. It was introduced by [Chen et al.
(2016)][paper]. There is also [a blog post][blog_post] about it with nice
visualizations from Open AI.

## Background

When gradients are computed activations from forward pass are needed. Let's
think about a layer in a neural network which takes inputs $x$ and performes
linear function $g$ and then non-linear activation $f$. To propagate
gradients through the layer we need to compute the followind derivatives:

$$
\frac{\partial f(g(x))}{\partial x} = \frac{\partial f(g(x))}{\partial g(x)}
\frac{\partial g(x)}{\partial x}
$$

Here we need to know the activation of our layer $f(g(x))$ as well as
activations of the previous layer $x$.

Therefore to efficiently compute gradients, we would need to store all
activations for all inputs.

**Gradient checkpointing** draws a compromise between speed and memory by
effectively selecting what activations are going to be remembered and what
activations are going to be computed on-line when backpropagation happens.

## Gradient checkpointing in detail

Read the paper/blog. Not yet needed.

## Implementation

In PyTorch it is easy as the following code taken from HuggingFace code for
RoBERTa:

```python
class RobertaEncoder(nn.Module):
    def __init__(self, config):
        super().__init__()
        ...
        self.layer = nn.ModuleList([RobertaLayer(config) for _ in range(config.num_hidden_layers)])
        ...

    def forward(
        self,
        hidden_states: torch.Tensor,
        attention_mask: Optional[torch.FloatTensor] = None,
        ...
    ) -> Union[Tuple[torch.Tensor], BaseModelOutputWithPastAndCrossAttentions]:

        ...
        for i, layer_module in enumerate(self.layer):
            ...
            if self.gradient_checkpointing and self.training:

                def create_custom_forward(module):
                    def custom_forward(*inputs):
                        return module(*inputs, past_key_value, output_attentions)

                    return custom_forward

                layer_outputs = torch.utils.checkpoint.checkpoint(
                    create_custom_forward(layer_module),
                    hidden_states,
                    attention_mask,
                    layer_head_mask,
                    encoder_hidden_states,
                    encoder_attention_mask,
                )
            else:
                ...
```

In the code above `torch.utils.checkpoint.checkpoint(func, *args)` ensures that
any computation inside `func(*args)` does **not** save intermediate results.
This means that when doing backward pass only activations from the previous
layer are stored and all activations inside a single `RobertaLayer`
(self-attention + FFN) need to be recomputed.
