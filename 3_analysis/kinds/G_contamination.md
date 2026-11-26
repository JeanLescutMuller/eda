# G. Is the eval clean? Train / test overlap and leakage

Items in the evaluation set that also appear, exactly or nearly, in the training data (or a benchmark that leaked online), or features that carry the answer. A contaminated eval rewards memorization: the variant trained on the leaking source looks best.

## What you'll hear

- "Our model aces the benchmark but fails on new data: why?"
- "Could the eval set have leaked into the training data?"
- "The mix with more public data scores best: is it real?"
- "This feature predicts churn almost perfectly: too good to be true?"

## Ask first

| Ask | Why |
|---|---|
| Where did each eval item come from, and each training source? | public sources and scraped data leak most |
| What makes two items "the same": an id, a text, a hash, a geometry, a near-copy? | ids differ by construction across sources; match on the content |
| Was the split random by row, or by user, time or group? | a row split leaks across rows of the same user or session |
| Which features exist at prediction time? | a feature recorded after the outcome is leakage |

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| no eval item is in the training data | no match on the id, the content hash, or near-duplicates |
| the overlap is concentrated in one source | the overlap's split by source matches the sources' shares |
| the contamination inflates the score | the score on clean items equals the score on all items |
| the variant ranking is driven by contamination | the ranking holds on the clean items only |
| a feature leaks the label | its importance or AUC alone is not near-perfect; it exists before the outcome |
| the split leaks across groups | the same user, session or entity is not on both sides |

## The notebooks

1. **Exact matches**: on the id (`eval.id.isin(train.id)`), then on a content hash of the normalized content (`hashlib.sha256(text.strip().lower().encode()).hexdigest()`), or on a tuple of exact descriptive fields when the content is structured (`merge(on=[...])`).
2. **Near-duplicates**: for text, word n-gram overlap (e.g. a shared 13-gram) or MinHash / LSH (`datasketch.MinHash`) for Jaccard similarity above ~0.8; for numbers, a nearest-neighbour distance (`sklearn.neighbors.NearestNeighbors`) with a tolerance; the count per similarity bin (`px.histogram`), and a few pairs read by eye to set the threshold.
3. **Split by source**: the contaminated items per training source and per eval segment (`px.bar(x="source", y="n", text_auto=True)`); duplicates across sources too (the same item scraped twice).
4. **The score, all vs clean**: the outcome on contaminated vs clean items per variant (`px.bar(x="variant", y="rate", color="subset", barmode="group", error_y=...)`). A much higher rate on contaminated items confirms memorization.
5. **The ranking on the clean set**: rerun the main comparison (`D_model_eval.md`) on clean items only, side by side with all items.
6. **Leaky features**: each feature's single-feature AUC (`sklearn.metrics.roc_auc_score(y, x)`), sorted; anything near 1 is suspect; check its timestamp against the outcome's.
7. **Group leakage**: the entities (users, sessions, documents) present in both splits; if any, re-split by group (`sklearn.model_selection.GroupShuffleSplit`) or by time.
8. **The clean set**: the eval without the contaminated items, pickled (`df_eval_clean.pkl`), its size and what was dropped in the markdown above.

## The test

`tests.md`: two-proportion z or Fisher's exact for the contaminated vs clean rate; the main comparison's test rerun on clean items; Holm if several variants.

## Traps

| Trap | Fix |
|---|---|
| matching on ids only | content hashes and near-duplicates |
| a near-duplicate threshold never checked | read a sample of pairs at the threshold |
| contamination reported as a count only | its effect on the score and the ranking |
| removing items from training when only the eval can change | remove them from the eval, say the eval got smaller |
| a row-level split for grouped data | a group or time split |
| a too-good feature kept | check when it's recorded |

## The answer's shape

How many eval items are contaminated (exact and near), from which sources, by how much they inflate each variant's score, whether the conclusion changes on the clean set, and the clean set to use from now on.
