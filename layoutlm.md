# LayoutLM


LayoutLM is a Transformer-based model combining image and text information to
understand structured documents such as scanned receipts or forms. There are
three versions of the model:

- [v1](./layoutlm_v1.md) -- combines text and layout information
- [v2](./layoutlm_v2.md) -- combines text, layout and image information
- [v3](./layoutlm_v3.md) -- simplifies processing of v2 into a single
  transformer


The first
version of the model, described by  while the second version, introduced by , also uses image features.

The v1 and v2 differ quite dramatically so this note describes v1 only briefly
as an introduction to processing text and image. Rest of the note is dedicated to
v2 only.

TODO: there is also a [v3](https://arxiv.org/abs/2204.08387) ...

