# B. Are users sticking?

Users grouped by when they started (the cohort: signup week, first purchase month), then followed over time: what share is still active after 1, 2, … periods. Separates "new users are worse" from "everyone is leaving" from "the product changed for everyone at a date".

## What you'll hear

- "How does retention look?"
- "Are the users we acquired this quarter worse than before?"
- "Did the new onboarding improve week-4 retention?"
- "What's our churn rate?"

## Ask first

| Ask | Why |
|---|---|
| What counts as active (a login, a purchase, a key action)? | a login-based curve can stay flat while real use falls |
| Cohort by what (signup date, first purchase, a channel) and at what period (day, week, month)? | the grid of the triangle |
| Retention or churn, and is coming back after a gap counted? | "active in period n" vs "active in every period up to n" give different curves |
| Is the last cohort complete? | young cohorts have no late periods yet: never compare them there |

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| newer cohorts retain worse | the curves of recent cohorts lie on the older ones |
| a change at a calendar date hit every cohort | the drop shows by cohort age, not by calendar date |
| the acquisition mix changed (a channel, a country) | the gap holds within each channel |
| retention flattens (a loyal core) | the curve keeps falling at the same rate |
| it's seasonality | cohorts of the same month last year look alike |

## The notebooks

1. **Cohort and age per activity**: `cohort = first activity period`, `age = period − cohort` (in weeks or months), one row per user and active period.
2. **The triangle**: active users / cohort size per cohort × age, `px.imshow(df_tri, text_auto=".0%", color_continuous_scale="Blues")`, cohorts as rows.
3. **The curves**: `px.line(df_tri_long, x="age", y="retention", color="cohort")` for a few cohorts (the oldest, middle and newest, or grouped by quarter), with the cohort sizes in the legend.
4. **By calendar date**: the same data with the calendar period on x; a drop on one date across every cohort is a product or measurement change, not a cohort effect.
5. **By acquisition channel or segment**: retention at a fixed age (week 4) per channel and cohort (`px.line(color="channel")`); a worse total from a worse mix.
6. **Churn as time-to-event** (optional): with `lifelines`, `KaplanMeierFitter` per group and `logrank_test` between two; handles users still active (censored) properly.
7. **The value**: retention × revenue per active user per age, summed: lifetime value per cohort, if the question is about money.

## The test

`tests.md`: retention at a fixed age between two cohorts groups as a two-proportion z; a log-rank test for whole survival curves; aggregate to one row per user before testing.

## Traps

| Trap | Fix |
|---|---|
| young cohorts compared at ages they haven't reached | compare only at ages every cohort has |
| a calendar effect read as a cohort effect | the calendar-date view |
| the cohort size not shown | sizes in the legend; small cohorts are noisy |
| mix change read as worse users | retention per channel |
| "active" defined by a trivial event | use the action that matters |
| a churn rate from users still active treated as churned | survival analysis, censoring |

## The answer's shape

The retention at the ages that matter (week 1, 4, 12) for recent vs older cohorts, with CIs; whether a gap is by cohort, by calendar date or by mix; where the curve flattens; and what it means in users or revenue.
