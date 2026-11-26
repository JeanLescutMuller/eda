# How would you measure success?

Choosing the metric before the analysis or the experiment: what number says the product, the model or the change is working. Asked in almost every product question, often first. The answer is a small set: one primary metric, a few guardrails, and how each is computed.

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| the proxy follows the goal | across past changes, the proxy and the goal moved in different directions |
| the metric is sensitive enough to detect a realistic change | its minimum detectable effect at the available n is larger than the expected effect |
| the metric can't be gamed without hurting users | an obvious way to raise it lowers the goal (clickbait raises clicks, lowers retention) |
| the guardrails catch the likely harms | a known past failure wouldn't have moved any guardrail |
| the ratio metric is stable | its numerator and denominator are dominated by a few heavy users |

## The test

`tests.md`: power and minimum detectable effect for each metric; for ratio metrics, the delta-method variance or a user-level bootstrap before any test; Spearman between proxy and goal across past periods.

## Traps

| Trap | Fix |
|---|---|
| a vanity metric (page views, sign-ups) | tie it to the goal through the metric tree |
| a metric too slow to move in a test | a faster leading metric, validated against the goal |
| a ratio of totals tested as if per-user | delta method or a user-level bootstrap |
| heavy users drive the metric | cap it, or use a median |
| many primary metrics | one primary; the rest are guardrails or diagnostics |
| no guardrail for the obvious harm | list the ways to game it, and a guardrail for each |

## The answer's shape

One primary metric with its exact definition and why it follows the goal; 2–4 guardrails, each tied to a harm; the minimum detectable effect at the expected n; and what number would count as success.
