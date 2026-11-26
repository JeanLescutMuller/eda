# How would you measure success?

Choosing the metric before the analysis or the experiment: what number says the product, the model or the change is working. Asked in almost every product question, often first. The answer is a small set: one primary metric, a few guardrails, and how each is computed.

## What you'll hear

- "How would you measure the success of this feature?"
- "What metric should this team optimise?"
- "Our metric went up but the business didn't improve. Why?"
- "Which metric would you use for this A/B test?"

## Ask first

| Ask | Why |
|---|---|
| What's the goal, in business terms (revenue, retention, cost, quality)? | the metric is a measurable stand-in for it |
| Who acts on the metric, how often? | daily decisions need a fast, sensitive metric; strategy a slow, faithful one |
| What could go wrong if the team pushes the number up? | the guardrails, and the gaming to guard against |
| What can be measured today, and how soon after the change? | a perfect metric 6 months later is useless for a 2-week test |

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| the proxy follows the goal | across past changes, the proxy and the goal moved in different directions |
| the metric is sensitive enough to detect a realistic change | its minimum detectable effect at the available n is larger than the expected effect |
| the metric can't be gamed without hurting users | an obvious way to raise it lowers the goal (clickbait raises clicks, lowers retention) |
| the guardrails catch the likely harms | a known past failure wouldn't have moved any guardrail |
| the ratio metric is stable | its numerator and denominator are dominated by a few heavy users |

## The notebooks

1. **The metric tree**: in markdown, the goal → the primary metric → its parts (e.g. revenue = users × conversion × order value); each with its exact numerator, denominator, unit and window.
2. **Proxy vs goal**: over past periods, segments or experiments, the proxy against the goal (`px.scatter(x="proxy", y="goal", trendline="ols")`, Spearman's ρ); a proxy that doesn't track the goal is not a metric.
3. **Sensitivity**: the metric's day-to-day or user-to-user variance, and the minimum detectable effect for the expected n (`tests.md`, power); `px.line` of the daily metric with its noise band.
4. **Distribution**: `px.histogram(log_x=True)` of the per-user metric; heavy tails call for a capped (winsorised) version or a median.
5. **Ratio metrics**: a per-user ratio (clicks per session) vs a ratio of totals (all clicks / all sessions) give different answers; the variance of a ratio of totals with the delta method, or bootstrap over users.
6. **Gaming and guardrails**: for each way the primary metric could be pushed up, the guardrail that would catch it (latency, error rate, retention, complaints, cost), in a table.
7. **The final set**: one primary metric, 2–4 guardrails, 1–2 diagnostics, each with its definition and the change that would count as a win.

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
