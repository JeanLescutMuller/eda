# C. Design an experiment: how would you test this?

The test hasn't run yet. The job is to choose the unit, the metric, the size and the duration so that the result, whatever it is, answers the decision.

## What you'll hear

- "How would you test whether this new feature increases engagement?"
- "How many users do we need, and how long should it run?"
- "We can only test on 5 % of traffic: is that enough?"
- "Users share content with each other: how do we test a change to sharing?"

## Ask first

| Ask | Why |
|---|---|
| Which decision does the test serve, and what's the smallest effect worth shipping? | that's the MDE (minimum detectable effect); everything is sized from it |
| What's the baseline rate or mean, and its variance per unit? | the sample size depends on it; take it from historical data |
| How much traffic per day can go into the test? | duration = needed units / daily units per arm |
| Do units affect each other (social features, marketplaces, shared inventory)? | interference breaks a user-level split: cluster or switchback |

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| the MDE is detectable with the traffic available | the power calculation needs more units than the traffic gives in a reasonable duration |
| the metric moves fast enough to measure in the test window | the metric (retention, LTV) only matures after the window: use a leading proxy, validated |
| units are independent | users interact (sharing, a two-sided market, a shared budget): spillover between arms |
| the weekly cycle doesn't matter | the metric differs by weekday: run full weeks |
| a pre-period covariate reduces variance | its correlation with the metric is low (CUPED gain ≈ ρ²) |

## The notebooks

1. **The baseline from history**: the metric per unit over past weeks, its mean and standard deviation (`df.groupby("user_id").y.sum().agg(["mean", "std"])`), and per weekday (`px.box(x="weekday", y="y")`).
2. **Power and sample size**:
   - a mean: `statsmodels.stats.power.tt_ind_solve_power(effect_size=mde / sd, alpha=0.05, power=0.8, ratio=1)`;
   - a proportion: `NormalIndPower().solve_power(proportion_effectsize(p0, p0 + mde), alpha=0.05, power=0.8)`.
   Plot units needed vs the MDE (`px.line(x="mde", y="n_per_arm")`): the human picks the trade-off.
3. **Duration**: units needed / units per day per arm, rounded **up to full weeks**; at least one week, ideally two (novelty).
4. **Variance reduction**: the correlation between the pre-period and in-test metric; CUPED divides the variance by about `1 - ρ²`, so the needed n too.
5. **The unit and interference**:
   - no interaction: randomize users (or accounts), analyze per user;
   - social or marketplace effects: cluster randomization (by city, by social cluster), analyzed per cluster; or a switchback (whole market alternates treatment over time slots) for supply-demand systems;
   - fewer, bigger units cost power: redo step 2 with the cluster count and the design effect `1 + (m - 1) × ICC`.
6. **The plan**, one markdown table: unit, split, primary metric, guardrails, MDE, n, duration, the stopping rule (fixed horizon, or a sequential test), segments planned in advance.
7. **An A/A test** if the pipeline is new: the same split with no change, to check SRM and that p-values are uniform.

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
