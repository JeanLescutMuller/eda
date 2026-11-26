# Pairs of ids: users ↔ items, sender ↔ receiver, buyer ↔ seller

A table whose rows link two entities is a graph, even when nobody calls it one. Most questions need only pandas; use `networkx` only for components or paths.

## The cells, in order

1. **The two sides**: which column is the source and which the target, and whether they are the same kind of entity (users → users: one-sided; buyers → sellers, users → items: two-sided). The unique count on each side.
2. **Self-loops**: rows where source = target (a user rating their own item, a transfer to oneself): count; red if impossible by design, orange otherwise.
3. **Duplicate edges**: the same pair more than once (`df.duplicated(["src", "dst"], keep=False)`). Real repeats (several purchases) or double logging: compare their timestamps; same second is a logging retry, red.
4. **Degree distribution**: edges per entity, each side separately (`df.groupby("src").size()`), `px.histogram(log_x=True, log_y=True)`. A heavy tail is normal; say the share of all edges held by the top 1 % of entities.
5. **Power users dominating**: the top 10 entities by degree with their share of edges (`px.bar(text_auto=".1%")`). One entity with > 10 % of all edges is orange: every average is mostly theirs; plan to report with and without it. An entity linked to almost every other one is often a bot, a test account or a default value.
6. **Two-sided counts**: for a marketplace, per period, active buyers, active sellers, and edges (transactions) per side (`px.line`). A side that grows while the other shrinks changes every per-user metric; say which side a metric is "per".
7. **Isolated and orphan entities**: entities in the entity table (users, items) with no edge at all, and ids in the edges with no row in the entity table (red: check again in `20_joins`).
8. **Components**, only when the question is about groups or reach:

   ```python
   import networkx as nx

   g = nx.from_pandas_edgelist(df_edges, "src", "dst")
   s_sizes = pd.Series([len(c) for c in nx.connected_components(g)])
   ```

   The size of the largest component and the count of small ones. Fraud rings and test clusters show up as dense small components.
9. **Reciprocity**, for one-sided graphs (follows, messages): the share of edges that go both ways. Very low or very high compared with what the product implies: orange.

Add each finding to the flags table of `table.md`.
