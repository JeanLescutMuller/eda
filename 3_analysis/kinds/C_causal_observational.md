# C. Did it work, without an experiment?

The change already happened, to some units or at some date, with no randomization. The job is to find a comparison that stands in for "what would have happened otherwise", and to say what that comparison assumes.

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| the change caused the gain | the comparison group moved the same way |
| the treated would have improved anyway (selection) | matched or weighted untreated units show the same gain |
| a concurrent event explains it | the gain also appears in untreated units at the same date |
| the pre-trends differ | treated and control trends are parallel before the change |
| the effect is an artifact of the cut-off | units just above and just below the threshold look alike on every other variable |

## The test

`tests.md`: regression with clustered standard errors by unit; bootstrap over units for matching estimates; placebo distribution as a permutation test for synthetic control.

## Traps

| Trap | Fix |
|---|---|
| correlation read as causation (feature users retain better) | ask who chooses the feature; propensity or a natural experiment |
| non-parallel pre-trends | the event-study plot; a different control group |
| a concurrent change | a control group exposed to it too, or a placebo date |
| errors not clustered by unit | `cov_type="cluster"` |
| poor overlap in propensities | trim to the common support, say who's excluded |
| one specification reported | 2–3, and the placebo tests |

## The answer's shape

The effect with its CI, the method and the one assumption it rests on, the evidence for that assumption (the pre-trend plot, the balance table, the placebo), and how a test would settle it.
