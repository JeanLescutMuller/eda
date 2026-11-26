# D. Which group is better: vendors, teams, regions, variants?

Two or more groups compared on an outcome, without a randomized split: vendors delivering data, teams resolving tickets, regions selling, suppliers' defect rates. The groups usually differ in more than the label.

## What you'll hear

- "Vendor A's annotations have fewer errors than vendor B's: should we switch?"
- "Which region performs best?"
- "Team X closes tickets faster: what are they doing right?"
- "Is supplier 3 really worse, or just unlucky?"

## Ask first

| Ask | Why |
|---|---|
| Did the groups work on the **same items** or a different mix? | different mixes make the raw comparison unfair |
| What's the outcome per unit, and what's the unit? | the rate per item, per task, per order; rows are rarely independent |
| How were items assigned to groups? | if harder items went to one group, the gap is the assignment |
| What gap would change the decision (cost, switch, contract)? | separates significant from worth acting on |

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| group A beats B | the difference's 95 % CI includes 0 |
| the gap is the mix of items (difficulty, category, size) | the gap holds within each segment, or after standardization |
| the gap is a different version, date or tool | the "what changes with the groups" check (`1_table/table.md`) shows no confounder |
| the gap is a few extreme units (one bad batch, one bad annotator) | the gap holds without the top contributors, or on medians |
| groups are measured differently (another grader, another definition) | the same grader or definition applies to all groups |

## The notebooks

1. **The raw comparison**: the outcome per group with its 95 % CI (`px.bar(x="group", y="mean", error_y="ci", text_auto=".2f")`).
2. **The mix**: the share of each segment per group (`px.histogram(x="group", color="segment", barnorm="percent", text_auto=".0f")`). Different mixes: orange, go on.
3. **Within each segment**: the outcome per group and segment (`px.bar(x="segment", y="mean", color="group", barmode="group", error_y="ci")`). Same direction everywhere: the gap is real; reversed: Simpson's paradox.
4. **Standardize or adjust**:
   - direct standardization: each group's per-segment rates weighted by one common mix (the overall mix);
   - regression: `ols("y ~ group + segment", df)` or `logit` for a binary outcome; the group coefficient is the gap at an equal mix.
5. **Same items, paired**: if the groups scored the same items, the per-item difference and its CI (`ttest_rel`, McNemar for binary): far more power than step 1.
6. **Concentration**: the outcome per sub-unit inside each group (annotator, batch, store), sorted (`px.bar`); a gap driven by 1–2 sub-units is a local problem, not a group one.
7. **3+ groups**: one omnibus test first, then pairwise vs a reference with Holm (`tests.md`).

## The test

`tests.md`: Welch / two-proportion z for the raw gap; paired t or McNemar on shared items; regression for the adjusted gap; ANOVA / Kruskal-Wallis then Holm for 3+ groups; clustered errors or a cluster bootstrap when sub-units repeat.

## Traps

| Trap | Fix |
|---|---|
| different item mixes | stratify, standardize or adjust |
| a confounder that splits like the groups (version, date) | the TVD check; compare within the confounder |
| rows not independent (many rows per annotator or store) | aggregate, or cluster the errors |
| one bad sub-unit generalized to the group | the concentration plot |
| the best of many groups declared the winner | the omnibus test, Holm, and the CI of the winner vs the second |
| different measurement per group | the same grader or definition, or say it can't be compared |

## The answer's shape

The adjusted gap with its CI, whether it holds in each main segment, the confounder or concentration found if any, and the decision it supports (switch, keep, audit a sub-unit).
