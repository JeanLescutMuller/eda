# E. What will the number be next week, month or quarter?

Forecasting one or a few time series: demand, revenue, traffic, tickets, compute. The question is a number with a range, at a horizon, for a decision (stock, staff, budget, capacity).

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| the series has a trend | the STL trend component is flat compared with the residual |
| there's a weekly / yearly season | the seasonal component is small, or the autocorrelation at lag 7 / 12 isn't |
| a model beats the seasonal-naive baseline | its backtest error (MASE) is ≥ 1 |
| a break changed the level | the residuals before and after the date have the same mean |
| holidays / promotions drive the peaks | the peaks don't line up with the event calendar |

## The evaluation

The backtest above, not the in-sample fit. Compare models on the same cut-offs; a paired comparison of their errors per cut-off (`tests.md`, Wilcoxon) when the human asks whether one is really better.

## Traps

| Trap | Fix |
|---|---|
| a random train/test split | rolling-origin backtest only |
| no baseline | seasonal naive first; a model must beat it |
| a partial last period read as a drop | drop or flag it |
| MAPE on a series near 0 | MAE / MASE |
| a point forecast with no range | prediction intervals, with their backtest coverage |
| a known break or future event ignored | a regressor, a dummy, or only the history after the break |
| one series forecast when the decision needs the segments | forecast per segment and check they add up (or reconcile) |

## The answer's shape

The forecast at the horizon with its 80 % (or 95 %) interval, the model and its backtest MASE against seasonal naive, the assumptions (no break, the event calendar used), and what would make it wrong.
