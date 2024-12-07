---
tags: [ml]
---
# NeMo

NeMo (Neural Modules) is a python framework built by Nvidia to speed up and
streamline experimentation with neural models.

The framework has several key components:
- NeMo's `NeuralModule` class and its API that wraps every model
- configuration
- model collections
- toolkits

For training NeMo uses [Pytorch Lightning's (PL)](./pytorch_lightning.md).

## `NeuralModule` class

`NeuralModule` is torch's `torch.nn.Module` and PL's `LightningModule` plus all
functionality needed to train, evaluate or finetune a model. So `NeuralModule`
is able to:

- load dataset
- preprocess dataset
- postprocess predictions
- plus all stuff from `torch.nn.Module` -- forward pass
- plus all stuff from `LightningModule` -- configure optimizers, schedulers,
  define train/val/test steps

### Checkpoints and serialization

## Configuration

## Model collections

NeMo's model collections are divided to 3 parts based on domain:
-  `nemo.collections.asr` -- sound/speech processing, recognizing events from
   sound, transcribing speech
-  `nemo.collections.nlp` -- NLP models, LLMs, Bert-like classifiers & embedders
-  `nemo.collections.tts` -- Speech synthesis models, e.g. Tacotron, FastPitch


