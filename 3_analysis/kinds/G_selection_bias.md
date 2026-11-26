# G. Who's missing from this data? Selection bias

The rows you have are not a random sample of the population the question is about: only survivors, only responders, only users who reached a step, only cases someone chose to log. The job is to find who's missing and how it bends the answer.

## What you'll hear

- "Our survey says 85 % of users are satisfied: is that right?"
- "Customers who stayed 2 years spend more: should we push long contracts?"
- "We only have data on approved loans: is the model fair?"
- "The returned products have more defects: is the factory worse?"

## Ask first

| Ask | Why |
|---|---|
| Which population is the question about? | the gap between it and the rows is the bias |
| How did a row get into this data: a filter, a choice, a survival, a response? | the selection mechanism decides the direction of the bias |
| Is there a reference with the full population (a census, all signups, all applicants)? | compare the sample to it |
| Does the selection depend on the outcome itself? | then no reweighting on features can fix it fully |

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| the sample matches the population | its mix on key traits matches the reference |
| survivorship: only those who lasted are seen | the early cohorts' full membership, including leavers, shows the same pattern |
| response bias: who answers differs from who doesn't | responders and non-responders match on the traits known for both |
| the filter depends on the outcome (approved loans, reported incidents) | the outcome is observed for the filtered-out cases too, or the result holds on a random slice |
| the effect comes from selection, not the trait | the pattern holds in the reference population |

## The notebooks

1. **The funnel into the data**: counts from the full population to the rows you have, at each filter (`px.funnel(x="n", y="step")`); the share kept at each step.
2. **Sample vs reference**: the mix on each known trait (country, age, plan, tenure) side by side (`px.bar(x="trait_value", y="share", color="source", barmode="group", text_auto=".0%")`), TVD per trait.
3. **Responders vs non-responders** (or kept vs dropped): the traits known for both, compared; a trait that predicts response predicts bias in any outcome linked to it.
4. **Survivorship**: rebuild the cohort from its start (`B_cohort_retention.md`): the outcome for everyone who started, including those who left; compare with the survivors only.
5. **Reweighting**: post-stratification (weight each cell by population share / sample share) or a response propensity model (`logit("responded ~ traits", df_all)`, weights `1 / p`); the outcome before and after weighting, side by side (`px.bar`).
6. **Bounds when selection depends on the outcome**: the best and worst case for the missing rows (all missing satisfied / all unsatisfied): the range the truth must lie in.

## The test

`tests.md`: chi-square goodness-of-fit of the sample's mix vs the reference shares; a weighted mean's CI by bootstrap; no test fixes a selection on the outcome: report bounds.

## Traps

| Trap | Fix |
|---|---|
| the sample's rate reported as the population's | compare with a reference, reweight |
| survivors compared with survivors | rebuild the full cohort |
| reweighting on traits when selection depends on the outcome | bounds; a random audit slice |
| extreme weights | trim, report the effective sample size |
| the missing rows assumed missing at random | test it: compare kept vs dropped on known traits |

## The answer's shape

Who's missing and how many, the direction and size of the bias on the headline number (raw vs reweighted, or bounds), and the data that would remove it (a random sample, the leavers, the rejected cases).
