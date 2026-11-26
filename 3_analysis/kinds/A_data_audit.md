# A. Is the data right?

Checking whether a dataset can be trusted before anyone draws a conclusion from it: a pipeline changed, a number looks off, a new source arrived. Most of the work is already in `1_table/table.md` and `2_joins/joins.md`; this file turns their flags into a verdict and sizes each problem.

## What you'll hear

- "Something looks off in this dashboard since last month."
- "We changed the exporter / the logging / the vendor: did it break anything?"
- "Can we trust this data for the decision?"
- "Why don't these two reports agree?"

## Ask first

| Ask | Why |
|---|---|
| What changed, and when (a release, a new version, a new source, a migration)? | every check below is split before / after that date or by that version |
| Which number is the decision built on? | size each problem by how much it moves that number, not by how many rows it touches |
| Is there a second source of truth (a finance total, a raw log, a count from another system)? | a reconciliation is the strongest check there is |
| Can the bad rows be fixed, or only excluded? | decides the cleaning rule the human will choose |

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| a version, exporter or pipeline change broke the parsing | the null and error shares are the same for every version |
| rows were loaded twice (a retry, a backfill, two ids for one record) | no duplicates on the content columns without the id |
| a unit or scale changed (mm vs cm, cents vs euros, % vs fraction) | the column's median and range are steady over time and across sources |
| events stopped being logged for a platform, version or country | the row count per day is steady for every value of that column |
| a join or a filter drops or duplicates rows | row counts add up at every step (`2_joins/joins.md`) |
| the problem is real data, not a bug | the oddity has no mechanical cause in any column above |

## The notebooks

1. **Volume over time, by source**: `px.histogram(df, x="date", color="source")` per day or week; a step, a gap or a spike at one date is the first lead.
2. **Nulls and errors by date and version**: `df.assign(null=df[col].isna()).groupby([pd.Grouper(key="date", freq="W"), "version"]).null.mean()`, as `px.line(color="version")`. A new version with new nulls or parse failures: red.
3. **Before / after the change**: for each column, its distribution before and after (`px.histogram(color="period", barmode="overlay", histnorm="percent")`); medians and ranges side by side for numbers (units), value shares for categories (renamed or new values).
4. **Duplicates**: on every column but the id, and on the content columns that define a record; split by source and by load date (a backfill shows as one date).
5. **Units and scale**: per source and per month, `median`, `min`, `max` of each number; a factor of 10, 100 or 1,000 between two groups is a unit change.
6. **Reconciliation**: totals per day against the second source, as `px.line` of both and their ratio; a ratio that moves is a logging change.
7. **Size each problem**: the headline number with and without the affected rows (`px.bar`), so the human sees which problems matter for the decision.
8. **The verdict table**: one row per problem: what, how many rows (n and %), since when, effect on the headline, proposed fix (fix, exclude or keep).

## The test

Usually none: gate 0 of `tests.md`. A change you can point at (a version, a date, a unit) is stated and sized, not tested. A test only when the shift is gradual and could be chance: a two-proportion z on the null share before / after, or a chi-square on category shares.

## Traps

| Trap | Fix |
|---|---|
| a p-value on a bug | state the cause and size it |
| problems counted in rows, not in effect | size each one on the headline number |
| the time and the source confounded (a source only exists after a date) | compare within the period where both exist |
| a fix applied in place | a new `_clean` frame, its rule in the markdown, the human's call |
| one problem found, the search stopped | run every check above; planted bugs come in groups |
| a second look skipped | rerun the duplicate and null checks on the cleaned table |

## The answer's shape

A verdict per problem (🟢 fine, 🟠 usable with a caveat, 🔴 must be fixed or excluded), each with its size, its date or version, and its effect on the decision's number; then the cleaning rules proposed for the human to accept, and whether the data can support the decision once they're applied.
