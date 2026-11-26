# 20_joins: the tables together

Load the latest pickles: a table's `_clean` pickle when the human had it cleaned, else the one from `00`. One cell per check, each with its verdict.

## The cells, in order: 5, then the pickle

1. **Join keys**: for each pair of tables sharing a key, the keys in the left only / both / right only (`merge(how="outer", indicator=True)`, `px.histogram(x="_merge", text_auto=True)`): orphans red.
2. **Rows each join duplicates**: the row count before and after, with `merge(validate="m:1")` (or the cardinality you expect). Check that the parts add up (matched + unmatched = the left table's rows). More rows after: red, and show which keys repeat and why.
3. **Rows each join loses**: how many, and from which groups or segments. A loss concentrated in one group biases every comparison after it: red.
4. **Coverage per group**: every item against every group compared, as a stacked bar per group (complete green, missing red, duplicated orange). Groups compared on different items is the most common reason a headline difference isn't real.
5. **Overlap**: items in two tables that must be disjoint (train / test, a benchmark / a private set): match on the id, then on the content (the same text, the same hash) when ids differ by construction; the share shared, split by source. Red if any.
6. **Pickle the joined table** (`df_<a>_<b>.pkl`), its row count and key in the markdown above.

Then hand back (`SKILL.md`).
