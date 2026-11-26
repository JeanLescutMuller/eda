# D. Evaluate LLM outputs: which model or prompt is better, and can the grader be trusted?

Outputs of one or more models (or prompts, or system versions) on a set of prompts, scored by a human, a rule, a test suite or an LLM judge. Two questions hide in one: is the grader reliable, and given that, which system is better.

## What you'll hear

- "Model B wins 58 % of the head-to-head comparisons: is it better?"
- "We use GPT-as-a-judge: can we trust its scores?"
- "Pass@1 went from 41 % to 45 %: real?"
- "The new prompt scores higher on our rubric: ship it?"

## Ask first

| Ask | Why |
|---|---|
| Who or what grades: a human, a rule, unit tests, an LLM judge? | a judge's biases and noise come before any model comparison |
| Is there a human-labeled subset to check the judge against? | without it, the judge's accuracy is unknown |
| Are all systems scored on the **same prompts**, with how many samples each? | per-prompt pairing; pass@k needs n samples per prompt |
| Did the rubric, the judge model or its prompt change between runs? | rubric drift makes runs incomparable |

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| the judge agrees with humans | Cohen's κ (or Krippendorff's α) on the labeled subset is high (> 0.6) and its confusion matrix is balanced |
| the judge prefers the first position | swapping A and B flips its verdict at the expected rate (wins hold after swapping) |
| the judge prefers longer answers | the win rate doesn't rise with the length difference |
| the grader is flaky | rescoring the same output gives the same score |
| B beats A | the per-prompt paired difference's CI includes 0 |
| the gain is in one task type only | per-category differences point the same way |

## The notebooks

1. **The grader first**:
   - agreement with humans: `sklearn.metrics.cohen_kappa_score(human, judge)`, and `px.imshow(pd.crosstab(human, judge), text_auto=True)`;
   - position bias: each pair judged twice, A–B and B–A; the share of inconsistent verdicts;
   - length bias: win rate per bin of length difference (`px.bar`), Spearman's ρ;
   - flakiness: the same outputs rescored, the share of changed scores; a self-check on known-good and known-bad answers.
2. **Per prompt**: the score per system and prompt, averaged over samples; check completeness (every system on every prompt).
3. **pass@k**: with n samples per prompt and c correct, the unbiased estimator `1 - comb(n - c, k) / comb(n, k)` per prompt, then the mean over prompts; never `1 - (1 - p̂)^k` from the pooled rate.
4. **The paired comparison**: per prompt, B − A, its mean and 95 % CI (paired t / Wilcoxon, or a bootstrap over prompts); for pass / fail with one sample, McNemar; for pairwise preferences, the win rate with a Wilson CI, ties reported separately (or a Bradley-Terry model across many systems).
5. **Per category** (task type, language, difficulty): the paired difference with its CI, Holm across categories (`px.bar(error_y=...)`).
6. **Failure modes**: a sample of the prompts where the systems disagree most, read by eye; refusals, truncations, format errors counted separately from wrong answers.
7. **Robustness**: the result under a second grader (human subset, another judge, swapped positions), side by side.

## The test

`tests.md`: paired tests over prompts; McNemar for paired pass / fail; binomial / Wilson for a win rate against 50 %; bootstrap over prompts (keeping all samples of a prompt together) for pass@k differences; Holm across categories.

## Traps

| Trap | Fix |
|---|---|
| an unvalidated judge | κ against a human subset first |
| position or length bias | swap positions; control for length |
| samples counted as independent prompts | aggregate to the prompt, bootstrap prompts |
| a biased pass@k | the unbiased estimator per prompt |
| rubric or judge drift between runs | rescore all systems with the same rubric and judge |
| format failures counted as wrong answers | count them apart; they're often a cheap fix |
| ties dropped silently | report them; they change the win rate |

## The answer's shape

The grader's reliability (κ, biases found), the better system's gain on paired prompts with its CI, where it wins and loses (categories, failure modes), and what would make the eval trustworthy if it isn't yet.
