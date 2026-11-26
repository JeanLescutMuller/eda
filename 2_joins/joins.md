# 20_joins: the tables together

Load the latest pickles: a table's `_clean` pickle when the human had it cleaned in its `1N` notebook, else the one from `00`. Every check below is one cell, plus its good / warning / bad line.

## The cells, in order

1. **Join keys**: for each pair of tables sharing a column, a bar of the keys found in the left only / both / right only (`merge(how="outer", indicator=True)`): orphans red.
2. **Rows each join duplicates**: the row count before and after, and `merge(validate="m:1")` (or the cardinality you expect). More rows after than before is red: show which keys repeat on the right and why (a key duplicated in one table, a compound key joined on half of it). Check that the parts add up (matched + unmatched = the left table's rows): if they don't, a join duplicated something, and the human shouldn't be the one to notice.
3. **Rows each join loses**: an inner join drops the orphans of step 1; say how many, and which groups or segments they come from (a loss concentrated in one group biases every comparison after it).
4. **Shared columns agree**: a column in both tables (a family, a name, a date) must match on the joined rows; the share that disagrees, red if any.
5. **Coverage per group**: the full list of items (tasks, users, prompts) against each group compared: a stacked bar per segment × group, complete green, missing red, duplicated orange. Groups compared on different items, or on a different mix of segments, is the most common reason a headline difference isn't real.
6. **New or missing categories**: a category present for one group only (Deepchecks NewCategory): orange. A category nested in the group by design (each author works for one vendor) is expected: neutral.
7. **Overlap**: items shared between tables that must be disjoint (train / test, a public benchmark / a private set, the same record under two ids): the share shared, red if any (Deepchecks TrainTestSamplesMix). Exact matches on the id first, then on the content (the same text, the same geometry, the same hash) when the ids differ by construction. Split the overlap by source: it's rarely spread evenly.
8. **The outcome per segment**: if a table holds an outcome (passed, label, score), join it to the categories of the other tables and plot its rate per value of each category (sorted, the overall rate as a line). A value at 0 % or 100 % over 20+ rows is red: a broken segment (a grader, unit or logging bug), not a weak one (Cleanlab `underperforming_group`); far from the rest, orange. This checks the data, it doesn't answer the question: report it, don't explain it.
9. **The joined table**: pickle it (`df_<a>_<b>.pkl`), with its row count and key in the markdown above.

Then stop (`SKILL.md`), with the 3 bullets.
