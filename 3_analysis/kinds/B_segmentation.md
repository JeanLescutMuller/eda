# B. Which groups are there?

Splitting users, customers, items or sessions into groups that behave differently, either along dimensions that already exist (country, plan, tenure) or with clusters found in the data. The goal is a few segments the business can act on, not an algorithm.

## What you'll hear

- "Who are our users?"
- "Which customers should we target with this offer?"
- "Are there different kinds of usage?"
- "Group these products by how they sell."

## Ask first

| Ask | Why |
|---|---|
| What will the segments be used for (targeting, pricing, a product decision)? | picks the features: what the action can change |
| Existing dimensions first, or clusters? | rules on known columns (plan × tenure) are easier to act on and explain |
| How many segments can the business use? | 3–6 in practice; more is rarely acted on |
| Over which period, and at which unit (user, account, session)? | behaviour features need a window |

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| the population is not homogeneous on the outcome | the outcome's spread is the same within every candidate split |
| an existing dimension already separates the behaviour | its segments' outcome distributions overlap almost fully |
| a found cluster is real | it changes a lot with the seed, k or a resample |
| a cluster is only one feature's scale | it disappears once features are scaled |
| a segment is just size (big vs small users) | it holds after normalising by activity |

## The notebooks

1. **Features per unit**: one row per user (or item), a few behaviour columns (frequency, recency, spend, tenure, share of each product); `df.describe()`, and skewed columns log-transformed (`np.log1p`).
2. **Known dimensions first**: the outcome per value of each existing dimension (`px.box(x=dim, y=outcome)`), and a two-way table of the best pair (`px.imshow(pivot, text_auto=".0f")`).
3. **Scale**: `StandardScaler().fit_transform(X)` on the chosen features; a cluster on raw scales follows whichever column has the largest units.
4. **Choose k**: `KMeans(n_clusters=k, n_init=10, random_state=0)` for k = 2..8, inertia and `silhouette_score` per k (`px.line`); pick a k the business can use where the silhouette is reasonable.
5. **Stability**: refit on 5 bootstrap samples or 5 seeds, and compare labels with `adjusted_rand_score`; below ~0.7, the clusters are not real.
6. **Profile each segment**: size (n and %), the mean of every feature per segment as a heatmap (`px.imshow(profile_z, text_auto=".1f")`, z-scored by column), the outcome per segment; give each a plain name from its profile.
7. **Look at it**: `px.scatter` of the two main PCA components (`PCA(2)`), coloured by segment (dark blue, cyan, black, gray, purple).
8. **Act on it**: per segment, the size and the outcome gap that justify a different action.

## The test

`tests.md`: Kruskal-Wallis on the outcome across segments, chi-square for a categorical outcome. Clusters are built to differ on their own features: a test on those features is circular; test only on an outcome not used to build them.

## Traps

| Trap | Fix |
|---|---|
| features on different scales | scale before clustering |
| skewed features (spend, counts) | log first |
| clusters that change with the seed | the stability check; report it |
| testing a cluster on the features it was built on | test on a held-out outcome |
| too many segments to act on | cap k at what the business can use |
| segments that are only activity level | normalise by activity, or say so |

## The answer's shape

3–6 named segments, each with its size, its profile in one line, its outcome, and the action it suggests; how stable they are; and whether an existing dimension would do almost as well (simpler to use).
