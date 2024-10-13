---
tags: [python, coding]
---
# Python Debugger

There is a python debugger called `pdb`. Don't be scared to use it.

## Usage with breakpoints

Place the following snippet when you want to set a breakpoint.

```python
import pdb

pdb.set_trace() 
```

Then when you execute a script with `python script.py` the execution stops with
pdb console, where you can ask for variable contents, step through the code or
continue until the next breakpoint.

The same thing happens when the code is executed in a notebook.

## Debug an exception

In notebooks you can just execute cell with `%pdb 1` (which turns the
debugger on) and run the code which throws the exception. The execution will
stop at the place where the execution was thrown.
