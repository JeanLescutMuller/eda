# 40_report: the answer

The last step, or whenever the human says "summarise" or "what do you recommend". Written twice, the same text: in the chat first (the human reads it now), then as markdown cells of `40_report.ipynb` (the record). No new analysis here: every number comes from a cell already run, named by notebook and cell.

## The order

1. **The answer, first, in one or two sentences**, tied to the decision: "Ship variant B: +1.8 points of conversion over control (95 % CI +0.4 to +3.2), no guardrail moved." If there's no clear answer, say that first: "No variant is clearly better: every gap is within ± 2 points of noise."
2. **The findings**, as a table, most important first:

   | Finding | Status | Numbers | Where |
   |---|---|---|---|
   | 12 % of test users also appear in the training period | 🔴 bad | 1,450 of 12,000 users, 90 % from one region | `20_joins`, cell 7 |
   | variant B beats control on the clean users | 🟢 good | +1.8 points, CI +0.4 to +3.2, p = 0.01 | `31_readout`, cell 5 |
   | the gain is in two segments only | 🟠 warning | +5 and +4 points; −1 to +1 elsewhere | `31_readout`, cell 7 |

   Status: 🟢 good, 🟠 warning, 🔴 bad, the same as the notebooks' lines. Every number with its unit, its statistic (mean, median, share) and, for an effect, its CI.
3. **What was excluded or changed, and why**: each cleaning rule and exclusion, with how many rows it removed (n and %), and the headline with and without it if it moved.
4. **Caveats**: what the data can't say (too few runs, a group confounded with a date, a short window, a segment too small), each in one line, with how much it could change the answer.
5. **The recommendation**: the action, tied to the decision written in the first cell of `3_analysis/analysis.md`'s notebooks; its expected effect with a range; the risk if it's wrong; the guardrail to watch.
6. **What would settle what's open**: the next check, experiment or data, with the size needed (more runs, n users, k weeks; `3_analysis/tests.md`, power).

## Rules

| Rule | Detail |
|---|---|
| Answer first | the reader may stop after the first sentence |
| Only numbers already computed | each with the notebook and cell it came from; no new number in the report |
| Effect, CI, then p | never a p-value alone; "not significant" always with the CI's width |
| Plain words | no jargon the decision-maker wouldn't use; a test named once, in the table |
| Known vs guessed | "the data shows" vs "the most likely explanation", never mixed in one sentence |
| Short | the chat version fits on one screen; the detail lives in the notebooks |

## In the notebook

`40_report.ipynb`: markdown cells only, in the order above; if one plot carries the answer, the one `px` call that draws it from a pickle, so the report stands alone. Then stop.
