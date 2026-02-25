# TWIST

- tries to avoid training training LM from scratch by initializing from LM
  - authors use Open Pretrained Transformer (OPT) trained on subwords
  - used to predict phonemes after dropping embedding matricies
- similarly to AudioLM, trained only on semantic tokens
- uses HifiGAN trained from scratch on tokens
  - architecture adjusted to also ingest F0, and speaker embedding

- two metrics to evaluate LM part: sWUGGY, sBLIMP
  - both replace original text and test whether model assigns higher probability
    to original tokens vs replaced ones
  - sWUGGY replaces with non-words
  - sBLIMP replaces with gramatically incorrect words

- audio tokenizer -- played with frequency and number of tokens
  - didn't get that completely, some iterations with HuBERT

- a lot of the presentation is dedicated just to ML
- TODO: what is perplexity? how it is measured?

- high level:
 - text
 - extract semantic tokens from HuBERT
 - train LM on semantic tokens
 - run semantic tokens through vocoder

# SpeechGPT

- does something similar to TWIST
- 
