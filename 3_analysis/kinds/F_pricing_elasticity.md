# F. How should we price? What happens if the price changes?

Estimating how demand responds to price (the elasticity), then the price that maximises what the business wants (revenue, margin, volume under a constraint).

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| demand falls as price rises (elasticity < 0) | the log-log slope's CI includes 0 or is positive after controls |
| demand is elastic (|ε| > 1): a price rise loses revenue | |ε| < 1 with its CI |
| the observed slope is confounded by demand | it changes a lot once season, segment or promotion is controlled |
| segments differ in sensitivity | the price × segment interaction is near 0 |
| the discount stole sales from other products | the substitutes' sales didn't drop during it |

## The test

The elasticity's CI from the regression (`tests.md`, regression row); a price test read as an A/B test (`C_ab_test_readout.md`), with revenue per visitor as the metric.

## Traps

| Trap | Fix |
|---|---|
| prices set by demand, read as demand set by price | controls for season and segment, or price variation from a test |
| an optimum outside the observed prices | stay within the range; propose a test beyond it |
| revenue optimised when the business cares about margin | the objective first; margin needs cost |
| a discount's lift counted without the pull-forward and cannibalisation | the weeks after, and the substitutes |
| one elasticity for everyone | per segment |

## The answer's shape

The elasticity with its CI (overall and per segment), the price that maximises the stated objective within the observed range, the expected change in revenue / margin with a range, and the price test that would confirm it.
