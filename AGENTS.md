# eda: agent guide

Development home: Mac.

A hierarchical skill: `SKILL.md` is the base, always read; every other `.md` is read only when `SKILL.md`'s last table (or `3_analysis/analysis.md`'s, for `kinds/`) says so. `README.md` lists the tree and where each part comes from.

- **Public repo**: nothing personal or interview-specific goes in it (no person, company or exercise names).
- **A new sub-file** needs a row in the routing table that sends agents to it, and a line in `README.md`'s tree; a file nothing routes to is never read.
- **`kinds/` files share one template**: What you'll hear · Ask first · Hypotheses to list · The notebooks · The test · Traps · The answer's shape. `D_model_eval.md` is the reference.
- **The base stays short**: rules every step needs go in `SKILL.md`; anything only one step or one kind of data needs goes in its sub-file.
- **Paths are relative** to `SKILL.md`, which tells an agent with only that file to fetch the rest from `raw.githubusercontent.com/JeanLescutMuller/eda/main/<path>`: renaming a file breaks that, so update every routing table with it.
- Test a change by running an agent with the skill on a small dataset in a scratch folder, then check the notebooks' names and order, the cell formatting (no `;` chains), the colours, what it flagged, and that it stopped after each step.
