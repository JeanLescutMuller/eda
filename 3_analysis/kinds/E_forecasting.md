# E. What will the number be next week, month or quarter?

Forecasting one or a few time series: demand, revenue, traffic, tickets, compute. The question is a number with a range, at a horizon, for a decision (stock, staff, budget, capacity).

## What you'll hear

- "Forecast next quarter's sales."
- "How many support tickets should we staff for in December?"
- "Will we run out of capacity before the summer?"
- "Our forecast was way off last month; can you do better?"

## Ask first

| Ask | Why |
|---|---|
| The horizon and the granularity (daily for 2 weeks? monthly for a year?) | sets the model, the backtest and what "accurate" means |
| What decision uses it, and what costs more: over or under? | a stock-out and an overstock rarely cost the same; it picks the quantile to report |
| Known future events: holidays, launches, price changes, promotions | regressors, or a structural break the history can't show |
| How much history, and did anything change in it (a new product, a pipeline change, COVID)? | a break in the history makes the old part misleading |

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| the series has a trend | the STL trend component is flat compared with the residual |
| there's a weekly / yearly season | the seasonal component is small, or the autocorrelation at lag 7 / 12 isn't |
| a model beats the seasonal-naive baseline | its backtest error (MASE) is ≥ 1 |
| a break changed the level | the residuals before and after the date have the same mean |
| holidays / promotions drive the peaks | the peaks don't line up with the event calendar |

## The notebooks

1. **The series**: `px.line(df, x="ds", y="y")`; check the time index is complete (`asfreq`, gaps counted), the last period isn't partial (`1_table/time.md`), and units don't change.
2. **Decompose**: `statsmodels.tsa.seasonal.STL(y, period=7).fit()`; plot trend, season and residual (`px.line` on a long frame, `facet_row="component"`).
3. **Baselines first**: naive (last value), seasonal naive (same day last week / last year), the mean of the last k periods. Every model is judged against these.
4. **Backtest with a rolling origin**: several cut-off dates, forecast the horizon from each, keep the errors per cut-off and horizon (`sklearn.model_selection.TimeSeriesSplit`, or a loop over cut-offs). Never a random split.
5. **Models**: `statsmodels.tsa.holtwinters.ExponentialSmoothing(y, trend="add", seasonal="add", seasonal_periods=7)` (ETS); `statsmodels.tsa.statespace.sarimax.SARIMAX(y, order=…, seasonal_order=…, exog=holidays)` when there are regressors.
6. **Errors**: MAE and MASE (the error divided by the seasonal-naive error; < 1 beats it) per model, per horizon (`px.line(x="horizon", y="mase", color="model")`). MAPE only when y is far from 0: it explodes near 0 and punishes over-forecasts less than under-forecasts.
7. **Prediction intervals**: `get_forecast(h).conf_int(alpha=0.2)` (80 %) and check their coverage in the backtest: an 80 % interval should contain ~80 % of the actuals.
8. **The forecast**: history, forecast and interval in one plot (`px.line` plus `fill="tonexty"` via `add_scatter`), with the decision's threshold drawn if there is one.

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
