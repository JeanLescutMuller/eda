# 3N_<question>: the analysis

One notebook per question (`30_<name>`, `31_<name>`…), named as the human says. The human owns the hypotheses; your job is to make each one testable, test it, and say what the numbers do and don't show.

## Question → hypotheses → checks

1. **Deliverables first**: the first cell after the setup gives the number the task asks for, as a placeholder, filled in as the analysis goes. Never let the answer wait for the end.
2. **List the hypotheses** in a markdown table, each with the check that would prove it wrong:

   | # | Hypothesis | Disproved if | Notebook cell |
   |---|---|---|---|
   | H1 | mix B passes more than mix A | the paired difference per item has a 95 % CI including 0 | 4 |
   | H2 | the gap comes from the item mix, not the data mix | the gap holds within each family | 6 |

   Include the boring ones: a bug, a confounder, a change of version or date, chance.
3. **Gate 0, for each hypothesis: is it a bug?** If you can point at the mechanical cause (a unit, a parser, a join, a version change), state it and size it; don't test it. A test answers "could chance explain this gap?", not "what caused it?".
4. **Plot before testing**: one plot that shows the gap, at the unit of analysis, with its spread.
5. **Test** with `tests.md`: the outcome's type and the design pick the test; report the effect size and its 95 % CI with the p-value, always.
6. **Split it**: a gap that holds overall must be checked within each main segment (Simpson's paradox: the mix moved, not the rate), and a gap found in one segment among many is a lead, not a conclusion.
7. **Say what it means for the decision**, in one line under the last plot, the verdict in its colour (`SKILL.md`, "Under each plot").

## Which kind of analysis this is

Read the one file that matches the question (sometimes two). The shape decides, not the domain.

| Shape | What you'll hear | Read |
|---|---|---|
| **A. Is the data right?** | "something looks off", "we changed the pipeline", "can we trust this?" | `kinds/A_data_audit.md` |
| **B. What happened?** | "the metric dropped, why?" | `kinds/B_metric_drop.md` |
| | "who are our users?", "which customers are different?" | `kinds/B_segmentation.md` |
| | "where do users drop off?" | `kinds/B_funnel.md` |
| | "are new users sticking?", "how does retention look?" | `kinds/B_cohort_retention.md` |
| | "find the weird ones", "is this spike normal?" | `kinds/B_anomaly.md` |
| **C. Did X cause Y?** | "read this A/B test", "did the launch work?" | `kinds/C_ab_test_readout.md` |
| | "how would you test this?", "how long should it run?" | `kinds/C_experiment_design.md` |
| | "we can't run a test, but did it work?" | `kinds/C_causal_observational.md` |
| **D. Which is better?** | "vendor A or B?", "which group performs better?" | `kinds/D_compare_groups.md` |
| | "which checkpoint, mix or model is better?" | `kinds/D_model_eval.md` |
| | "evaluate these LLM outputs", "is the judge reliable?" | `kinds/D_llm_eval.md` |
| | "is the new ranking or recommender better?" | `kinds/D_ranking_eval.md` |
| **E. What will happen?** | "forecast next quarter" | `kinds/E_forecasting.md` |
| | "who will churn, convert, buy?" | `kinds/E_propensity.md` |
| | "build a quick model", "can we predict X?" | `kinds/E_quick_model.md` |
| **F. What should we do?** | "how should we price?", "what if we raise prices?" | `kinds/F_pricing_elasticity.md` |
| | "where should the budget, data or effort go?" | `kinds/F_allocation.md` |
| | "where should the cut-off be?" | `kinds/F_threshold_tradeoff.md` |
| | "is it worth it?", "how big is this?" | `kinds/F_sizing_roi.md` |
| **G. Is the world the same?** | "will this hold in production?", "train vs eval?" | `kinds/G_distribution_shift.md` |
| | "who's missing from this data?" | `kinds/G_selection_bias.md` |
| | "is the eval clean?", "do train and test overlap?" | `kinds/G_contamination.md` |
| **Any** | "how would you measure success?", "which metric?" | `kinds/metric_design.md` |
