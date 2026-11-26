---
name: eda
description: Data analysis in Jupyter notebooks, from the raw files to the answer - load, check each table, join, then the analysis the question calls for (what happened, did X cause Y, which is better, what will happen, what should we do, is the data or the world the same), each step its own numbered notebook the human can read while the next one is written. Plotly Express one call per plot, numbers on the plots, a good / warning / bad line under each, green / orange / red kept for what a check finds. Hierarchical - this file is the base; it says which sub-file to read for a data format (JSON, logs, text, time, geo, graphs) or a kind of analysis (A/B test, metric drop, model evaluation, forecasting, pricing, distribution shift…), each with the hypotheses to list, the test to run and the traps. Use for any data analysis, data-science exercise or interview, before any model or test.
---

# EDA: from the raw data to the answer

The human owns the question, the hypotheses and every decision about the data (what to clean, what to drop, what to conclude). This skill gets the data on the table fast, checks it, and runs the analysis the human asks for, in notebooks the human reads while you write the next one.

**This file is the base: always follow it.** Read a sub-file only when the table at the end says so. Paths are relative to this file; if you only have this file, fetch a sub-file from `https://raw.githubusercontent.com/JeanLescutMuller/eda/main/<path>`.

## The pipeline: one notebook per step

```
00_loading.ipynb          load every table, show it raw, fix the types, pickle each one        → 0_load/load.md
1N_eda_<table>.ipynb      one per table (10, 11, 12…): profile, key, duplicates, flags          → 1_table/table.md
20_joins.ipynb            the tables together: keys, rows duplicated or lost by each join       → 2_joins/joins.md
3N_<question>.ipynb       one per question (30, 31…): hypotheses, plots, tests                  → 3_analysis/analysis.md
40_report.ipynb           the answer, the numbers behind it, caveats, the recommendation        → 4_report/report.md
```

- **A new notebook for each step, named as the human says** (or as above). Once you've moved on, never edit an earlier notebook unless the human asks: the human may be reading it, and every write makes Jupyter reload it on their screen.
- **Notebooks hand over through pickles**, never through each other's cells: `00` writes `df_<table>.pkl`; a cleaned table is a new pickle (`df_<table>_clean.pkl`); every later notebook starts by loading the pickles it needs.
- **Do only what was asked, then stop.** When the human gives one step at a time, do that step and hand back: no running ahead into the next notebooks. When the human only says "analyse this", go step by step anyway and stop after each notebook with 3 bullets: what it shows · what is flagged (red / orange) · what to look at next.
- How to append, run and save one cell at a time: `notebook_io.md`.

## Rules

| Rule | Detail |
|---|---|
| Minimal code | pandas, Plotly Express, scipy / statsmodels; the library call, not your own version of it; no helper functions, no classes; a comment says *why*, never what |
| Normal formatting | one statement per line, never `;` to chain statements; a long call split over lines inside its parentheses; a cell is one step, about 15 lines at most |
| One cell at a time | write it, run it, save the notebook, then the next; append only; never reread a notebook (you know what you wrote); never load the same data twice in a notebook |
| Plots | Plotly Express only, never matplotlib; one `px` call per plot (several plots in a cell: `display(px.…, px.…)`); numbers on the plot (`text_auto=True`, or a label in the title); a title states the finding ("B misses 120 of 180 tasks"), not only what is drawn |
| Under each plot | a markdown line: **good**, **warning** or **bad**, and why, with the numbers |
| Colours | green `#2ca02c` good, orange `#ff7f0e` warning, red `#d62728` bad: only for what a check finds, never to tell categories apart. Categories, in this order: dark blue `#1f3a93`, cyan `#00bcd4`, black `#000000`, gray `#808080`, purple `#8e44ad`; never the same colour twice in one plot; 6+ categories: the top 4 and the rest as "other" in gray, or one facet per category. The same category keeps its colour in every plot (`color_discrete_map` or a fixed `category_orders`) |
| Names | `df_<table>` from the file name (`tasks.csv` → `df_tasks`); a derived frame keeps the name plus what changed (`df_tasks_clean`, `df_tasks_by_day`); `s_` a Series or a set, `l_` a list, `d_` a dict; scalars with no prefix |
| Raw data is read-only | never drop, fill, fix or deduplicate a `df_<table>` in place; a fix is a new frame, made when the human asks or agrees, its rule written in the markdown above it |
| Rigor in proportion | a mechanical cause already found (a bug, a unit, a join) is stated and sized, not tested; a test is for "could chance explain this gap?" |
| A second look after every fix | after a cleaning or a join, rerun the key and duplicate checks on the result |

The first cell of every notebook:

```python
import pandas as pd
import plotly.express as px

px.defaults.template = "plotly_white"
px.defaults.color_discrete_sequence = ["#1f3a93", "#00bcd4", "#000000", "#808080", "#8e44ad"]

df_tasks = pd.read_pickle("df_tasks.pkl")
```

## Which file to read, and when

| When | Read |
|---|---|
| writing to a notebook at all | `notebook_io.md` |
| step `00` | `0_load/load.md` |
| a file is JSON, JSONL, or has lists or dicts inside cells | `0_load/json.md` |
| parquet, Excel, SQL, several files per table, or too big for pandas | `0_load/big_or_other.md` |
| each step `1N` | `1_table/table.md` |
| `ydata-profiling` will not install or run | `1_table/column_flags.md` |
| a column of free text, prompts or model outputs | `1_table/text.md` |
| a date or time column | `1_table/time.md` |
| an event log: one row per click, call, request or session event | `1_table/events.md` |
| countries, regions, addresses or coordinates | `1_table/geo.md` |
| pairs of ids: users ↔ items, sender ↔ receiver, buyer ↔ seller | `1_table/graph.md` |
| step `20` | `2_joins/joins.md` |
| every step `3N` | `3_analysis/analysis.md` (it routes to the right file in `3_analysis/kinds/`) |
| a p-value, a confidence interval, a sample size | `3_analysis/tests.md` |
| step `40`, or "summarise", "what do you recommend" | `4_report/report.md` |

Where each part comes from: `README.md`.
