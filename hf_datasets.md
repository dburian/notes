---
tags: [ml,coding]
---
# HuggingFace `datasets`

HuggingFace's `datasets` library composes several useful features:

- dataset repository -- source of data
- dataset transformation tools
- API that is useful for ML training


## `IterableDataset`

IterableDataset are efficient for ML training since they are lazy: all
transformations are done lazily and the data are *streamed* from disk. This
means that compared to classic map-stype `Dataset`

- it is more memory efficient. 
- but also that it cannot support full shuffle (it implements approximate
  shuffle with buffer)

## Sharding

Datasets can be split into multiple *shards* -- pieces of dataset, designed to
run on different [*node* (GPU)](./distributed_training.md).
`split_dataset_by_node` retrieves shards assigned to a a given node.

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
resulted in precision/rounding errors, somewhere in these to steps:
1. array of floates is encoded into bytes and stored
2. upon access the bytes are decoded into array of floates

#### Prefer casting from `bytes`

In my experience the best way how to embed audios is to load them into bytes
first:

```python
with open('audio/file.wav', mode='rb') as aud_file:
    audio_bytes = aud_file.read()
```

and then store it in `Audio()` column. This avoids all of the issues described
above.
