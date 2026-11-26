# B. Where do users drop off?

A sequence of steps (visit → sign-up → activation → purchase; request → quote → order), and the question is where users are lost, which segments are lost more, and what changed. Usually built from an event log.

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| one step loses most users | the step-to-step rates are similar |
| a segment (device, channel, country, version) converts worse at one step | that step's rate is the same across segments |
| the drop is a tracking issue at one step | the next step's count is not higher than this one's (users can't skip a missing event) |
| users don't drop, they're slower | the conversion at a longer window is the same as before |
| the overall rate changed because the entry mix changed | the rate holds within each entry channel |

## The test

`tests.md`: a two-proportion z per step between two segments or periods (the denominator is the users who reached the previous step), chi-square across several segments; Holm or Benjamini-Hochberg across steps and segments.

## Traps

| Trap | Fix |
|---|---|
| conversion measured on all users, not those who reached the step | each step's rate on the previous step's users |
| a window too short for slow converters | compare the time-to-convert curves |
| entry mix changed, every channel flat | rates per channel; mix vs rate (`B_metric_drop.md`) |
| a missing event read as a drop-off | the impossible-order check |
| sessions and users mixed | one unit throughout |
| the lowest-rate step chosen to fix | rank by users lost × value |

## The answer's shape

The funnel with each step's rate, the step that loses the most (in users and value), the segments and dates where it's worse, whether it's behaviour or tracking, and the size of the gain if that step matched the best segment.
