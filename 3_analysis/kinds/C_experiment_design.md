# C. Design an experiment: how would you test this?

The test hasn't run yet. The job is to choose the unit, the metric, the size and the duration so that the result, whatever it is, answers the decision.

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| the MDE is detectable with the traffic available | the power calculation needs more units than the traffic gives in a reasonable duration |
| the metric moves fast enough to measure in the test window | the metric (retention, LTV) only matures after the window: use a leading proxy, validated |
| units are independent | users interact (sharing, a two-sided market, a shared budget): spillover between arms |
| the weekly cycle doesn't matter | the metric differs by weekday: run full weeks |
| a pre-period covariate reduces variance | its correlation with the metric is low (CUPED gain ≈ ρ²) |

## The test

`tests.md`: the power formulas; Welch / two-proportion z at the end; delta method if the unit of analysis differs from the unit of randomization; Holm across the planned guardrails.

## Traps

| Trap | Fix |
|---|---|
| an MDE chosen to fit the traffic | start from the effect that would change the decision |
| a lagging metric (retention at 90 days) in a 2-week test | a leading proxy shown to move with it |
| part-week duration | full weeks |
| interference between arms | cluster or switchback, with the power redone |
| too many metrics or segments | one primary; segments named in the plan |
| no stopping rule | a fixed horizon, or a sequential design |

## The answer's shape

The plan table, the n and duration with the MDE they buy, the main risk (interference, a lagging metric) and how the design handles it.
