# Writing to a notebook: one cell at a time

The human watches the notebook fill up. Each cell must appear, with its output, before you write the next one. Pick the first way that exists in your environment, and don't build your own tooling beyond it.

| Your environment | How |
|---|---|
| The notebook tool your environment gives you (a Jupyter MCP server, a hosted notebook's own agent tools) | its own "append cell" + "execute cell" calls; it saves for you |
| Jupyter is running and you have a shell | `nbformat` to append the cell, `nbclient` to run it, then write the file (below) |
| Nothing but files | append the cell with `nbformat`, say which cells to run, and don't claim outputs you did not see |

## With a shell: append, run, save

```python
import nbformat
from nbclient import NotebookClient

path = "10_eda_tasks.ipynb"
nb = nbformat.read(path, as_version=4)
nb.cells.append(nbformat.v4.new_code_cell(source))
NotebookClient(nb, kernel_name="python3", timeout=600).execute()
nbformat.write(nb, path)
```

- This reruns every cell of the notebook each time. That is fine for the short notebooks of this skill (one step each). If a notebook gets slow, keep one kernel alive (`nbclient`'s `KernelManager`, or `jupyter_client`) and run only the new cell, then write its outputs into the file.
- A new notebook: `nbformat.v4.new_notebook()`, written once, before its first cell.
- Never rewrite or reorder the human's cells; never touch a notebook the human is working in.
- A cell that errors: fix it by appending a corrected cell, or, if it's the last cell and nobody has read it yet, replace that one cell. Never leave a traceback as the last output.

## Installing a package

`%pip install <package>` in a cell of the notebook (it installs into the kernel's own environment), then restart nothing: import it in the next cell. If the install fails (no network, no wheel for this Python), say so in one line and use the fallback the sub-file names.
