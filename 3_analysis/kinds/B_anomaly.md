# B. What's unusual?

Finding the points, days or units that don't fit: a spike in a time series, a transaction unlike the others, a sensor that drifts, a user that behaves like a bot. The first job is to decide what "normal" is; the second is to check that each anomaly is real and not a data problem.

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| the spike is a data problem (a duplicate load, a unit, a backfill) | the raw rows look normal and are not duplicated (`A_data_audit.md`) |
| the spike is seasonal | it's outside the range of the same hour or weekday in past weeks |
| the anomalies share a cause (a source, a version, a region) | they're spread like the rest across every category |
| a unit is anomalous on one feature only | it's normal on that feature within its peer group |
| the anomalies are bots or tests | their timing, frequency and ids look like real users |

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
