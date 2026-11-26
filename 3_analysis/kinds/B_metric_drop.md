# B. Why did the metric move?

A metric (revenue, active users, conversion, pass rate, latency) changed, and the question is why. The answer is almost always one of four things: a measurement change, a mix change, a rate change in one segment, or an outside event.

## What you'll hear

- "Daily active users dropped 10 % last week. Why?"
- "Conversion is down since Tuesday."
- "Revenue per user went up but total revenue went down."
- "Our model's accuracy in production fell this month."

## Ask first

| Ask | Why |
|---|---|
| The exact definition, and did it change (a new event, a filter, a dashboard fix)? | a third of metric drops are a change in how it's measured |
| Since when, compared to what (last week, the same week last year)? | seasonality and holidays explain many drops |
| Sudden or gradual? | sudden: a release, an outage, a logging change; gradual: a trend, a mix shift, competition |
| What shipped or happened around that date (releases, campaigns, prices, outages, holidays)? | the list of outside events to check |

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| a logging or definition change | the raw events and a second source show the same drop |
| seasonality or a calendar effect | the same period last year (or last week) shows no such drop |
| a mix shift: more of a low-value segment, same rates | the drop holds at fixed mix (decomposition below) |
| a rate drop in one segment (a platform, a version, a country, a channel) | every segment's rate is steady |
| a release or an outage | the drop doesn't start at the release date, or holds on versions without it |
| an outside cause (a competitor, a campaign ending, a market change) | the drop is the same where that cause doesn't apply |

## The notebooks

1. **The metric over time**: `px.line(df_daily, x="date", y="metric")` with last year or last period overlaid (`color="period"`), and the release and event dates as `add_vline`. Is the drop real, sudden, seasonal?
2. **Numerator and denominator apart**: a ratio drops when the top falls or the bottom grows; plot both (`px.line`, one facet each).
3. **By every segment**: for each dimension (platform, app version, country, channel, new vs returning, device), the metric per value over time (`px.line(color=dim)`), and the change per value as a sorted bar (`px.bar(text_auto=".1f")`).
4. **Mix vs rate decomposition**: for the main dimension, total change = Σ (share change × old rate) + Σ (old share × rate change) + interaction:

   ```python
   d = df_seg.pivot(index="segment", columns="period", values=["share", "rate"])
   mix = ((d.share.after - d.share.before) * d.rate.before).sum()
   rate = (d.share.before * (d.rate.after - d.rate.before)).sum()
   ```

   Shown as a bar per segment and component; it says whether the drop is "who" or "how well".
5. **Drill down**: the segment with the largest contribution, split again by the next dimension; stop when one cell explains most of the drop.
6. **Measurement check**: events per user, nulls in key fields, a new or missing event name around the date (`A_data_audit.md`).
7. **Before / after test** on the segment found, if the drop could still be noise.

## The test

`tests.md`: a two-proportion z for a rate before / after, Welch's t for a mean per user; aggregate to users or days first (rows within a user aren't independent). Many segments: Benjamini-Hochberg. A drop that starts on a release date and lives on one version is gate 0: state it.

## Traps

| Trap | Fix |
|---|---|
| Simpson's paradox: every segment flat, the total down | the mix vs rate decomposition |
| the denominator changed, not the numerator | plot both |
| a holiday or seasonality read as a drop | compare with the same period before |
| the first segment that dropped called the cause | its contribution to the total, not its own % change |
| a small segment with a large % change | rank by contribution in absolute terms |
| a logging change taken for a behaviour change | check the raw events and a second source first |

## The answer's shape

The size of the drop (absolute and %), how much of it is measurement, mix and rate, the segment and date where it lives, the most likely cause with the evidence for it, and what to check next to confirm (a rollback, a fix, a second source).
