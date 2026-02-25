---
tags: [ml,coding]
---
# HuggingFace `datasets`

HuggingFace's `datasets` library composes several useful features:

- dataset repository -- source of data
- dataset transformation tools
- API that is useful for ML training


## `IterableDataset`

`IterableDataset` are efficient for ML training since they are lazy: all
transformations are done lazily and the data are *streamed* from disk. This
means that compared to classic map-stype `Dataset`

- it is more memory efficient -- only samples that are requested are loaded
- but also that it cannot support full shuffle (it implements approximate
  shuffle with buffer)

More notes:
- the code tries to hide it, but **`IterableDataset` is subclass of `torch`'s IterableDataset**
- you can get `IterableDataset` by:
  - `datasets.load_dataset(..., streaming=True)` -- load `IterableDataset` from
    url
  - `Dataset.from_generator(..., streaming=True)` -- load `IterableDataset` from
    generator
- You **cannot** save `IterableDataset` to disk. You must first create `Dataset`
  out of it using `from_generator(..., streaming=False)` first.


## Sharding

Datasets can be split into multiple *shards* -- pieces of dataset, designed to
run on different [*nodes* (GPUs)](./distributed_training.md).
`split_dataset_by_node` retrieves shards assigned to a given node.

## `from_generator()` method

### arguments must be pickable

We can create 🤗 using an iterable using the `from_generator()`. However, 🤗
strangely **requires** the iterable to be pickable, **just so** it can generate
hash out of the arguments, cache the result and retrieve it, in case we want to
build the dataset again using the same iterable. IMHO this is rarely the case
and so there is a quick
[fix](https://github.com/huggingface/datasets/issues/6194#issuecomment-1708080653):

```python
import uuid

class _DatasetGeneratorPickleHack:
    def __init__(self, generator, generator_id=None):
        self.generator = generator
        self.generator_id = (
            generator_id if generator_id is not None else str(uuid.uuid4())
        )

    def __call__(self, *args, **kwargs):
        return self.generator(*kwargs, **kwargs)

    def __reduce__(self):
        return (_DatasetGeneratorPickleHack_raise, (self.generator_id,))


def _DatasetGeneratorPickleHack_raise(*args, **kwargs):
    raise AssertionError("cannot actually unpickle _DatasetGeneratorPickleHack!")

def gen(unpickable_argument) -> Generator:
  ...

Dataset.from_generator(_DatasetGeneratorPickleHack(gen))
```

Slightly less save, but a quick workaround is just to generate random
fingerprint:
```python
Dataset.from_generator(gen, fingerprint=str(uuid.uuid4))
```

### control how much memory generation takes up memory

`from_generator()` without `streaming=True` bufferes the results in RAM before
writing them to disk to store as intermediate results. Size of that buffer is
given by `writer_batch_size`. The larger the buffer, the more RAM you'll need
and the less often will `Dataset` create an intermediate cache file.

## Audio datasets

`datasets` can contain audio column with an `Audio()` feature, that enables to
**embed** audio within the dataset. However, I've run into some gotchas:

You can cast of the following column formats to `Audio()` to say to 🤗: "I
have this audio at this path, and I'd like you to load it and embed it.":
- string column containing path to audio
- column of dicts with
  - `array` key: loaded audio tensor (keep in mind, multiple dimensions might
    not be supported)
  - `sampling_rate` key: sampling rate of given audio
- column bytes: basically loaded audio file in memory

### Loading issues

#### Variable sampling rate

If you have variable sampling rate, I'd avoid loading via string path to audio.
In my experience `datasets` then mixes up the sampling rates, **without
resampling the audio**.

#### Embed it without path

If you want to embed the audio files without relying on the audio files, I'd
advise (even though it might not be necessary) to also avoid casting paths to
`Audio()` feature.

#### Small rounding errors when casting from dicts with `array` key

When casting a column with dicts w/ `array` key, `datasets` encodes it, together
with `sampling_rate` into `bytes`, which are then stored. In my experience, this
resulted in precision/rounding errors (~$e^{-5}$), somewhere in these to steps:
1. array of floates is encoded into bytes and stored
2. upon access the bytes are decoded into array of floates


### Prefer casting from `bytes`

In my experience the best way how to embed audios is to load them into bytes
first:

```python
with open('audio/file.wav', mode='rb') as aud_file:
    audio_bytes = aud_file.read()
```

and then store it in `Audio()` column. This avoids all of the issues described
above.
