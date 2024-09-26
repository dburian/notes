# Multiprocessing in Python

By default python code is synchronous -- it runs in single process and in single
thread. However, the `multiprocessing` module allows for execution in multiple
processes. There are few gotchas that one needs to be aware of.

## Forking is bad

The default method how to obtain another process is to fork the current one.
Forking copies all memory of the parent process to the child process, however,
it doesn't copy everything. This can [cause
deadlocks](https://pythonspeed.com/articles/python-multiprocessing/). So it is
recommended to use other methods such as 'spawn':

```python
from multiprocessing import get_context

with get_context('spawn').Pool as p:
  p.imap(...)
```

## When spawning the worker function should be from another module

With the `spawn` method, a new python interpreter is created. This means it has
no clue about the globals in the parent process and needs to import them. So if
the worker function is in the main module of the parent process, there is a risk
of running that multiprocessing again in the child process. Aparently there are
some boundaries, so it doesn't happen. The solution is to either have the worker
function in another module, or maybe adding the `if __name__ == '__main__':`
before multiprocessing code could help.

## Introducing `joblib`

With the mentioned above hassles in mind, I found that the experience using
`joblib` is much better. It is an external package with no further dependencies.
And it works like so:

```python
from joblib import Parallel, delayed

with Parallel(
  n_jobs=32, # By default num. of CPUs
  verbose=11 # Above 0 it logs something, above 10 it reports on every iteration
  batch_size=1 # Number of runs per CPU
  ) as parallel:
  result = parallel(delayed(expensive_fn)(args, kwarg1=other_argument) for args in prepared_job_args)
```

From my experience it is quicker or as fast as spawn multiprocessing, you don't
have to create single-argument functions and place them in different modules,
and I've encountered zero unexpected errors.
