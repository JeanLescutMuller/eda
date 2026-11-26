# 1N_eda_<table>: one table, alone

One notebook per table, numbered in the order the task names the tables (`10_eda_tasks`, `11_eda_samples`…). The library does the per-column work; your cells cover only what it doesn't.

## The cells, in order: 6, no more

1. **First cell** (`SKILL.md`), loading this table's pickle.
2. **The profile**:

   ```python
   from ydata_profiling import ProfileReport

   ProfileReport(df_tasks, title="tasks", minimal=len(df_tasks) > 50_000).to_file("tasks.html")
   ```

   Tell the human the file's name: they open it themselves; don't read it back. Not installed: one `%pip install ydata-profiling` in a cell. If that or the report fails, don't fix it: skip it, use `column_flags.md` in this notebook, say so in one line.
3. **The grain and unique values**: a markdown line "one row = one <what>", then `df.nunique(dropna=False)` with its share of the row count. Unique on every row: a key candidate; a share near 1 that isn't 1: a key with duplicates.
4. **Key and duplicates**: rows per key value (1 = green, more = red), exact duplicate rows, duplicates on every column but the id (the same record loaded twice). For duplicated keys, which columns differ between the copies.
5. **A version, source or exporter column, if there is one**: the null share and the error share per value and per month (`px.line(color=...)`). A new version with new nulls or parse failures is the most common planted bug.
6. **The flags**: one markdown cell, every flag from the report and the cells above:

   | Flag | Status | Why |
   |---|---|---|
   | `task_id` unique | 🟢 good | unique over 1,200 rows: the key |
   | duplicates ignoring the id | 🔴 bad | 118 records loaded twice, 93 % from vendor B |

Then hand back (`SKILL.md`). Don't clean anything: propose the cleaning rule in one line; once the human agrees, one cell builds `df_<table>_clean`, reruns cell 4 on it, and pickles it.

Anything more (which columns determine which, logical rules between columns, how columns differ between groups): only when the human asks.

| The table has | Add, from |
|---|---|
| dates or times | `time.md` (3 cells) |
| free text, prompts, model outputs | `text.md` (3 cells) |
