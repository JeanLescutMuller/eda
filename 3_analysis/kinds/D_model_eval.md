# D. Which model, checkpoint or training mix is better?

Comparing trained models (checkpoints, data mixes, hyperparameters, architectures) on an evaluation set. Usually: several runs per variant (seeds), every run scored on the same eval items, each item scored one or more times (samples, pass@k).

## What you'll hear

- "Which of these four data mixes should go into the next training run?"
- "Checkpoint 2 scores higher than checkpoint 1: is it really better?"
- "We trained each config with 3 seeds; which one wins?"
- "Does adding the synthetic data help?"

## Ask first

| Ask | Why |
|---|---|
| What's the eval unit: one sample, one item, one run? | the test and its n follow from it |
| Are all variants scored on the **same** items? | yes: a paired design, far more power |
| How many seeds per variant, and does any variant change more than one thing at once? | 3 seeds rarely separate variants a point apart; a variant that changes two things can't say which one helped |
| Could any eval item have leaked into the training data? | contamination inflates the variant trained on the leaking source (`G_contamination.md`) |

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| variant B beats A | the paired per-item difference's 95 % CI includes 0 |
| the gap is seed noise | the gap is larger than the spread between seeds of one variant |
| the gap comes from contaminated items | the gap holds on the clean items alone |
| the gap is in one family or difficulty level only | the per-family differences all point the same way |
| the variant trained more on a family wins on that family | the per-family gain doesn't follow the training mix |
| a broken grader or item (always 0 % or 100 %) drives it | the gap holds without those items |

## The notebooks

1. **The data's shape**: items × variants × seeds × samples, and check it's complete (every item scored by every run; `2_joins/joins.md` coverage per group).
2. **Aggregate to the right unit**: per item and variant, the pass rate over seeds and samples (`groupby(["variant", "item"]).passed.mean()`); per run, the pass rate over items.
3. **Two views, on purpose**:
   - per item (`px.box(x="variant", y="pass_rate")`): the spread between items, usually huge (± 25 points);
   - per run (`px.strip(x="variant", y="pass_rate")`, every seed shown): the seed noise, the one that decides whether a gap is real.
   They answer different questions; say which is which under each plot. A variant that looks best per run with 3 points per box proves little.
4. **The paired difference**: per item, B − A, then its mean and 95 % CI (`ttest_rel`, or a bootstrap over items). This is the headline test: every item is its own control.
5. **Seeds and items together**: when seed noise is large, a mixed model (`mixedlm("pass_rate ~ variant", groups=item)` on per-run-per-item rows) or a bootstrap that resamples items and seeds; the paired test over items alone ignores the seed noise and can be over-confident.
6. **Per segment** (family, difficulty, length): the paired difference per segment with its CI (`px.bar(error_y=...)`); a gain in one segment and a loss in another often cancel overall. Correct for the number of segments (`tests.md`).
7. **Robustness**: the headline with all items vs without contaminated, broken or duplicated items, side by side (`px.bar(barmode="group")`).
8. **Difficulty**: pass rate vs an item's complexity (`px.box` per quartile, Spearman's ρ); tells where every variant fails.

## The test

`tests.md`: paired t / Wilcoxon on per-item differences; McNemar on pass / fail per item when each item is scored once; a mixed model or a two-level bootstrap with seeds; Holm when several variants are each compared to a baseline.

## Traps

| Trap | Fix |
|---|---|
| samples treated as independent (4 samples × 400 items = "1,600 observations") | aggregate to the item first |
| per-item spread mistaken for noise on the variant's effect | the paired difference removes the items' spread |
| seed noise ignored | show every seed; with 3 seeds say how big a gap they could detect |
| an unpaired test on a paired design | `ttest_rel`, not `ttest_ind` |
| a variant that changes two things | say the gap can't be attributed to either |
| a contaminated eval | rerun on the clean items before any conclusion |
| a winner picked among many variants or segments | correct for the comparisons; the second best is often within noise |

## The answer's shape

The recommended variant, its gain over the baseline in points with a 95 % CI, on the clean items, per segment where it matters; and what would settle it if it's not conclusive (more seeds, more items, a cleaner eval), with the n needed (`tests.md`, power).
