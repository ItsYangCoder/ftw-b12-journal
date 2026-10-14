# Journal - 2026-10-14 - Stable IDs Need a Business Key

## Today in one sentence
- I learned that an append-safe Silver table needs two different kinds of identity: a business key for matching records and a stable surrogate ID for downstream relationships.

## What I learned
- Today I finalized the SQL-only GRDP per-capita Silver notebook and merged it into the PondoLens `dev` branch through [PR #38](https://github.com/ItsYangCoder/ph-government-budget-analysis/pull/38). The notebook cleans the 2021–2025 Bronze data, validates it, allocates IDs for new keys, performs a Delta `MERGE`, and reconciles the result.
- The business key is `(year, region_code, price_basis, geographic_level)`. Together, these columns describe which GRDP-per-capita observation a row represents. `grdp_per_capita_id` is different: it is a surrogate key used as a compact technical identifier. The number `100000001` does not contain business meaning; it is simply the first value in the reserved ID range.
- Existing IDs must stay stable. If a region name or GRDP value is corrected, the `MERGE` matches the row through its business key and updates the changed attributes without assigning a new ID. If the business key has never appeared before, the notebook allocates the next ID after the current maximum using `ROW_NUMBER()`.
- This prevents a full rebuild from renumbering old records. Stable IDs matter because a future Gold fact or dimension may reference them. Changing those IDs during every refresh would break those relationships even when the underlying observation is still the same.
- The notebook validates before writing. It rejects blank keys, non-positive GRDP-per-capita values, unsupported price bases, invalid geography levels, and duplicate business keys. It also checks that a national record uses `PHL` and that current-price rows use the observation year while constant-price rows use `2018`.
- `MERGE` helps with reruns because matched rows are updated only when selected values changed, while unmatched rows are inserted. This is more suitable than a plain `INSERT`, which could append duplicate versions of the same observation. The final `EXCEPT ALL` comparison checks both directions so missing, extra, or changed rows can be detected.
- There is still an important limit: ID allocation reads `MAX(grdp_per_capita_id)` before assigning new values. Two writers doing that at the same time could choose the same next range. The safe operating rule is one writer per target table until a stronger concurrency-safe ID strategy is added.
- Repository checks passed and the pull request was merged, but that is not the same as a Databricks runtime test. The notebook still needs an actual Databricks run to verify the SQL variables, temporary views, `MERGE`, and rerun behavior against the Delta table.

![Databricks catalog showing the final GRDP per-capita Silver table columns](../assets/2026-10-14-grdp-silver-schema.png)

The screenshot shows the final Silver table shape: one surrogate ID, the fields that define the observation, the GRDP-per-capita measure, and processing metadata.

## Terms I am still learning
- **Business key** - the real data fields that uniquely identify an observation. Here it is year, region code, price basis, and geographic level.
- **Surrogate key** - a generated technical ID with no business meaning, such as `100000001`.
- **Upsert** - update a matching row or insert it when no match exists.
- **Idempotent rerun** - running the same input again should not create duplicate business records or unnecessarily change stable IDs.
- **Reconciliation** - comparing expected source rows with the saved target rows to prove that nothing is missing, extra, or unexpectedly changed.

## What confused me
- At first, I treated the large numeric ID as if it defined the record. The clearer model is that the business key defines the record, while the numeric ID is only a stable reference.
- I also had to separate code validation from runtime validation. A passing repository check proves that the notebook structure and supported SQL were accepted by CI; it does not prove that every statement has executed successfully in Databricks.
- I still want to understand the best production ID strategy when more than one job could write to the same table. The current `MAX + ROW_NUMBER()` approach is reasonable only with the documented single-writer rule.

## One small next step
- [ ] Run the notebook twice in Databricks with the same Bronze snapshot, then add one controlled new business key and verify that existing IDs stay unchanged, the unchanged rerun adds no duplicate, and the new row receives the next ID.

## Git checkpoint
- [x] I created or updated a file
- [x] I wrote a commit
- [x] I pushed my changes

## Decisions or assumptions (optional)
- Keep `grdp_per_capita_id` as an integer in the current reserved range and block the load if it would exceed `2147483647`.
- Use `(year, region_code, price_basis, geographic_level)` as the business key.
- Normalize region codes without dashes in this Silver table, including `IVB` for MIMAROPA and `PHL` for the national row.
- Use one writer per target table. Do not overlap a Bronze refresh with the Silver load.
- Do not claim the notebook is production-verified until the Databricks runtime and rerun checks are complete.

## Evidence from today (optional)
- [Merged PondoLens PR #38: Restore SQL-only GRDP per capita Silver notebook](https://github.com/ItsYangCoder/ph-government-budget-analysis/pull/38)
- [Current GRDP per-capita Silver notebook on `dev`](https://github.com/ItsYangCoder/ph-government-budget-analysis/blob/dev/databricks/notebook/03_silver/grdp_per_capita.ipynb)
- [Safe table-schema screenshot](../assets/2026-10-14-grdp-silver-schema.png)

## Reflection (optional)
- The most useful lesson was that “continue the ID” is not just a numbering problem. It depends on first defining what makes a row the same business record across runs.
- The SQL became easier to reason about after separating the work into cleaning, validation, target checks, ID allocation, `MERGE`, and reconciliation. Each section answers one specific failure question.
- I want the next test to focus on behavior, not appearance: same input, corrected input, and one new key. Those three cases will show whether the notebook really preserves identity over time.

## Mood or meme (optional)
- Today’s reminder: the ID can be random-looking, but the rule behind it cannot be random.