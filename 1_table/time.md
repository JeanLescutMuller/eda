# Date and time columns

Dates carry most of the planted problems: a pipeline that changed on a day, a group that started later, a last month that isn't over. These cells go after `table.md`'s, in the same notebook.

## The cells, in order

1. **Parsing**: how many values failed to parse in `00` (NaT after `errors="coerce"`); red if any value that looked like a date failed. Mixed formats in one column (`2024-03-01` and `01/03/2024`): orange, and check day-first vs month-first on a day > 12.
2. **Time zones**: aware or naive (`s.dt.tz`); mixed offsets in the strings; local times without a zone. Convert everything to UTC (`tz_convert("UTC")`) and say so. Two tables in different zones joined on a date shift rows by a day: check before `20_joins`.
3. **Range**: min and max per column; a date in the future, before the product existed, or a default (`1970-01-01`, `1900-01-01`, `2099-12-31`): red.
4. **Volume per period**: `px.histogram(df, x="ts")` per day (or hour, or month), per group if any (`color=group` or `facet_row=group`).
   - gaps (a day with 0 rows): red, unless the business is closed;
   - a spike (a day with 5× the median): orange, show its rows: duplicates, a backfill, a bot;
   - the first and last periods: partial if the data starts or ends mid-period: orange, and drop them from any per-period average.
5. **Duplicates per period**: rows duplicated on the key and the date; the same event twice in one second is a logging retry.
6. **Seasonality**: count per day of week (`s.dt.day_name()`) and per hour of day (`s.dt.hour`). A weekly cycle is normal and must be compared like for like (Monday vs Monday); no cycle where one is expected is orange (a timestamp of ingestion, not of the event).
7. **Daylight saving**: in local time, a missing hour in spring and a doubled hour in autumn; counts per hour on those dates. Orange if per-hour metrics will be compared.
8. **The date confounded with a group or a version**: per group (and per version, source, exporter column), the first and last date (`groupby(g).ts.agg(["min", "max"])`, as `px.timeline` or a bar). A group that starts later than the others is compared on a different period: red if the periods don't overlap, orange if they partly do. The same for a version that changed on a date: plot the null share and error share per day, coloured by version.
9. **Event time vs ingest time**: if there are two timestamps (`created` and `loaded`, `event_ts` and `received_ts`), the lag distribution (`px.histogram(lag)`), negative lags (red: clocks or zones), and late arrivals in the last days (the last days look low because their data hasn't all arrived: orange).
10. **Order**: within a key, dates that should be ordered (created ≤ updated ≤ closed); rows that break it: red if any.

Add each finding to the flags table of `table.md`.
