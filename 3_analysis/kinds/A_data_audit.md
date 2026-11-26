# A. Is the data right?

Checking whether a dataset can be trusted before anyone draws a conclusion from it: a pipeline changed, a number looks off, a new source arrived. Most of the work is already in `1_table/table.md` and `2_joins/joins.md`; this file turns their flags into a verdict and sizes each problem.

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| a version, exporter or pipeline change broke the parsing | the null and error shares are the same for every version |
| rows were loaded twice (a retry, a backfill, two ids for one record) | no duplicates on the content columns without the id |
| a unit or scale changed (mm vs cm, cents vs euros, % vs fraction) | the column's median and range are steady over time and across sources |
| events stopped being logged for a platform, version or country | the row count per day is steady for every value of that column |
| a join or a filter drops or duplicates rows | row counts add up at every step (`2_joins/joins.md`) |
| the problem is real data, not a bug | the oddity has no mechanical cause in any column above |

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
