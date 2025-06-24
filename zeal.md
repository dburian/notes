---
tags: [tools]
---

# Zeal

[Zeal](https://zealdocs.org/) is a offline documentation browser with plenty of
available docsets, either from the app itself or downloadable from [User
contributions](https://zealusercontributions.vercel.app/).

Zeal is a free app for Windows and Linux, that uses docsets in the same format
as [Dash](https://kapeli.com/dash) -- a paid app for MacOS. That's why the two
apps use some of the same tools.

## Generating your own docset

There are plenty of options to do so -- described in [dash
docs](https://kapeli.com/docsets). I, however, only have experience with using
[Sphinx](./sphinx.md) and then [`doc2dash`](https://pypi.org/project/doc2dash/)
package.

### Sphinx + `doc2dash`

This method generates html docs from a locally cloned and installed package and
then uses [`doc2dash`](https://pypi.org/project/doc2dash/) package to generate a
Dash docset out of it.

To generate html documentation see [Sphinx](./sphinx.md). Then turn the 
`build/html` folder with `objects.inv` into a docset using:
```bash
doc2dash -n "package_name" -d './docset/' -j 'path/to/build/html'
```

That's it. Move the generated `./docset/package_name.docset` dir into
`~/.local/share/Zeal/Zeal/docsets/` or wherever your Zeal stores docsets and
you're good to go.
