# eda: a data-analysis skill for coding agents

A skill (the Agent Skills format read by Claude Code and Codex, or any agent told to read a file) for data analysis in Jupyter notebooks, from the raw files to the answer: load, check each table, join, then the analysis the question calls for. Despite the name, it covers the whole analysis, not only the exploration.

```
00_loading           load, show raw, fix types, pickle
1N_eda_<table>       one per table: ydata profile, key, duplicates, flags
20_joins             keys, rows duplicated or lost, coverage per group, overlap
3N_<question>        one per question: hypotheses, plots, tests
40_report            the answer, the numbers, the recommendation
```

- **One notebook per step**: the human reads one while the agent writes the next; nothing they're reading changes under them.
- **Short, plain code**: Plotly Express, one call per plot, numbers on the plots, a good / warning / bad line under each; green, orange and red only for what a check finds.
- **The human decides**: the agent stops after each step, proposes cleaning rules instead of applying them, and lists hypotheses with what would disprove them.

## Files: a base, and sub-files read only when needed

`SKILL.md` is the base, always read; its last table says which sub-file to read when.

```
SKILL.md                       the pipeline, the rules, which file when
notebook_io.md                 append, run and save one cell at a time
0_load/       load.md · json.md · big_or_other.md
1_table/      table.md · column_flags.md · text.md · time.md · events.md · geo.md · graph.md
2_joins/      joins.md
3_analysis/   analysis.md (the first 5 minutes, question → hypotheses, which kind) · tests.md (which test when)
  kinds/      A  is the data right?    data_audit
              B  what happened?        metric_drop · segmentation · funnel · cohort_retention · anomaly
              C  did X cause Y?        ab_test_readout · experiment_design · causal_observational
              D  which is better?      compare_groups · model_eval · llm_eval · ranking_eval
              E  what will happen?     forecasting · propensity · quick_model
              F  what should we do?    pricing_elasticity · allocation · threshold_tradeoff · sizing_roi
              G  is the world the same? distribution_shift · selection_bias · contamination
              metric_design            how to measure success
4_report/     report.md
```

Every file in `kinds/` has the same sections: what you'll hear, what to ask first, the hypotheses to list with what disproves each, the notebooks, the test, the traps, the answer's shape.

## Use

Claude Code: clone it as a skill folder.

```bash
git clone https://github.com/JeanLescutMuller/eda ~/.claude/skills/eda
```

Anywhere else (a hosted notebook, a fresh machine): fetch the base file, and the agent fetches the sub-files it needs from the same place.

```bash
curl -fsSL https://raw.githubusercontent.com/JeanLescutMuller/eda/main/SKILL.md -o EDA.md
```

then tell the agent: "Read EDA.md and follow it."

## Where it comes from

| From | What |
|---|---|
| Anthropic's [`explore-data`](https://github.com/anthropics/knowledge-work-plugins/tree/main/data/skills/explore-data) skill | the grain ("one row per what?"), column kinds, null thresholds |
| [ydata-profiling](https://github.com/ydataai/ydata-profiling) | the per-table report and its alerts' default thresholds |
| [Deepchecks](https://github.com/deepchecks/deepchecks) data integrity and train-test suites, [Great Expectations](https://github.com/great-expectations/great_expectations), [pandera](https://github.com/unionai-oss/pandera) | hidden nulls, mixed types, compound keys, value ranges, overlap between sets |
| [Cleanlab](https://github.com/cleanlab/cleanlab) | underperforming groups as a data problem |
| OpenAI's [`jupyter-notebook`](https://github.com/openai/skills/tree/main/skills/.curated/jupyter-notebook) skill | one step per cell, a note per step |
| Wickham, [R for Data Science](https://r4ds.hadley.nz/eda) | variation within a column, covariation between columns |
| Kohavi, Tang, Xu, *Trustworthy Online Controlled Experiments* | A/B readouts: sample-ratio mismatch, ratio metrics, guardrails, peeking |
| Hyndman, Athanasopoulos, [*Forecasting: Principles and Practice*](https://otexts.com/fpp3/) | baselines first, time-based backtests, MASE |
| added here | coverage per group, columns that change with the groups, rows duplicated by joins, one notebook per step, the question → hypothesis → test routine and the kinds of analysis |

MIT licence.
