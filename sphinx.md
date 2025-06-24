# Sphinx

[Sphinx](https://www.sphinx-doc.org/) is a python tool to generate
documentation. It's very versatile, and therefore the specific usecases might
not be clear.

## Quickstart

Steps:
1. have a locally cloned package e.g. `datasets` from 🤗
2. inside the package directory create a `sphinx` directory
3. install the package to be able to import all of its modules
4. install `sphinx`
5. run `sphinx-quickstart` inside `sphinx` directory and use separate source and
   build directories:
```bash
pwd
# > /home/user/repos/datasets

cd sphinx
pwd
# > /home/user/repos/datasets/sphinx

sphinx-quickstart
# > Separate source and build directories (y/n) [n]: y
```
6. Run `sphinx-apidoc` to populate the `sphinx/source` directory:
```bash
cd ../
pwd
# > /home/user/repos/datasets/

sphinx-apidoc -o sphinx/source src/datasets
```

7. Adjust `sys.path` inside `sphinx/source/conf.py`. You have to edit `sys.path`
   such that the package is a subdirectory. Since `datasets` use the 'src'
   structure, we need to:
```python
import os
import sys
sys.path.insert(0, os.path.abspath('../../src'))
```

8. Adjust `extensions` inside `sphinx/source/conf.py`. I usually use:

```python
extensions = [
    'sphinx.ext.autodoc', # Required to make this setup work
    'sphinx.ext.napoleon', # Required for Google or Numpy style docstrings
    'sphinx.ext.intersphinx',
    "sphinx.ext.viewcode",
    "sphinx.ext.githubpages",
]
```

9. Adust `html_theme` inside `sphinx/source/conf.py`. I use
   `pydata_sphinx_theme`, but have a look at
   [sphinx-themes.org](https://sphinx-themes.org/), there are bunch more. To use
   `pydata_sphinx_theme` you need to install it first:
```bash
pip install pydata-sphinx-theme
```

then adjust the `html_theme` variable inside `sphinx/source/conf.py`:
```python
html_theme = 'pydata_sphinx_theme'
```

10. Generate the html documentation:
```bash
cd sphinx
pwd
# > /home/user/repos/datasets/sphinx/

make html
```

11. Preview the documentation using a `http.server`:
```bash
cd build/html
pwd
# > /home/user/repos/datasets/sphinx/build/html

python -m http.server -b 127.0.0.1
```

---
Source:
- inspired by a [blog post by Cissy
  Shu](https://medium.com/@cissyshu/a-step-by-step-guide-to-automatic-documentation-using-sphinx-a697dbbce0e7)


