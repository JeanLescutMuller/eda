# Date and time columns: 3 cells

After `table.md`'s cells, in the same notebook.

1. **Parse failures and range**: values that failed to parse in `00` (NaT after `errors="coerce"`), and min / max. Any failure, a date in the future or a default (`1970-01-01`, `2099-12-31`): red. Mixed time zones: convert to UTC and say so.
2. **Volume per day, by group**: `px.histogram(df, x="ts", color=group)`. A gap (a day with 0 rows): red; a spike (5× the median): orange; a partial first or last period: orange, left out of any per-period average.
3. **The date confounded with a group or version**: per group (and per version or source), the first and last date (`df.groupby(g).ts.agg(["min", "max"])`). A group that starts later is compared on a different period: red if the periods don't overlap, orange if they partly do.

Add each finding to the flags table.
