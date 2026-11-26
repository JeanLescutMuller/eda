# No notebook tool, only a shell: append, run, save one cell

Only when the environment gives no way to add and run cells. Same rules as `SKILL.md`, "The environment is not yours": nothing here installs, restarts or changes anything.

```python
import nbformat
from nbclient import NotebookClient

path = "10_eda_tasks.ipynb"
nb = nbformat.read(path, as_version=4)
nb.cells.append(nbformat.v4.new_code_cell(source))
NotebookClient(nb, timeout=600).execute()
nbformat.write(nb, path)
nbformat.validate(nbformat.read(path, as_version=4))
```

- One cell per call, run before the next is written. This reruns the notebook's cells each time: fine for one-step notebooks.
- `NotebookClient` uses the notebook's own kernel (its metadata, else the default one): never name another kernel or Python.
- A new notebook: `nbformat.write(nbformat.v4.new_notebook(), path)` once, before its first cell.
- `nbformat` or `nbclient` missing: write the cells with `nbformat` only if it exists, tell the human which cells to run, and claim no output you didn't see.
