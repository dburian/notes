# Pip

Pip is the default package manager for python.

## Install failing

Even with requirements, the install can fail on one system, while running ok in
another.

### Building vs Downloading binaries

I've experienced that building a package required different set of dependencies.
This resulted in a wierd behaviour:
- locally pip downloaded binary
  - no build was needed
- in GilabCI pip downloaded a source package
  - build required different version of python and failed

So despite the same version of the same package was downloaded, the results
differed. One can controll if binaries or source packages are downloaded by
`--no-binary` and `--only-binary` flags for `pip install`.
