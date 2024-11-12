---
tags: [ml, speech_to_text]
---

[paper]: https://proceedings.neurips.cc/paper_files/paper/2019/file/f63f65b503e22cb970527f23c9ad7db1-Paper.pdf
[transformer_tts_paper]: https://d1wqtxts1xzle7.cloudfront.net/107383662/1809.08895v2-libre.pdf?1700031865=&response-content-disposition=inline%3B+filename%3DClose_to_Human_Quality_TTS_with_Transfor.pdf&Expires=1730048288&Signature=c22bhjoiwMqUgo0Y5-Sk6EVtKP98r35wkkn74gn1r2pEUVlbg4FdQ3GIwSVqYER6GekdPdXwQRlo4FPnOm~O4Mpvz4bJK7H6Y5KpBXw0vr5beNCNHnLYARE2eHeHN53EfzppbMKNWQt5WYdSkRHDtVyxlYbLYfP-CRC02DCg4m6Amn8iJacYkFoK1OgmG5iQUb1fNXkH1AD-jfpSsScMbbyqxUEiegEtRA7HZbZq8QP2uiJVPBb05UaA6R3cEVB38Deva~p9-D7PcL44xJHQaXHQChJ479z7E3U510tIIc~gVO-l285nd~A1QoaP~TaoZTG-kPqRBZ6Gr~3cfiZDDg__&Key-Pair-Id=APKAJLOHF5GGSLRBV4ZA

# FastSpeech

FastSpeech is a [Transformer](./transformer.md)-based speech-to-text model with
faster inference speed than the autoregressive competition. FastSpeech was
introduced by [Ren et al. (2019)][paper].

The model is quite old, but I wanted to read it to answer two questions
that crept up during reading [FastSpeech2](./fastspeech2.md):

- why predicting phoneme length as part of the model (and in the middle of it)?
- why using 1D Convolutions in Transformer layers instead of the typical FFN?

The premise of the model is to avoid the slow autoregressive [mel
spectogram](./spectogram.md) prediction, thereby making the prediction parallel
and much quicker.

## Architecture

The whole architecture is split up to two parts:
- phoneme processing
- mel frame generation

Both steps are done using several transformer layers. '*Length Regulator*'
separates the two parts, which maps the $N$-long sequence of phonemes to
$M$-long sequence of what will be mel frames. After this mapping, new positional
embeddings are added.

### Predicting phoneme length

With parallel prediction, the model needed to resolve the many-to-many mapping.
Whereas autoregressive models can just predict mel frames until it 'feels right'
and use something like a [CTC loss](./ctc.md), FastSpeech needs to know how many
frames it needs to generate before hand. To do that, authors propose to insert a
'*Length Regulator*' that is trained to predict how many mel frames each phoneme
will take. With this information, phonemes' hidden states can be **duplicated**
to account their duration. From that point onwards, the number of mel frames is
set and the many-to-many problem resolved.

*Length Regulator* is trained according to an autoregressive [TTS
Transformer][transformer_tts_paper] encoder-decoder model. So it is a
student-model training, where the true phoneme to duration mapping is obtained
from [encoder-decoder attention](./transformer.md) coefficients.

### Transformer layers

Transformer layers are same as in the original model except for FFN after the
self-attention layer. FFN layer is replaced by:

- 1D 3k Convolution
- ReLU
- 1D 3k Convolution

The authors justified this by saying that phonemes are more locally dependent
than tokens of text. I find this rather vague but their ablation study showed
that 1D Convolutions produce more human-preferable results.
