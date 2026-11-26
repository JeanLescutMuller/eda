# No notebook tool, only a shell: append, run, save one cell

Only when the environment gives no way to add and run cells. Same rules as `SKILL.md`, "The environment is not yours": nothing here installs, restarts or changes anything.

## Once: the kernel already set up

```python
from pathlib import Path

import nbformat

l_existing = sorted(Path(".").glob("*.ipynb"), key=lambda p: p.stat().st_mtime)
d_kernelspec = None
for p in reversed(l_existing):
    d_kernelspec = nbformat.read(p, as_version=4).metadata.get("kernelspec")
    if d_kernelspec:
        break
```

The newest notebook that has a kernel gives it; every notebook you create gets the same one. `None`: the default kernel, for all of them.

## A new notebook, then each cell

```python
from nbclient import NotebookClient

path = "10_eda_tasks.ipynb"
nb = nbformat.v4.new_notebook()
if d_kernelspec:
    nb.metadata["kernelspec"] = d_kernelspec
nbformat.write(nb, path)

nb = nbformat.read(path, as_version=4)
nb.cells.append(nbformat.v4.new_code_cell(source))
NotebookClient(nb, timeout=600).execute()
nbformat.write(nb, path)
nbformat.validate(nbformat.read(path, as_version=4))
```

- One cell per call, run before the next is written. `NotebookClient` runs it on the notebook's own kernel: never name another kernel or Python.
- A package: `%pip install <package>` as a cell of the notebook, run the same way, so it lands in that kernel's environment.
- `nbformat` or `nbclient` missing: tell the human which cells to run, and claim no output you didn't see.
