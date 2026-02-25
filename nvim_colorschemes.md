---
tags: [editor]
---

# Nvim colorschemes

- `Inspect` command


- nvim tries to guess background from terminal color
- if the user didn't set it manually, nvim will guess it and apply
  automatically
- if nvim applies it, the colors are reset
- ergo to make my colorscheme stick, I have to set background color
- setting it right after `highlight clear` did the trick
- it seems it doesn't matter whether it agrees with the actual colors
  (either dark or light, just set it)
