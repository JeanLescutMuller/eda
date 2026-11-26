# Text columns: free text, prompts, model outputs

`ydata-profiling` says little about text beyond lengths. These cells go after `table.md`'s, in the same notebook, one per check.

## The cells, in order

1. **Lengths**: characters and words per row (`s.str.len()`, `s.str.split().str.len()`), `px.histogram`, per group if there are groups. A spike at one exact length is orange (a template, a truncation limit); lengths that differ a lot between groups is orange (the groups write differently, or one is cut).
2. **Empty and blank**: nulls, `""`, whitespace only (`s.str.strip().eq("")`), placeholders ("N/A", "none", "-"). Share > 1 % orange, > 20 % red.
3. **Exact duplicates**: `s.duplicated(keep=False)` share, and the most repeated values (`s.value_counts().head(10)`). The same text many times is a template, a default value or a bot: orange, red if it's an outcome or a model output that should vary.
4. **Near-duplicates**: normalise (lower case, strip punctuation and whitespace) and count duplicates again; for longer texts, the same first 100 characters. Many more duplicates after normalising: orange.
5. **Templates and boilerplate**: the most common first and last 5 words (`s.str.split().str[:5].str.join(" ").value_counts()`). A shared opening or signature in most rows inflates every length and similarity: say so.
6. **Encoding garbage**: rows with `Ã`, `â€`, `�` or control characters (`s.str.contains("Ã|â€|�")`): mojibake from a wrong encoding, red if any; HTML tags or escaped markup (`&amp;`, `<br>`): orange.
7. **Language**: if the text should be in one language, the share in others (`langdetect`, or the share of non-ASCII letters as a quick proxy), per group. A group with a different language mix is orange.
8. **Model outputs**, when the text is generated:
   - refusals ("I can't", "I'm sorry", "As an AI"): share per model or group;
   - truncation: rows ending mid-sentence, or at the maximum length seen (`len == s.str.len().max()`);
   - empty or error outputs ("error", stack traces, `null`);
   - the prompt echoed back in the output;
   - each: share per group, red if one group has far more.
9. **Top words or n-grams**: `sklearn.feature_extraction.text.CountVectorizer(ngram_range=(1, 2), stop_words="english")`, the top 20 per group as `px.bar`. A word that appears in one group only is often a label leak or a template.
10. **Length vs the outcome**: if there's an outcome (passed, label, score), `px.box(x=pd.qcut(length, 4), y=outcome)` and Spearman's ρ. A strong link is orange: a grader or a model that rewards length, or a confounder for every later comparison.

Add each finding to the flags table of `table.md`.
