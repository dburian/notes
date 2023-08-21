# Gradient checkpointing

When training large models, the amount of available memory becomes limiting.
Gradient checkpointing is a technique to reduce memory load during training,
while slightly increasing training time.

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
