# Places: countries, regions, addresses, coordinates

These cells go after `table.md`'s, in the same notebook.

## The cells, in order

1. **Country codes**: which standard (ISO 3166 alpha-2 `CH`, alpha-3 `CHE`, full names, a mix). A mix in one column is orange: map everything to alpha-3 (`pycountry` if installed, else a small dict for the values present). Codes that aren't countries (`XX`, `EU`, `UK` instead of `GB`, empty): count them, orange.
2. **Spelling variants**: the same place written two ways (`"Zurich"` / `"Zürich"` / `"ZURICH"`): lower case, strip accents (`unicodedata.normalize("NFKD", s)`), and count the values that merge. Many merges: orange, and propose the mapping.
3. **Coordinates**:
   - out of range (latitude outside [−90, 90], longitude outside [−180, 180]): red;
   - exactly `(0, 0)` ("null island") or another repeated default point: red;
   - latitude and longitude swapped: points that land in the sea or in the wrong country when swapped back look right; check a few known places;
   - too few decimals (rounded to a city) or too many (fake precision): say which.
4. **Map**: `px.scatter_map(df, lat="lat", lon="lon", color=group)` (`px.scatter_mapbox` on older Plotly) for points, sampled if more than ~50,000; `px.choropleth(df_by_country, locations="iso3", color="n")` for counts per country. Points far from where the business operates: orange.
5. **Region mismatches**: two place columns that must agree (a country and a city, a postcode and a region, an IP country and a billing country): the share that disagrees. Red if the analysis groups by one of them.
6. **Per-capita or per-unit normalisation**: counts per country mostly follow population or the user base. Before comparing places, divide by the right denominator (users, population, stores) and say which; a raw count map is labelled as such.
7. **Concentration**: the share of rows in the top 1 and top 5 places (`value_counts(normalize=True)`). One place with most of the rows means every overall metric is really that place's: orange, and plan to split by it.

Add each finding to the flags table of `table.md`.
