# Journal - 2026-10-15 - Unmatched Budget Rows Are Still Evidence

## Today in one sentence
- I learned that a reliable Gold fact table should preserve appropriation-only and obligation-only records instead of hiding them with an inner join.

## What I learned
- Today I worked on the PondoLens regional expenditure Gold fact and the first annual approved-budget dashboard view. The fact table has a clear grain: one row for each `year × region × particular` combination.
- The two Silver sources do not match perfectly. Regional appropriations describe approved budget amounts, while regional obligations describe spending commitments. Even after standardizing year, category, particular, and region code, some keys appear in only one source.
- An inner join would keep only records found on both sides. That would make the dataset smaller, but it would also erase real gaps. The Gold notebook therefore uses a `FULL OUTER JOIN` and records one of three statuses: `MATCHED`, `APPROPRIATION_ONLY`, or `OBLIGATION_ONLY`.
- The validated output contains 4,487 rows. The match-status counts are: 874 rows for 2021, 893 for 2022, 893 for 2023, 855 for 2024, and 972 for 2025. These totals include unmatched records instead of silently dropping them.
- The status is not only a technical flag. It changes what can be calculated. A matched row can support both the execution gap and execution rate. An appropriation-only row has no observed obligation in the matched dataset, while an obligation-only row has no matched appropriation denominator. In both cases, `NULL` means unavailable or unmatched; it must not be replaced with zero without evidence.
- The notebook calculates `execution_gap_amount` as appropriation minus obligation. It calculates `execution_rate` as obligation divided by appropriation times 100, using `NULLIF` so a zero appropriation does not cause an invalid division. Negative gaps and rates above 100 are retained for review instead of being automatically corrected.
- Summary rows such as geographic `TOTAL`, category `GRAND TOTAL`, and particular `ALL` are excluded from the fact grain. Mixing them with detail rows would double-count money when the dashboard applies `SUM`.
- Central Office and Nationwide rows are retained because they are valid expenditure locations in the source, but a regional map must filter to `dim_regions.geographic_level = 'region'`. A row can be valid for a national bar chart and still be inappropriate for a choropleth.
- The dimensions are resolved before the fact is written. Missing year, region, or particular dimension keys block the load. This prevents a fact row from being created with a foreign key that points to nothing.
- The annual dashboard sums `appropriation_amount` by year. The chart makes the measure and unit explicit as approved budget in Philippine-peso trillions. It shows the reported series rising from about ₱4.51 trillion in 2021 toward a higher value in 2025, but the visual alone does not explain why the budget changed.

![Databricks dashboard line chart of approved Philippine government budget by year from 2021 to 2025](../assets/2026-10-15-approved-budget-by-year.png)

This inspected dashboard screenshot is useful evidence because it shows the Gold measure after aggregation, with a readable title, year axis, peso-trillion unit, and a 2021 tooltip value of ₱4.51 trillion.

## Terms I am still learning
- **Grain** - the exact meaning of one row. Here, one row represents one year, one geography, and one budget particular.
- **Full outer join** - a join that keeps matched rows and unmatched rows from both input tables.
- **Match status** - a label showing whether a Gold row came from both sources, appropriation only, or obligation only.
- **Foreign key** - an ID in the fact table that points to a valid member in a dimension table.
- **Reconciliation** - checking that source row counts and amounts are still represented correctly after joins and transformations.

## What confused me
- I initially expected the appropriation and obligation datasets to match row for row. The unmatched counts showed that different source coverage is a real data condition, not automatically a failed join.
- I also had to separate missing data from zero. An unmatched obligation does not prove that spending was zero, and an unmatched appropriation does not provide a safe denominator for an execution rate.
- The choropleth remains an open design issue because Databricks needs compatible Philippine boundary geometry. Region names or codes alone are not enough when the built-in geographic roles do not recognize the required administrative boundaries.

## One small next step
- [ ] Create one dashboard validation query grouped by year and match status, then compare its row counts and appropriation totals with the annual chart before adding more visualizations.

## Git checkpoint
- [x] I created or updated a file
- [x] I wrote a commit
- [x] I pushed my changes

## Decisions or assumptions (optional)
- Use `FULL OUTER JOIN` so unmatched source records remain visible.
- Keep `MATCHED`, `APPROPRIATION_ONLY`, and `OBLIGATION_ONLY` as explicit fact attributes.
- Exclude source summary rows from the detail fact to avoid double counting.
- Keep Central Office and Nationwide in the fact, but exclude them from region-only maps.
- Treat the current 4,487-row result and IDs `60001–64487` as a validated baseline, not a permanent hard-coded limit.
- Do not interpret the line-chart trend as a cause-and-effect conclusion; it only describes approved amounts by year.

## Evidence from today (optional)
- [Regional expenditure Gold notebook on `dev`](https://github.com/ItsYangCoder/ph-government-budget-analysis/blob/dev/databricks/notebook/04_gold/fact_regional_expenditure.ipynb)
- [Current PondoLens `main...dev` comparison](https://github.com/ItsYangCoder/ph-government-budget-analysis/compare/main...dev)
- [Approved-budget dashboard screenshot](../assets/2026-10-15-approved-budget-by-year.png)

## Reflection (optional)
- The most important shift today was accepting that unmatched rows are not noise to remove. They are evidence about where the two government budget sources do not align at the chosen grain.
- Building the dashboard made the modeling decisions more concrete. A clean chart can still be wrong if totals, summary rows, unmatched records, units, or filters are handled incorrectly underneath it.
- I want every visual to have a small validation query beside it. That feels less exciting than designing the chart, but it is what makes the result defensible.

## Mood or meme (optional)
- A prettier chart cannot rescue a lossy join.