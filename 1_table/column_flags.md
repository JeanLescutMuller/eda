# Without ydata-profiling: the flags by hand

One cell computes them (`df.isna().mean()`, `df.nunique()`, `df.describe()`, `df.dtypes`); a second cell is the flags table of `table.md` with the ones that fire. No plot per column.

| Flag | Rule | Status |
|---|---|---|
| nulls | share > 1 % orange, > 20 % red | orange / red |
| hidden nulls | "", "NA", "null", −1, 0, 999 where they can't be real | orange |
| mixed types | numbers and strings in one column | orange |
| constant | one distinct value | red |
| high cardinality | a category with > 50 values | orange |
| imbalance | one value far more common than the others, in a category of 3+ values | orange |
| skew | max more than 100× the median: use log scales or medians | orange |
| impossible values | a rate outside [0, 1], a negative count or duration, a date in the future | red |
| spelling variants | the same category written two ways ("GPT-4" / "gpt4") | orange |
| numeric correlation | above 0.9 between two numbers | orange |
