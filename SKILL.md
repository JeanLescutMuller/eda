---
name: eda
description: Data analysis in Jupyter notebooks, from the raw files to the answer - load, check each table, join, then the analysis the question calls for - one numbered notebook per step, written only when the human asks for it, so the human reads one while the next is written. Plotly Express, numbers and 95 % confidence intervals on the plots, a coloured good / warning / bad line under each. Works in any notebook environment (a hosted coding pad, a local Jupyter) without touching its kernel or packages. This file is the base; it names the sub-file to read for a data format (JSON, logs, text, time, geo, graphs) or a kind of analysis (A/B test, metric drop, model evaluation, forecasting, pricing, distribution shift…). Use for any data analysis, data-science exercise or interview.
---

# EDA: from the raw data to the answer

The human owns the question, the hypotheses and every decision about the data. You get the data on the table fast and run what the human asks, one step at a time.

This file is the base: always follow it. Read a sub-file only when the last table says so. If you only have this file, fetch a sub-file from `https://raw.githubusercontent.com/JeanLescutMuller/eda/main/<path>`.

## The human may be speaking, through speech recognition

Their words can arrive misheard. Read for the meaning a data scientist would intend, above all for tool and method names: "wide data profiling" or "why data" is `ydata-profiling`, "plot Lee express" is Plotly Express, "pickled" or "pickle" files, "pee value", "see I" (CI), "Simpson's". Act on the closest data-science meaning without asking; ask in one line only when two readings would lead to different work.

## Do only what was asked, then stop

- **One ask, one notebook, then hand back.** The steps below are a map, not a plan to run: don't read ahead, install, profile or prepare anything for a step the human hasn't asked for. Every minute spent ahead, the human waits.
- **All the work in the notebook, cell by cell**: no exploring the data in a shell first, no script that builds a notebook at once. Write a cell, run it, look at its output, then write its verdict.
- **Every hand-back: the notebook's name and at most 3 bullets** (what it shows · what is flagged · what to look at next). Nothing else in the chat: the detail is in the notebook. Asked only to "analyse this": go step by step anyway, the same hand-back after each notebook.

```
00_loading           load every table, show it raw, fix the types, pickle each one     → 0_load/load.md
1N_eda_<table>       one per table (10, 11…): profile, key, duplicates, flags            → 1_table/table.md
20_joins             keys, rows each join duplicates or loses, coverage                  → 2_joins/joins.md
3N_<question>        one per question (30, 31…): hypotheses, plots, tests                → 3_analysis/analysis.md
40_report            the answer, the numbers behind it, the recommendation               → 4_report/report.md
```

A new notebook per step, named as the human says. Never edit one you've moved past unless asked: the human may be reading it. Notebooks hand over through pickles (`df_<table>.pkl`, a cleaned one `df_<table>_clean.pkl`), never through each other's cells. If the environment allows only one notebook, one `## <step>` section per step instead, appended at the end.

## The environment is not yours

These hold in any environment: a hosted pad with its own agent tools, a local Jupyter, a plain folder.

| Rule | Detail |
|---|---|
| Use the notebook tools you're given | if the environment has its own way to add and run cells, use only that. A shell recipe only when there is none: `notebook_io.md` |
| **One kernel: the one already set up** | before your first notebook, look for a notebook already in the working directory (the human's, or the task's): its kernel (`metadata.kernelspec`) is the one that works here. Give every notebook you create that same kernel, run every cell on it, and install packages only into it (`%pip install` in a cell it runs). No notebook yet: the environment's default kernel, the same for every notebook |
| Never change the environment | no new or changed Python environment, no `uv` / `conda` / `pip uninstall` / upgrade, no other Python version, never restart or kill a kernel or server the human uses. The only changes allowed: installing a package into the kernel, as below |
| Check the packages once, in the first cell | its imports tell you what exists. Missing: install it as below. If that or the import still fails, don't fix it: use the fallback the sub-file names, or tell the human in one line |
| Every notebook must open | write it only through the notebook tool or `nbformat`, never as hand-written JSON; after each write, check it (`nbformat.validate(nbformat.read(path, as_version=4))`, or reopen it with the notebook tool). A notebook that fails is fixed before you say it's ready |
| A cell that errors | append a corrected cell (or replace that last cell if nobody has read it); never leave a traceback as the last output |

Installing a package, in a notebook cell, so it lands in the kernel's own Python:

```python
%pip install ydata-profiling
```

If its output says `No module named pip` (the cell doesn't error, so read the output; environments made with `uv` have no pip), install pip into the kernel's Python first, then retry once:

```python
import sys

!{sys.executable} -m ensurepip
%pip install ydata-profiling
```

Two tries at most, then the fallback.

The first cell of every notebook:

```python
import pandas as pd
import plotly.express as px

px.defaults.template = "plotly_white"
px.defaults.color_discrete_sequence = ["#1f3a93", "#00bcd4", "#000000", "#808080", "#8e44ad"]

df_tasks = pd.read_pickle("df_tasks.pkl")
```

## Code and plots

| Rule | Detail |
|---|---|
| Minimal code | pandas, Plotly Express, scipy / statsmodels: the library call, no helper functions, no classes; one step per cell, about 15 lines at most |
| Normal formatting | one statement per line, never `;`; a long call split inside its parentheses |
| Plots | Plotly Express only, never matplotlib; one `px` call per plot (several in a cell: `display(px.…, px.…)`); numbers on the plot (`text_auto=True`); a title that states the finding |
| **A 95 % confidence interval on every bar** | every bar, box or point of a mean, a rate or a share, unasked: `df_rate = df.groupby("group").passed.agg(["mean", "sem"]).reset_index()` then `px.bar(df_rate, x="group", y="mean", error_y=1.96 * df_rate["sem"], text_auto=".1%")`; Wilson for a rate near 0 or 1 (`3_analysis/tests.md`). A count of the whole dataset is exact: "exact count" in its title |
| **The verdict, in colour, under each plot** | a markdown cell, exactly one of `🟢 <span style="color:#2ca02c">**good**</span>: …` · `🟠 <span style="color:#ff7f0e">**warning**</span>: …` · `🔴 <span style="color:#d62728">**bad**</span>: …`, then why, with the numbers |
| Colours | green, orange, red only for what a check finds. Categories: dark blue `#1f3a93`, cyan `#00bcd4`, black `#000000`, gray `#808080`, purple `#8e44ad`, in that order, never one colour twice in a plot; 6+ categories: top 4 and "other" in gray. A category keeps its colour in every plot |
| Names | `df_<table>` from the file name; a derived frame adds what changed (`df_tasks_clean`) |
| Raw data is read-only | a fix is a new frame, made when the human agrees, its rule in the markdown above it; after it, rerun the key and duplicate checks |

## Which file to read, and when

| When | Read |
|---|---|
| no notebook tool, only a shell | `notebook_io.md` |
| step `00` | `0_load/load.md` |
| JSON, JSONL, lists or dicts inside cells | `0_load/json.md` |
| each step `1N` | `1_table/table.md` |
| no `ydata-profiling` | `1_table/column_flags.md` |
| free text, prompts or model outputs | `1_table/text.md` |
| dates or times | `1_table/time.md` |
| step `20` | `2_joins/joins.md` |
| each step `3N` | `3_analysis/analysis.md`, which routes to `3_analysis/kinds/` |
| a p-value, a confidence interval, a sample size | `3_analysis/tests.md` |
| step `40`, "summarise", "what do you recommend" | `4_report/report.md` |
