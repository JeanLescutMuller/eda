# Parquet, Excel, SQL, many files, big files

The same goal as `load.md` (raw, then typed, then pickled); only the reading changes.

## The cells, in order

1. **Parquet**: `pd.read_parquet(path)` (needs `pyarrow`: `%pip install pyarrow` if missing). The types are already stored: check them anyway (a date stored as a string, an id stored as a float). A folder of parquet files is one table: `pd.read_parquet(folder)`.
2. **Excel**:
   - list the sheets first: `pd.ExcelFile(path).sheet_names`; one table per sheet that holds data, named after the sheet;
   - a header that isn't on the first row (a title, a logo, blank rows): `header=n` or `skiprows`; look at the first 10 rows raw (`header=None, nrows=10`) to find it;
   - merged cells come in as one value then NaN: `ffill()` on that column, said in the markdown;
   - totals and notes at the bottom: look at `tail()`, cut them;
   - numbers stored as text, dates as Excel serial numbers (`pd.to_datetime(x, unit="D", origin="1899-12-30")`).
3. **SQL**: list the tables first (`sqlite_master` for SQLite, `SHOW TABLES` in duckdb), then `pd.read_sql("SELECT * FROM t", con)` per table. With a big database, read the row count first (`SELECT COUNT(*)`), and push filters and aggregations into the query.
4. **Many files for one table** (one per day, per shard, per source):

   ```python
   from pathlib import Path

   l_paths = sorted(Path("data").glob("events_*.csv"))
   df_events = pd.concat(
       [pd.read_csv(p).assign(source_file=p.name) for p in l_paths],
       ignore_index=True,
   )
   ```

   Keep the file name as a column: a problem often belongs to one file. Check every file has the same columns (a set per file, compared): a file with a different schema is orange.
5. **Too big for pandas** (more than about a quarter of the memory): don't load it all.
   - duckdb queries the files where they are, and returns a small frame:

     ```python
     import duckdb

     df_events_by_day = duckdb.sql(
         "SELECT date_trunc('day', ts) AS day, count(*) AS n FROM 'data/*.parquet' GROUP BY 1"
     ).df()
     ```

   - or a sample, with the seed and the share stated in the markdown: `USING SAMPLE 1%` in duckdb, or `df.sample(frac=0.01, random_state=0)`; sample whole units (all of a user's events), never rows of a unit;
   - or chunks: `pd.read_csv(path, chunksize=1_000_000)`, aggregating each chunk.
6. **Row counts match**: the rows read against the source (`COUNT(*)`, the file's line count minus the header, the sum over files). A difference is red until explained.
7. **Pickle** each table (or the sample, named `df_<table>_sample.pkl`).
