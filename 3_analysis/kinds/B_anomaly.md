# B. What's unusual?

Finding the points, days or units that don't fit: a spike in a time series, a transaction unlike the others, a sensor that drifts, a user that behaves like a bot. The first job is to decide what "normal" is; the second is to check that each anomaly is real and not a data problem.

## What you'll hear

- "Is yesterday's spike normal?"
- "Find the weird transactions."
- "Which machines behave differently?"
- "Are there bots in this traffic?"

## Ask first

| Ask | Why |
|---|---|
| Anomalous compared to what: the unit's own past, its peers, a rule? | picks the method: time series residual, a peer z-score, a threshold |
| What will be done with an alert, and what does a false alarm cost? | sets the threshold; a top-20 list is often better than a cut-off |
| Are there labelled past anomalies? | then it's a classification with a precision / recall to measure (`F_threshold_tradeoff.md`) |
| Seasonality (hour, weekday, month)? | a Monday peak isn't an anomaly |

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| the spike is a data problem (a duplicate load, a unit, a backfill) | the raw rows look normal and are not duplicated (`A_data_audit.md`) |
| the spike is seasonal | it's outside the range of the same hour or weekday in past weeks |
| the anomalies share a cause (a source, a version, a region) | they're spread like the rest across every category |
| a unit is anomalous on one feature only | it's normal on that feature within its peer group |
| the anomalies are bots or tests | their timing, frequency and ids look like real users |

## The notebooks

1. **Look first**: `px.line` of the series with its rolling median and band (`rolling(28).median()`, ± 3 × rolling MAD), or `px.box` per peer group for units.
2. **Robust z-score**: `(x − median) / (1.4826 × MAD)` per peer group (`groupby(...).transform`); |z| > 3.5 flagged (red), shown as `px.scatter(color="flag")` with green / red.
3. **IQR rule** on one variable: beyond Q1 − 1.5 IQR or Q3 + 1.5 IQR; the same plot.
4. **Time series with seasonality**: `statsmodels.tsa.seasonal.STL(s, period=7).fit()`, then the robust z-score of the residual; plot trend, seasonal and residual (`px.line`, one facet each) and the flagged points.
5. **Many dimensions at once** (only when single-feature checks miss it): `IsolationForest(random_state=0).fit(X_scaled)`, the `score_samples` as a histogram, the top 20 listed with their feature values so the human can see why.
6. **Profile the anomalies**: their share per source, version, region and date vs the rest (`px.bar(barmode="group", text_auto=".0%")`); a concentration is a cause.
7. **Check each top anomaly by hand**: the raw rows behind the 5 largest, in a table.

## The test

`tests.md`, rarely: an anomaly is a ranking, not a test. To check that anomalies concentrate in a category: chi-square or Fisher's exact on anomaly × category. To check a single point: its position against the same hour or weekday's historical distribution (a percentile).

## Traps

| Trap | Fix |
|---|---|
| mean and standard deviation pulled by the outliers themselves | median and MAD |
| seasonality flagged as anomalies | compare to the same hour or weekday, or STL |
| an isolation forest when one feature explains it | single-feature checks first; explain each flag |
| a data error reported as an event | check the raw rows of the top anomalies |
| a fixed threshold with no cost in mind | a ranked list, or a threshold set by the cost of a miss vs a false alarm |
| anomalies deleted | flag them, never delete; the human decides |

## The answer's shape

The method and threshold in one line, the number of anomalies (n and %), the top ones with what makes each unusual, where they concentrate (a source, a date, a version), which are data problems and which are real, and the next check or action.
