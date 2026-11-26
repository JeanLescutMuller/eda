# F. Is it worth it? How big is this?

Sizing an opportunity, a problem or an investment: how many users, how much revenue, how much a fix would save, and whether it pays back. Often with little data: the method is a transparent chain of assumptions, each with a range.

## What you'll hear

- "How much revenue are we losing to this bug?"
- "Is it worth building feature X?"
- "How big is the market for this product?"
- "If we fixed the parser, how much more usable data would we get?"

## Ask first

| Ask | Why |
|---|---|
| What decision does the size feed (go / no-go, priority vs other projects)? | the precision needed: an order of magnitude often decides it |
| The cost side: build, run, opportunity cost | ROI needs both sides |
| Incremental or total: what changes because of this, vs what would happen anyway? | total is almost always the wrong number |
| Which inputs are measured in the data, which are assumptions? | measured inputs anchor the chain; assumptions get ranges |

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| the opportunity is large enough to matter | the upper end of the range is below the decision's bar |
| the top-down and bottom-up estimates agree | they differ by more than their ranges |
| one assumption drives the answer | the tornado shows the bars of similar width |
| the effect is incremental | users who'd be affected already get the value another way (a workaround, a substitute) |

## The notebooks

1. **The chain**: a markdown table, one row per factor (users affected × share who'd use it × value per use × uses per year), each with a low, a base and a high value and its source (measured here, benchmark, assumption).
2. **Measure what you can**: each measured factor computed from the data in its own cell (`df.groupby(...).agg(...)`), with its CI where it applies (`tests.md`).
3. **Bottom-up**: the product of the chain at low / base / high.
4. **Top-down**: from a total (all revenue, all users, the market) times the share concerned; compared with bottom-up in one `px.bar`.
5. **Monte Carlo**: draw each factor from its range (`numpy.random.default_rng().uniform(low, high, 10_000)` or a triangular / lognormal), multiply, `px.histogram` of the result with the 10th, 50th and 90th percentiles marked.
6. **Tornado**: the result with each factor at its low and high while the others stay at base, sorted by the swing, as a horizontal `px.bar` with `base=`; the widest bar is what to measure next.
7. **ROI and payback**: value per period minus cost, cumulated over time (`px.line`), the break-even month marked; at the 10th / 50th / 90th percentile values.

## The test

Usually none: the ranges are the uncertainty. A test only for a measured factor (a rate, a difference) feeding the chain.

## Traps

| Trap | Fix |
|---|---|
| total size reported as the gain | incremental only: what changes because of the action |
| a single point estimate | low / base / high, or the Monte Carlo percentiles |
| false precision (three significant digits on guesses) | round to what the inputs justify |
| hidden assumptions | every factor in the table, with its source |
| no cost side | build, run and opportunity costs in the ROI |
| no cross-check | top-down vs bottom-up |

## The answer's shape

The size as a range (10th–90th percentile) with the base case, the ROI or payback at those values, the assumption that drives it most (the tornado's top bar) and how to measure it, and the decision it supports.
