---
tags: [platform, python]
---

# PyArrow

[PyArrow](https://arrow.apache.org/docs/python/index.html) is the python
implementation of Apache Arrow data format. `pyarrow` is used by
[🤗 `datasets`](./hf_datasets.md) under the hood.

## Docs

`pyarrow` is mostly just wrapper around C++ and CPython code. This makes it hard
to find more information if the
[API reference](https://arrow.apache.org/docs/python/api.html) doesn't suffice
as python code is often not existent. Ergo here I list sources that I found
helpful:

- `datasets`'s codebase -- grep `datasets`'s code to see how 🤗 uses `pyarrow`
  (my ability to read CPython code is minimal).
- [official codebooks](https://arrow.apache.org/cookbook/py/index.html) -- code
  snippets illustrating usage for concrete usecases

## Concepts

### Data structures

`pyarrow` uses file structures [represented by
objects](https://arrow.apache.org/docs/python/data.html). So you can just write:
```python
import pyarrow as pa

# 1D array
arr = pa.array([1, 2, 3, 4])

# Batch of equally sized arrays
batch = pa.RecordBatch.from_arrays([
    pa.array([1, 2, 3, 4]),
    pa.array([True, True, False, False])
  ],
  names=['tokens', 'mask']
)

# batches and tables have schema
batch.schema

# Table of several batches
table = pa.Table.from_batches([batch] * 5)
```

### Writing/Reading data

PyArrow can write and read data in multiple formats (csv, jsonl, parquet,
feather), but the main format is obviously `.arrow`.

TODO: write down high-level overview, once you understand it:
- why is IPC serialization used so often
- why is `.arrow` so good, when you can [memmory map even a `.txt`
  file](https://arrow.apache.org/docs/python/generated/pyarrow.memory_map.html)
