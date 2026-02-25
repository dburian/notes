---
tags: [ml, speech]
---

# Audio tokenizers

- VQ-VAE
  - how to quantize using VAE-like Encoder-Decoder setup
  - basics of quantization
  - straight-through estimation + codebook learning

- RVQ
  - higher precision == better reconstruction == smaller reconstruction error
  - dependent codebooks

- audio compression
  - bandwidth == bitrate = how much info we can transmit
    - pure audio = sample_rate * bit_depth (how much bits per sample) * number_of_channels
    - tokenized audio = (sample_rate / down_sample_factor) * number_of_codebooks * log_2(number_of_entries)
  - we can study compression-rate:
    - how to optimally encode indixes of codebook entry
