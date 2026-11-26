# B. Why did the metric move?

A metric (revenue, active users, conversion, pass rate, latency) changed, and the question is why. The answer is almost always one of four things: a measurement change, a mix change, a rate change in one segment, or an outside event.

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| a logging or definition change | the raw events and a second source show the same drop |
| seasonality or a calendar effect | the same period last year (or last week) shows no such drop |
| a mix shift: more of a low-value segment, same rates | the drop holds at fixed mix (decomposition below) |
| a rate drop in one segment (a platform, a version, a country, a channel) | every segment's rate is steady |
| a release or an outage | the drop doesn't start at the release date, or holds on versions without it |
| an outside cause (a competitor, a campaign ending, a market change) | the drop is the same where that cause doesn't apply |

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
