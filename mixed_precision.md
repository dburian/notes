---
tags: [ml,coding]
---
[paper]: https://arxiv.org/abs/1710.03740
# Mixed precision training

Technique used to lower GPU memory requirements and for modern cards speed up
training of neural networks introduced by [Narang et al. (2018)][paper].

The Mixed precision (MP) training saves memory by computing both the activations
and the gradients in half precision (FP16 instead of FP32). To have almost the
same accuracy in training as with FP32, there are some additional things that
need to be done:
- keep a master copy of weights in FP32 into which weight gradients are
  accumulated during the training (and from which FP16 weights are copied when
  doing next forward pass),
- scale loss before calling `.backward()` -- this ensures gradients will not be
  clipped to 0 in FP16 (since gradients are usually very small),
- do some operations still in FP32 -- some operations (mainly reductions such as
  Batch norm and softmax) are not limitted by memory bandwidth and benefit from
  the more flexible range of FP32. The authors pointed out that it was
  benefecial for these operations to compute in FP32, but write the result to
  memory in FP16. This did not slow down the training.

## Implementation in PyTorch

In PyTorch it is easy:

```python
with torch.autocast(used_device): # Automatic MP -- clever selection of precision for given operation
    loss = model(**batch)

loss = scaler.scale(loss)         # `torch.cuda.amp.GradScaler()`
loss.backward()                   # Compute gradients only after scaling the loss
```
