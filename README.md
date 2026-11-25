# eda: an exploratory-data-analysis skill for coding agents

A skill (`SKILL.md`, the Agent Skills format read by Claude Code and Codex) for the first part of any data analysis in a Jupyter notebook: load the tables, look at each one alone, then at the tables together, before any model or test.

```
# 1 Loading the data
# 2 EDA
## 2.1 Single-table analysis      per table: Overview · Single-column · Multi-columns
## 2.2 Cross-table analysis       join keys · coverage per group
```

- **Short cells, plots rather than tables**: the notebook is meant to be read and reviewed by a human, live.
- **Colours mean something**: the data in neutral colours; green, orange and red only for what a check finds.
- **Stops after each section**: the human owns the question and the hypotheses.

It builds on what the field already uses, and adds the checks they leave out when two groups are compared:

| From | What |
|---|---|
| Anthropic's [`explore-data`](https://github.com/anthropics/knowledge-work-plugins/tree/main/data/skills/explore-data) skill | the grain ("one row per what?"), column kinds, null thresholds |
| [ydata-profiling](https://github.com/ydataai/ydata-profiling) alerts | per-column flags and their default thresholds |
| [Deepchecks](https://github.com/deepchecks/deepchecks) data integrity, [Great Expectations](https://github.com/great-expectations/great_expectations), [pandera](https://github.com/unionai-oss/pandera) | hidden nulls, mixed types, compound keys, value ranges, overlap between sets |
| OpenAI's [`jupyter-notebook`](https://github.com/openai/skills/tree/main/skills/.curated/jupyter-notebook) skill | one step per cell, a note per step, runs top to bottom |
| Wickham, [R for Data Science](https://r4ds.hadley.nz/eda) | variation within a column, covariation between columns |
| added here | coverage per group, columns that change with the groups (a version, a date), joins between tables |

## Use

Claude Code: copy `SKILL.md` to `~/.claude/skills/eda/SKILL.md`. Anywhere else (a hosted notebook, a fresh machine), in the terminal:

```bash
curl -fsSL https://raw.githubusercontent.com/JeanLescutMuller/eda/main/SKILL.md -o EDA.md
```

then tell the agent: "Read EDA.md and follow it."

MIT licence.
