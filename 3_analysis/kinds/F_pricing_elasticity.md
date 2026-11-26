# F. How should we price? What happens if the price changes?

Estimating how demand responds to price (the elasticity), then the price that maximises what the business wants (revenue, margin, volume under a constraint).

## What you'll hear

- "Should we raise the price by 10 %?"
- "What's the optimal price for this product?"
- "Did the discount pay for itself?"
- "How price-sensitive are our customers, and does it differ by segment?"

## Ask first

| Ask | Why |
|---|---|
| The objective: revenue, margin, volume, market share? | the optimal price differs for each; margin needs the unit cost |
| Where does price variation come from in the data (tests, promotions, regions, list changes)? | without variation that isn't driven by demand, the elasticity can't be estimated |
| Competitors' prices, substitutes in the own range? | cannibalisation and competitor moves change the response |
| Short-term or long-term response? | a discount pulls purchases forward; the long-run effect is smaller |

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| demand falls as price rises (elasticity < 0) | the log-log slope's CI includes 0 or is positive after controls |
| demand is elastic (|ε| > 1): a price rise loses revenue | |ε| < 1 with its CI |
| the observed slope is confounded by demand | it changes a lot once season, segment or promotion is controlled |
| segments differ in sensitivity | the price × segment interaction is near 0 |
| the discount stole sales from other products | the substitutes' sales didn't drop during it |

## The notebooks

1. **Price and quantity over time**: `px.line` of both per product, promotions and price changes marked; check that price actually varies.
2. **Scatter**: `px.scatter(x="price", y="quantity", log_x=True, log_y=True, color="segment", trendline="ols")`.
3. **Log-log regression**: `smf.ols("np.log(quantity) ~ np.log(price) + C(month) + C(store) + promo", df).fit()`; the coefficient on log price is the elasticity, with its CI.
4. **Confounding**: compare the slope without and with controls. Prices set high when demand is high (peak season) bias the naive slope toward 0 or positive. Better variation: a price test, a regional rollout, an instrument (a cost shock) when the human asks for it.
5. **Per segment**: an interaction term (`np.log(price):C(segment)`), plotted as one elasticity per segment with CI (`px.bar(error_y=...)`).
6. **Cannibalisation**: sales of substitutes during the price change (`px.line`), or their price as a regressor.
7. **The curve**: predicted quantity over a price grid, then revenue `p × q(p)` and margin `(p − cost) × q(p)` (`px.line`, one line each), the current price marked; the optimum only within the range of prices observed.
8. **Pull-forward**: sales in the weeks after a promotion vs before; a dip after means the lift is partly borrowed.

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
