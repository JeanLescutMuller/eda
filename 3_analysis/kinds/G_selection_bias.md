# G. Who's missing from this data? Selection bias

The rows you have are not a random sample of the population the question is about: only survivors, only responders, only users who reached a step, only cases someone chose to log. The job is to find who's missing and how it bends the answer.

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| the sample matches the population | its mix on key traits matches the reference |
| survivorship: only those who lasted are seen | the early cohorts' full membership, including leavers, shows the same pattern |
| response bias: who answers differs from who doesn't | responders and non-responders match on the traits known for both |
| the filter depends on the outcome (approved loans, reported incidents) | the outcome is observed for the filtered-out cases too, or the result holds on a random slice |
| the effect comes from selection, not the trait | the pattern holds in the reference population |

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
