# Beginners guide to ASR

I write this, since I'm a beginner in speech ML (TTS, ASR/STT). This is what has
helped me get started.

## Common libraries

- loading files: `librosa`
- processing input data: `spark`

## Common values

- sampling rate: 16k

## Common metrics

- Word Error Rate (WER) -- word edit distance over the number of true words:

$$
\operatorname{WER}(y_\text{pred}, y_\text{true}) = \frac{
  \operatorname{ed}_w(y_\text{pred}, y_\text{true})
  }{
    |y_\text{true}|
  }
$$

## Common approaches


