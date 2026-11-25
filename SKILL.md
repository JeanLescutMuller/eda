---
name: eda
description: Exploratory data analysis (EDA) of one or more tables in a Jupyter notebook, in a fixed order - load, then each table alone (overview, each column, columns together), then the tables together (joins, coverage per group) - with short cells, plots rather than tables, and green / orange / red kept for what a check finds. Built on Anthropic's explore-data skill, the ydata-profiling alerts and the Deepchecks / Great Expectations data checks, plus the checks none of them make for comparing groups (coverage per group, a column that changes with the groups, joins). Use at the start of any data analysis, data-science exercise or interview, before any model or test.
---

# EDA: know the data before the question

The human owns the question and the hypotheses; this skill gets the data on the table fast, in a notebook the human can review live.
Work in the order below, **stop after each section** (Load, each table, cross-table) and wait.

## Notebook structure

```
# 1 Loading the data
# 2 EDA
## 2.1 Single-table analysis
### df_<table>            one per table, in the order the task mentions them
#### Overview             shape, dtypes, head
#### Single-column        one small plot per column, with its flags
#### Multi-columns        one row = ?, key and duplicates, what determines what, what changes with the groups
## 2.2 Cross-table analysis
                          join keys, coverage per group
(# 3 … the analysis: the human's, after the EDA)
```

A markdown cell opens each step: one line, what the step checks. After the step's plots: one line, what they show.

## Rules

