# D. Is the new ranking or recommender better?

A search engine, feed or recommender returns an ordered list; a new version is compared to the current one, offline on logged or labeled data, or online on users' clicks.

## What you'll hear

- "The new model has a higher NDCG offline: will it do better in production?"
- "CTR went up after the ranking change: is it the model?"
- "How would you evaluate a recommendation system?"
- "Users only click the top results: how do we learn anything from clicks?"

## Ask first

| Ask | Why |
|---|---|
| What are the labels: human relevance grades, clicks, purchases, dwell time? | clicks are biased by position; grades are not but cost more |
| Offline or online, and how did the old ranker produce the logs? | logs only show what the old ranker displayed: an offline gain on them is biased toward the old ranker |
| Which k matters (the first screen, the top 3)? | metrics@k must match what users see |
| What's the unit: a query, a session, a user? | the test is per query or per user, not per impression |

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| the new ranker ranks relevant items higher | the per-query NDCG@k difference's CI includes 0 |
| the offline gain carries online | the online CTR / conversion per user doesn't move |
| clicks reflect position, not relevance | CTR at the same rank is the same for both rankers (position-normalized) |
| the gain is in head queries only | the per-bucket (head / torso / tail queries) differences point the same way |
| more clicks, worse outcomes (clickbait) | downstream metrics (purchases, dwell, returns) move with the clicks |

## The notebooks

1. **Offline metrics, per query**: NDCG@k (`sklearn.metrics.ndcg_score`), MRR (1 / rank of the first relevant item), recall@k, precision@k; one value per query per ranker, then the mean with its 95 % CI (`px.bar(error_y=...)`).
2. **The paired difference per query**: new − old, mean and CI (paired t / Wilcoxon, or a bootstrap over queries); the distribution (`px.histogram`): how many queries got worse.
3. **Per bucket**: head / torso / tail queries, or by category; the paired difference per bucket with CIs.
4. **Position bias**: CTR per rank (`px.line(x="rank", y="ctr")`): it always falls with rank. Click-based offline metrics need a correction: inverse propensity weighting by the click probability per rank (estimated from randomized swaps, or a click model), or human labels instead.
5. **Logging bias**: items the old ranker never showed have no clicks; counterfactual estimators (IPS on logged propensities) or a randomized slice of traffic are the fix; otherwise, say the offline result favours the old ranker.
6. **Online**: an A/B test per user (`C_ab_test_readout.md`) on CTR, conversion and a downstream guardrail; or **interleaving** (team-draft: both rankers' results mixed in one list, the winner per query is the one whose results get more clicks) — far more sensitive, needing 10–100× fewer queries than an A/B test.
7. **Diversity and coverage** as guardrails: the share of catalogue shown, the share of the top 1 % items in recommendations.

## The test

`tests.md`: paired tests over queries for offline metrics; a bootstrap over queries for NDCG; a binomial test of the interleaving win share against 50 % (ties excluded and reported); the A/B readout per user online.

## Traps

| Trap | Fix |
|---|---|
| impressions counted as independent | aggregate per query or per user |
| click metrics without position correction | IPS or human relevance labels |
| offline logs from the old ranker | counterfactual estimates, or a randomized slice; say the bias |
| a mean gain hiding many worse queries | the per-query difference distribution |
| more clicks read as better | downstream outcomes as guardrails |
| k not matching the UI | the k users actually see |

## The answer's shape

The offline gain per query with its CI and the share of queries worse, whether position and logging bias were handled, the online result (A/B or interleaving) or the test that would confirm it, and the guardrails.
