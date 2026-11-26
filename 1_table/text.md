# Text columns, prompts, model outputs: 3 cells

After `table.md`'s cells, in the same notebook.

1. **Lengths**: characters per row (`s.str.len()`), `px.histogram(color=group)`. A spike at one exact length (a template, a truncation limit), or lengths that differ a lot between groups: orange.
2. **Empty and placeholder values**: nulls, `""`, whitespace only, "N/A", "none", "-"; for model outputs, also refusals ("I can't", "I'm sorry") and errors. Share per group: > 1 % orange, > 20 % red.
3. **Duplicates**: the share of repeated texts and the 10 most repeated (`s.value_counts().head(10)`). The same text many times where it should vary (an output, an answer): red.

Add each finding to the flags table.