| Rule | Detail |
|---|---|
| Short cells, minimal code | pandas + matplotlib only; one step per cell; about 10 lines a cell; at most one small plotting helper (≤ 10 lines) when the same plot is drawn 3+ times; no classes; a comment says *why*, not what |
| Plots, not tables | the only table is `head()` once per table; every number that matters is a label on a bar or in a title; a title states what the plot shows ("B is missing 120 of 180 assemblies tasks"), not only what it is |
| Colours | the data in neutral colours: `tab:blue`, `tab:purple`, `tab:gray`, `tab:cyan`, `tab:pink`, `tab:brown`, the same colour for the same group or category in every plot (the groups compared, A vs B, are neutral too). `tab:green` = check passed, `tab:orange` = warning, `tab:red` = problem: only for what a check finds (a flag in a title, a check's bar), never to fill a distribution |
| Names | `df_<table>` from the file name (`tasks.csv` → `df_tasks`); a derived frame keeps the name plus what changed (`df_samples_dedup`); `s_` a Series or a set, `l_` a list, `d_` a dict; scalars have no prefix |
| Raw data is read-only | never drop, fill, fix or deduplicate a `df_<table>`; a fix is a new frame, made only when the human asks |
| Run everything | run the cells top to bottom with the notebook's own kernel and keep the outputs; if you cannot run them, say which cells to run |
| After each section | stop and give 3 bullets: what the section shows · what is flagged (red / orange, with the check's name) · what to look at next. No answer to the business question, no fix to the data |

## 1 Loading the data

One cell: imports, the colour constants, one `pd.read_csv` per file into `df_<table>`; parse obvious dates (`parse_dates`). Read the task text (README or prompt) first: it names the tables, what one row is, and **the groups compared** (checkpoints, vendors, variants, periods): write that group column down, every later check uses it.

## 2.1 Single-table analysis, for each table

### Overview
`df.shape`, `df.dtypes`, `df.head()`. Then one line per column in markdown: its kind, from Anthropic's `explore-data`: **identifier**, **category** (≤ ~50 values), **number**, **time**, **boolean**, **text**.

### Single-column (variation)
One small plot per column, in one grid, chosen by kind:

| Kind | Plot |
|---|---|
| category, boolean | sorted horizontal bars of counts, labelled |
| number | histogram (log x if very skewed) |
| time | count per day (or per hour) |
| identifier | histogram of rows per value (a steady count is the expected shape) |
| text | histogram of lengths |

Flags, in the subplot's title, coloured by status (ydata-profiling alerts and their default thresholds; Deepchecks Data Integrity):

| Flag | Rule | Status |
|---|---|---|
| nulls | share > 1 % orange, > 20 % red (ydata MISSING; Anthropic's warning / alert) | orange / red |
| hidden nulls | placeholder values: "", "NA", "null", −1, 0, 999… where they cannot be real (Deepchecks MixedNulls) | orange |
| mixed types | numbers and strings in one column (Deepchecks MixedDataTypes) | orange |
| constant | one distinct value (ydata CONSTANT) | red |
| high cardinality | a category with > 50 values (ydata HIGH_CARDINALITY) | orange |
| imbalance | one value far more common than the others in a category of 3+ values (ydata IMBALANCE > 0.5) | orange |
| outliers | beyond 1.5 × IQR, share > 1 %; show them, never delete them | orange |
| impossible values | outside the column's domain: a rate outside [0, 1], a negative duration or count, a date in the future (Great Expectations `…to_be_between`) | red |
| spelling variants | the same category written two ways ("GPT-4" / "gpt4") (Deepchecks StringMismatch) | orange |

### Multi-columns (covariation)
1. **What one row is** (the grain), in one sentence, and the **key** it implies (often a compound one: group + item + sample).
2. **Key and duplicates**: rows per key value as a bar (1 = green; more = red). For duplicated keys, show which columns differ between the copies: if only a timestamp differs, the same result was logged twice. Exact duplicate rows too (ydata DUPLICATES).
3. **What determines what**: a matrix of the column pairs, each cell 1:1, 1:N, N:1 or N:M (`groupby(a)[b].nunique().max() == 1`). Two different columns that are 1:1 are redundant or confounded: orange.
4. **What changes with the groups compared**: for each other column, how different its mix is between the groups (total variation distance, 0 = same mix, 1 = no overlap), one bar per column: ≥ 0.9 red (it splits exactly like the groups, like a pipeline version or a run date: the groups' difference cannot be told apart from it), ≥ 0.1 orange (the groups are compared on different mixes), else green.
5. **Logical constraints**: rules that must hold between columns (passed ⇒ compiled, start ≤ end, parts sum to the total); the count of rows that break one: red if any.
6. Numeric correlations above 0.9 (ydata HIGH_CORRELATION): orange, as a heatmap only if there are 3+ numeric columns.

## 2.2 Cross-table analysis

1. **Join keys**: for each pair of tables sharing a column, a bar of keys in the left only / both / right only (`merge(indicator=True)`): orphans red. Check the join's cardinality with `merge(validate="m:1")` (or the expected one): a row explosion is red.
2. **Shared columns agree**: a column in both tables (a family, a name) must match on joined rows: the share that disagrees, red if any.
3. **Coverage per group**: the full list of items (tasks, users, prompts) against each group: a stacked bar per segment × group, complete green, missing red, duplicated orange. Groups compared on different items, or on a different mix of segments, is the most common reason a headline difference is not real.
4. **New or missing categories**: a category present for one group only (Deepchecks NewCategory): orange. A category nested in the group by design (each author works for one vendor) is expected: neutral, not orange.
5. **Overlap**: items shared between tables that should be disjoint (train / test, a public benchmark / a private set): the share shared, red if any (Deepchecks TrainTestSamplesMix).
6. **The outcome per segment**: if a table holds an outcome (passed, label, score), join it to the categories of the other tables and plot its rate per value of each category (author, family, source…), sorted, with the overall rate as a line. A value at 0 % or 100 % over 20+ rows is red: a broken segment (a grader, unit or logging bug), not a weak one (Cleanlab `underperforming_group`); far from the rest, orange. This checks the data, it does not answer the question: report it, don't explain it.

## Where this comes from

| Part | Source |
|---|---|
| grain, column kinds, thresholds, the order of the profile | Anthropic, `knowledge-work-plugins/data/skills/explore-data` |
| per-column flags and their defaults | ydata-profiling (pandas-profiling) alerts |
| hidden nulls, mixed types, spelling variants, overlap between sets | Deepchecks Data Integrity and Train-Test Validation suites |
| compound keys, value ranges, logical constraints | Great Expectations, pandera |
| short cells, a markdown note per step, run top to bottom | OpenAI, `skills/jupyter-notebook` |
| variation within a column, covariation between columns | Wickham, *R for Data Science*, Exploratory Data Analysis |
| coverage per group, what changes with the groups, joins | not covered by the profilers; added here for comparing groups (checkpoints, vendors, variants) |
