# D. Evaluate LLM outputs: which model or prompt is better, and can the grader be trusted?

Outputs of one or more models (or prompts, or system versions) on a set of prompts, scored by a human, a rule, a test suite or an LLM judge. Two questions hide in one: is the grader reliable, and given that, which system is better.

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| the judge agrees with humans | Cohen's κ (or Krippendorff's α) on the labeled subset is high (> 0.6) and its confusion matrix is balanced |
| the judge prefers the first position | swapping A and B flips its verdict at the expected rate (wins hold after swapping) |
| the judge prefers longer answers | the win rate doesn't rise with the length difference |
| the grader is flaky | rescoring the same output gives the same score |
| B beats A | the per-prompt paired difference's CI includes 0 |
| the gain is in one task type only | per-category differences point the same way |

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
