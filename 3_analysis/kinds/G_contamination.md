# G. Is the eval clean? Train / test overlap and leakage

Items in the evaluation set that also appear, exactly or nearly, in the training data (or a benchmark that leaked online), or features that carry the answer. A contaminated eval rewards memorization: the variant trained on the leaking source looks best.

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| no eval item is in the training data | no match on the id, the content hash, or near-duplicates |
| the overlap is concentrated in one source | the overlap's split by source matches the sources' shares |
| the contamination inflates the score | the score on clean items equals the score on all items |
| the variant ranking is driven by contamination | the ranking holds on the clean items only |
| a feature leaks the label | its importance or AUC alone is not near-perfect; it exists before the outcome |
| the split leaks across groups | the same user, session or entity is not on both sides |

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
