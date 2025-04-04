# Asynchornicity in Python

Writing asynchronous code is a bit easier than
[multiprocessing](./python_multiprocessing.md), since threads share the same
memory among [other
differences](https://en.wikipedia.org/wiki/Thread_(computing)#Threads_vs_processes).

## Recipe with `ThreadPoolExecutor`

In Python the easiest way how to use multiple threds is `ThreadPoolExecutor`
from `concurrent.futures`. The recipe goes like this:

1. Setup a function that should be carried out asynchronously

```python
def do_task(param1, param2):
  return param1 * param2, param1
```

2. Create `ThreadPoolExecutor`:

```python
from concurrent.futures import ThreadPoolExecutor

with ThreadPoolExecutor(max_workers=32) as executor:
  ...
```

3. Build an array of `Futures` by submitting a job to the executor:
```python
futures = []
with ThreadPoolExecutor(max_workers=32) as executor:
  for param1, param2 in arg_list:
    fut = executor.submit(do_task, param1, param2)
    futures.append(fut)
```

4. Wait for the tasks to finish (iteration in order of completion):

```python
from concurrent.futures import as_completed

for fut in as_completed(futures):
  mult, param1 = fut.result()
```
