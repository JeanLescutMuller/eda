# B. Where do users drop off?

A sequence of steps (visit → sign-up → activation → purchase; request → quote → order), and the question is where users are lost, which segments are lost more, and what changed. Usually built from an event log (`1_table/events.md`).

## What you'll hear

- "Where are we losing users in sign-up?"
- "Checkout conversion dropped: at which step?"
- "Which channel brings users who actually convert?"
- "The new onboarding: did it help?"

## Ask first

| Ask | Why |
|---|---|
| The steps, in order, and the event that marks each | the funnel's definition; a step with two possible events is ambiguous |
| Must steps happen in order, within a time window (same session, 7 days)? | changes the counts a lot |
| The unit: a user, a session, an order? | a user can enter the funnel many times |
| Can a user skip a step or enter in the middle? | strict funnels undercount; open funnels need a rule |

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| one step loses most users | the step-to-step rates are similar |
| a segment (device, channel, country, version) converts worse at one step | that step's rate is the same across segments |
| the drop is a tracking issue at one step | the next step's count is not higher than this one's (users can't skip a missing event) |
| users don't drop, they're slower | the conversion at a longer window is the same as before |
| the overall rate changed because the entry mix changed | the rate holds within each entry channel |

## The notebooks

1. **Per unit, the furthest step reached** and the time of each step: `df.pivot_table(index="user_id", columns="step", values="ts", aggfunc="min")`, steps in order.
2. **The funnel**: `px.funnel(df_counts, x="users", y="step")`, with the step-to-step and from-start conversion as text.
3. **Step rates per segment**: for each segment, the conversion of each step (`px.bar(x="step", y="rate", color="segment", barmode="group", text_auto=".0%")`); the step where segments diverge is the lead.
4. **Time to convert**: per step, the time from the previous step (`px.histogram(log_x=True)` or `px.ecdf(color="segment")`), and the share converted by day 1, 7, 30; a longer delay looks like a drop when the window is short.
5. **Over time**: each step's rate per week by entry cohort (`px.line(color="step")`), with release dates as `add_vline`.
6. **Impossible orders**: units reaching a step without the previous one; a large share is a tracking problem, not behaviour.
7. **The step to fix**: users lost at each step × the value of a conversion, as a bar: the biggest absolute loss, not the lowest rate.

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
