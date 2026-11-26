# D. Is the new ranking or recommender better?

A search engine, feed or recommender returns an ordered list; a new version is compared to the current one, offline on logged or labeled data, or online on users' clicks.

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| the new ranker ranks relevant items higher | the per-query NDCG@k difference's CI includes 0 |
| the offline gain carries online | the online CTR / conversion per user doesn't move |
| clicks reflect position, not relevance | CTR at the same rank is the same for both rankers (position-normalized) |
| the gain is in head queries only | the per-bucket (head / torso / tail queries) differences point the same way |
| more clicks, worse outcomes (clickbait) | downstream metrics (purchases, dwell, returns) move with the clicks |

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
