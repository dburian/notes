---
tags: [ml,coding]
---
# Torch `DataLoader` in a nutshell

`DataLoader` class is able to serve data to a ML model with a set of handy
features.


## `pin_memory`

When transfering data to a CUDA device, it is necessary to move the data to a
"page-locked" are in RAM, since CUDA drivers cannot accessed pages into which
RAM is normally divided. From the page-locked area, CUDA copies the data
straight into its RAM.

*Pinning memory* means we avoid coping the data from a page in memory to a
page-locked location, by saving the data straight in the page-locked memory
location. This **speeds up** the data transfers between CPU and CUDA.

## `num_workers`

If DataLoader is given more than one worker, it spawns/forks new processes, each
having access to the same dataset. It cycles the preocesses, giving each in turn
batch of indices (if `batch_sampler` is given) or index (if `sampler` is given).
Meaning the sampler is global for all processes.

After there is a global prefetch queue (of length `prefetch_factor` *
`num_workers`) joining output data from all processes.
