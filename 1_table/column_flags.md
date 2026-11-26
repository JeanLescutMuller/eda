# Per-column checks without ydata-profiling

The fallback when `ydata-profiling` won't install or run. The same checks, by hand: one plot per kind of column, the flags in the plot's title.

## Each column's kind

A markdown line per column, from Anthropic's `explore-data`: **identifier**, **category** (≤ ~50 values), **number**, **time**, **boolean**, **text**.

## One plot per column, by kind

| Kind | Plot |
|---|---|
| category, boolean | `px.histogram(df, y=col, text_auto=True).update_yaxes(categoryorder="total ascending")` |
| number | `px.histogram(df, x=col, log_x=…)`, log when the max is 100× the median |
| time | `px.histogram(df, x=col)`, binned per day or per hour |
| identifier | `px.histogram(df[col].value_counts())`: rows per value; a steady count is the expected shape |
| text | `px.histogram(df[col].str.len())`: lengths |

Several columns of one kind in one cell: `display(px.…, px.…)`.

## Flags

ydata-profiling's alerts and their default thresholds, and Deepchecks' Data Integrity checks:

| Flag | Rule | Status |
|---|---|---|
| nulls | share > 1 % orange, > 20 % red (ydata MISSING) | orange / red |
| hidden nulls | placeholders where they can't be real: "", "NA", "null", −1, 0, 999 (Deepchecks MixedNulls) | orange |
| mixed types | numbers and strings in one column (Deepchecks MixedDataTypes) | orange |
| constant | one distinct value (ydata CONSTANT) | red |
| high cardinality | a category with > 50 values (ydata HIGH_CARDINALITY) | orange |
| imbalance | one value far more common than the others, in a category of 3+ values (ydata IMBALANCE > 0.5) | orange |
| skew | skewness > 20 in absolute value (ydata SKEWED): use log scales or medians | orange |
| zeros | share of zeros > 10 % in a number (ydata ZEROS) | orange |
| outliers | beyond 1.5 × IQR, share > 1 %; show them, never delete them | orange |
| impossible values | outside the column's domain: a rate outside [0, 1], a negative duration or count, a date in the future (Great Expectations `expect_column_values_to_be_between`) | red |
| spelling variants | the same category written two ways ("GPT-4" / "gpt4") (Deepchecks StringMismatch) | orange |
| numeric correlation | above 0.9 between two numbers (ydata HIGH_CORRELATION): `px.imshow(df.corr(numeric_only=True), text_auto=".2f")` if 3+ numbers | orange |
