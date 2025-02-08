---
tags: [platform]
---

# Anaconda

[Anaconda](https://www.anaconda.com/) (or *conda* for short) is a high-level
package manager that sits above language-specific or OS-specific package
managers. It works on basis of separated environments, which, when activated,
provide binaries or libraries.

## Install

I install anaconda through `miniconda` that is the package manger plus minimal
overhead of code. The docs for the installation can be found [in anaconda
docs](https://docs.anaconda.com/miniconda/install/). `miniconda` installs conda
in a separate folder, meaning uninstalling is as easy as deleting a folder.

## Useful commands

To have conda available, you need to **source** it. By default, conda should
display the source code to you during install or automatically inserts it at the
end of our `.bashrc`.

Once sourced, binary `conda` is available and you can do the following:
```bash
# create python 3.10 environment
conda create -n test_env python==3.10
conda activate test_env

# installs into `test_env`
pip install torch

# list all installed conda packages
conda list

# Lists created environments
conda env list

# creates new environment as a clone of `test_env`
conda create -n other_env --clone test_env

# Installs new package into `test_env` and 'merges' `test_env` into `other_env`
conda activate test_env
pip install torchaudio
conda env export > test_env.yaml
conda env update -n other_env --file test_env.yaml
```
