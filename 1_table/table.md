# 1N_eda_<table>: one table, alone

One notebook per table, numbered in the order the task names the tables (`10_eda_tasks`, `11_eda_samples`…). The per-column work is offloaded to a library; your own cells cover only what it doesn't: the grain, the key, the duplicates, what determines what, what changes with the groups.

## The cells, in order

1. **First cell** (`SKILL.md`), loading this table's pickle.
2. **The profile**, one cell:

   ```python
   from ydata_profiling import ProfileReport

   ProfileReport(df_tasks, title="tasks", minimal=len(df_tasks) > 50_000).to_file("tasks.html")
   ```

   Tell the human the file's name: they open the HTML themselves. Don't parse the report back: it's dense, minified HTML. Read its alerts from the object instead if you need them (`report.get_description().alerts`).
   Not installed: one try, `%pip install ydata-profiling` in a cell. If it fails (it depends on `numba`, which can lag the newest Python), **don't fix it**: no other environment, no other Python. Skip the profile and use `column_flags.md`, in this same notebook. Say so in one line.
3. **The grain**: a markdown line, "one row = one <what>", and the key it implies (often compound: group + item + sample).
4. **Unique values per column**: `df.nunique(dropna=False)` with its share of the row count. A column unique on every row is a key candidate; a share near 1 that isn't 1 is a key with duplicates.
5. **Key and duplicates**: rows per key value as a bar (1 = green, more = red); exact duplicate rows; duplicates on every column but the id (the same record loaded twice under two ids); for duplicated keys, which columns differ between the copies (only a timestamp: the same result logged twice).
Cells 1–5 and 10 always. Cells 6–9 only when they apply (two-plus categories, groups compared, rules between columns, a version or source column), one cell each.

6. **What determines what**: for each pair of columns, 1:1, 1:N, N:1 or N:M (`df.groupby(a)[b].nunique().max() == 1`), as a matrix (`px.imshow`, `text_auto=True`). Two different columns that are 1:1 are redundant or confounded: orange.
7. **What changes with the groups compared** (if the task compares groups): for each other column, how different its mix is between the groups (total variation distance: 0 = same mix, 1 = no overlap), one bar per column: ≥ 0.9 red (it splits exactly like the groups, like a pipeline version or a date: the groups' difference can't be told apart from it), ≥ 0.1 orange, else green.
8. **Logical constraints**: rules that must hold between columns (passed ⇒ compiled, start ≤ end, parts sum to the total, a child's date after its parent's); rows that break one: red if any.
9. **A change of source, version or format over time**: if a column names a pipeline, an exporter, a version or a vendor, plot the null share and the error share per value and per month. A new version that brings new nulls or parse failures is the most common planted bug.
10. **The flags**: one markdown cell, a table of every flag, from the report and from the cells above:

    | Flag | Status | Why |
    |---|---|---|
    | `task_id` unique | 🟢 good | unique over 1,200 rows: the key |
    | duplicates ignoring the id | 🔴 bad | 118 records loaded twice, 93 % from vendor B |

Then stop (`SKILL.md`). Don't clean anything: propose the cleaning rule in one line and wait for the human's call; then a new cell builds `df_<table>_clean`, reruns cells 4–5 on it (a second look after the fix), and pickles it.

## Read as well, when the table has

| Column | Read |
|---|---|
| free text, prompts, model outputs | `text.md` |
| dates or times | `time.md` |
| one row per event (clicks, calls, requests) | `events.md` |
| places, coordinates | `geo.md` |
| two ids that link entities (user ↔ item) | `graph.md` |
