---
tags: [ml]
---
# NeMo

> This note covers NeMo version 1.23, there is a newer 2.0

NeMo (Neural Modules) is a python framework built by Nvidia to speed up and
streamline experimentation with neural models.

The framework has several key components:
- NeMo's `NeuralModule` class and its API that wraps every model
- configuration
- model collections
- toolkits

For training NeMo uses [Pytorch Lightning's (PL)](./pytorch_lightning.md).

## Installation

Due to Nvidia's crappy docs, and me experiencing major setbacks during
installation process, I note few valuable lessons here.

You can either [download a docker image, use
conda](https://docs.nvidia.com/nemo-framework/user-guide/latest/installation.html)
or [build the image
yourself](https://docs.nvidia.com/nemo-framework/user-guide/latest/nemotoolkit/starthere/intro.html). 

Be wary that building the image is resource-demanding, since
[TransformerEngine](https://github.com/NVIDIA/TransformerEngine) needs to
compile something. In my case, the compilation exhausted 16GB of mem, 12 cores,
and 24GB of swap, ran for 1h and then I stopped it.

The pre-build images have [versioning
table](https://docs.nvidia.com/nemo-framework/user-guide/latest/softwarecomponentversions.html),
which could prove helpful.

## `NeuralModule`

The core of NeMo is class encompasing a model called
[`NeuralModule`](./nemo_neural_module.md). `NeuralModule` includes pretty much
everything that we do with the model:

- the forward/backward passes
- the training/validation/test steps
- saving/loading to/from a checkpoint
- tensor typing
- loading and preprocessing train/validation/test datasets
- evaluation and postprocessing of predictions

The idea is to include everything in **one** class. Instead of having a
trainer, model, data loaders, and evaluation pipeline, we just have one
`NeuralModule`.

## Hydra

TODO: CLI format


## Model collections

NeMo's model collections are divided to 3 parts based on domain:
-  `nemo.collections.asr` -- sound/speech processing, recognizing events from
   sound, transcribing speech
-  `nemo.collections.nlp` -- NLP models, LLMs, Bert-like classifiers & embedders
-  `nemo.collections.tts` -- Speech synthesis models, e.g. Tacotron, FastPitch


