# 00_loading: every table, raw, then typed, then pickled

Goal: the human can look at every table within a minute or two of asking. Nothing here is analysis.

## Before the first cell

Read the task text (README, prompt, data dictionary). Write down, in the notebook's first markdown cell:

- the tables and what one row of each is meant to be;
- **the groups compared**, if any (checkpoints, vendors, variants, mixes, periods, arms): every later step uses that column;
- the outcome, if any (passed, converted, revenue, score).

## The cells, in order

1. The first cell of `SKILL.md` without the pickles: imports and Plotly defaults.
2. One read per file, as is: `df_<table> = pd.read_csv(path)`. Then `df_<table>.head()` for each, one table per cell, so the human sees the raw data before any change.
3. The types, one cell per table:
   - dates: `pd.to_datetime(..., utc=True)` when there's a time zone or a `Z`; say which columns were parsed and how many failed (`errors="coerce"`, then count the new NaT);
   - hidden nulls: `na_values=["", "NA", "N/A", "null", "None", "-"]` or a `replace` on the column; a sentinel number (`-1`, `999`, `0` where it can't be real) is flagged, not replaced: the human decides;
   - integers with missing values: `Int64`, not float;
   - a category (≤ ~50 values): `category`;
   - booleans written as `"yes"/"no"`, `"True"/"False"`, `0/1`: `boolean`;
   - ids stay strings (leading zeros, long numbers);
   - money, percentages, units in the text (`"12.5 %"`, `"3 mm"`): parsed into a number, the unit in the column name.
4. One cell: per table, `dtypes` next to the null count per column (`pd.concat([df.dtypes, df.isna().sum()], axis=1)`), and the row count.
5. One cell: `df_<table>.to_pickle("df_<table>.pkl")` for each table.

Then stop, with 3 bullets: the tables and their row counts; what parsing changed (dates, nulls, types); anything already odd (a column all null, a date in the future, ids not unique at first glance).

## Formats

- JSON, JSONL, nested fields: `json.md`.
- Parquet: `pd.read_parquet`; Excel: `pd.read_excel(sheet_name=None)` for every sheet; SQL: `pd.read_sql`.
- CSV traps: a `;` separator or a decimal comma (European exports: `sep=";", decimal=","`); an encoding error (`encoding="latin-1"`); a header on line 2+ (`skiprows`); a footer of totals (look at `tail()` too).
