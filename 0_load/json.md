# JSON, JSONL and nested fields

Goal: flat tables, one per level of nesting, each with the key of its parent, so the rest of the pipeline works on plain data frames.

## Before the first cell

Look at the first bytes of the file (`head -c 500`, or `open(path).read(500)`):

| Starts with | It is | Read with |
|---|---|---|
| `[` | one JSON array of records | `pd.read_json(path)` |
| `{` then a newline and another `{` | JSONL: one record per line | `pd.read_json(path, lines=True)` |
| `{` with a top-level key holding the list (`{"data": [...]}`) | an envelope | `json.load`, then the list under that key |
| `{` with ids as keys (`{"a1": {...}, "a2": {...}}`) | a dict of records | `pd.DataFrame.from_dict(d, orient="index")`, the index renamed to the id |

## The cells, in order

1. **Read as is**: the call above into `df_<table>_raw`; `head()` and `len()`. A JSONL file that fails on one line: read it line by line with `json.loads` in a `try`, count the bad lines (red if any), keep the good ones.
2. **Which columns are nested**: per column, the share of cells that are a `dict`, a `list`, or a scalar (`df[c].map(type).value_counts()`). A column mixing dicts and scalars or lists and nulls is orange: say which.
3. **Flatten the dicts**: `pd.json_normalize(records, sep=".")` turns nested dicts into `parent.child` columns. Stop at the depth that's useful (`max_level=1` or `2`); a deeper level of dicts stays one column for now.
4. **One table per list**: a field holding a list of records (an order's items, a conversation's messages) becomes its own table:

   ```python
   df_items = pd.json_normalize(
       records,
       record_path="items",
       meta=["order_id", ["customer", "id"]],
       sep=".",
   )
   ```

   The parent's key goes in `meta`, so the child table joins back. Name it `df_<parent>_<child>`.
5. **Lists of scalars** (tags, labels): `df.explode("tags")` into a long table with the parent key, or a count per row (`df.tags.str.len()`) when the values don't matter. Empty lists become NaN after `explode`: count them before.
6. **Ragged schemas**: records that don't share the same keys. A cell: per column, the share of records that have it (`df.notna().mean()`). A key present in < 5 % of records is orange (a rare field, or a schema version); a key that appears only after a date is a schema change: plot its presence per month.
7. **Types inside JSON**: numbers written as strings (`"12"`), booleans as `"true"`, dates as epoch seconds or milliseconds (`pd.to_datetime(x, unit="s")` or `"ms"`: check the year looks right). Then the type cell of `load.md`.
8. **Row counts**: parent rows, child rows, and that every child has a parent key that exists (green), none missing (red if any).
9. **Pickle each flat table**: `df_<parent>.pkl`, `df_<parent>_<child>.pkl`.

Then stop, with 3 bullets: the tables made and their keys; what was ragged or malformed; the row counts per level.
