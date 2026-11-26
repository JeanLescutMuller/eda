# B. Are users sticking?

Users grouped by when they started (the cohort: signup week, first purchase month), then followed over time: what share is still active after 1, 2, … periods. Separates "new users are worse" from "everyone is leaving" from "the product changed for everyone at a date".

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| newer cohorts retain worse | the curves of recent cohorts lie on the older ones |
| a change at a calendar date hit every cohort | the drop shows by cohort age, not by calendar date |
| the acquisition mix changed (a channel, a country) | the gap holds within each channel |
| retention flattens (a loyal core) | the curve keeps falling at the same rate |
| it's seasonality | cohorts of the same month last year look alike |

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
