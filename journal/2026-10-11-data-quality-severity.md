# Journal - 2026-10-11 - Data Quality Checks Need Severity

## Today in one sentence

I learned that a data quality check is only useful when it says not just what failed, but also how serious the failure is, which records or metrics are affected, and whether the pipeline should continue, quarantine data, or block publication.

## What I learned

- There was no new Data Engineering commit in the PondoLens repository today, so I used the team's latest profiling consolidation as a learning reflection instead of claiming new implementation work.
- The profiling results contain different kinds of problems, and they should not all have the same outcome. A failed numeric parse in one row, an unknown unit for an entire source, a small reconciliation difference, and an incompatible regional boundary are all quality findings, but their **blast radius** is different.
- A useful quality rule needs at least five parts:
  1. **Condition** — the exact SQL or Python expression being checked.
  2. **Scope** — the rows, table, metric, or publication affected.
  3. **Severity** — informational, warning, quarantine, or blocking.
  4. **Action** — continue, flag, isolate, or stop the affected output.
  5. **Evidence** — counts, sample rows, source notes, and test results.
- I can think of severity as a decision, not as a label added after a query. For example, 12 failed rate parses in `agency_actuals` may be handled at row level: keep the raw text, set the typed rate to NULL, assign `value_status = 'invalid_numeric'`, and quarantine or exclude those rows from rate calculations. That should not automatically erase every valid agency amount.
- An unresolved source unit has a wider impact. If a regional GAA value might be in thousands but the unit note is not verified, every peso-based total derived from that field could be off by a factor of 1,000. The ingestion can preserve the raw value, but normalized peso metrics should remain blocked.
- Reconciliation checks require a third state. The profiling review says only 102 of 1,008 agency groups produced a class-total comparison. When one side is absent, the result is **not comparable**. It is incorrect to convert that NULL difference into zero and count it as a pass.
- A warning should still be measurable. If national population differs from the sum of regions by 100 or 200 persons, I should record the absolute difference, relative difference, year, and source note. A tolerance can support review, but it should not silently rewrite the source totals.
- A geography mismatch is not just a row error. If population uses one regional boundary and GRDP or spending uses another, then the affected per-person or regional comparison is semantically unsafe. The tables may load successfully, yet the analytical output should be blocked until a verified crosswalk or compatible aggregation exists.
- This means pipeline success and publication readiness are separate states. A pipeline can finish with warnings and quarantined records while some metrics remain unavailable. Conversely, stopping every run for every imperfect cell can make the system unusable and hide which unaffected outputs are still trustworthy.
- I found a practical way to separate the decisions:
  - **Info** — record a profile or expected variation; no output is restricted.
  - **Warning** — continue, but expose the issue and monitor its count or size.
  - **Quarantine** — isolate invalid rows while allowing verified rows to continue.
  - **Block** — do not publish the affected table or metric because its meaning could be materially wrong.
- Severity should attach to the smallest safe scope. An invalid percentage may block one row's rate, not the whole dataset. An unknown unit may block every normalized amount from a source. An incompatible denominator may block only per-capita metrics while leaving raw counts available.
- Quality results also need stable identifiers such as `check_id`, `dataset`, `column`, `severity`, `failed_count`, `affected_output`, `run_id`, and `checked_at`. Without those fields, a dashboard may show a red number but not tell the team what decision to make.
- The Silver layer can therefore expose two related things: clean typed columns for valid values, and quality metadata that preserves why other values are missing, quarantined, or blocked. This is more honest than forcing every row into a clean-looking table.
- The important design principle is **fail at the right boundary**. I should stop only the output whose correctness is no longer defensible, while keeping raw evidence and unaffected outputs available for investigation.

## Terms I am still learning

- **Severity** — the level of response required when a quality rule fails.
- **Blast radius** — the amount of data or number of downstream outputs affected by one problem.
- **Quarantine** — keeping invalid or uncertain records separate so they are not used as trusted data while preserving them for review.
- **Publication gate** — a rule that decides whether a table, metric, or dashboard result is safe to release.
- **Tolerance** — an approved range of difference that may be acceptable because of rounding or another documented source behavior.
- **Three-state check** — a comparison that can be pass, fail, or not comparable.
- **Affected scope** — the smallest table, partition, column, row set, or metric that a quality result should restrict.
- **Fail closed** — withholding an affected output when required evidence is missing instead of assuming it is safe.

## What confused me

- I used to think a quality check should simply pass or fail. The profiling findings show that this is too limited. A failed parse may require row quarantine, while an unresolved unit may require a metric-level block.
- I am still unsure where the severity definition should live. It could be in SQL tests, Python configuration, a versioned data contract, or a dedicated quality-rule table. The important part is preventing different notebooks from assigning different consequences to the same finding.
- A tolerance is useful for small reconciliation differences, but it can also hide a real error if it is chosen only to make the check pass. I still need evidence for why a specific tolerance is acceptable.
- I also need to separate “the pipeline may continue” from “the data is ready for analysis.” Continuing execution is an operational decision; publishing a metric is a semantic decision.
- The term **blocker** can become too broad. If I block the entire pipeline because one metric lacks a verified denominator, I may unnecessarily prevent valid raw and Silver outputs from being produced.

## One small next step

- [ ] Create a small quality-rule matrix for three existing PondoLens findings—failed agency rate parses, unknown regional GAA units, and incompatible geography boundaries—with a check condition, severity, affected scope, action, and required evidence for each one.

## Git checkpoint

- [x] I created or updated a file
- [x] I wrote a commit
- [x] I pushed my changes

## Decisions or assumptions (optional)

- I will not use one global pass/fail result for every quality finding.
- I will assign severity based on the consequence of the issue, not only the number of failed rows.
- I will quarantine invalid rows when verified rows can still be used safely.
- I will block only the affected metric or dataset when the problem has a limited scope.
- I will keep raw values and source lineage even when a typed Silver value is NULL.
- I will treat NULL comparisons as not comparable unless both required values exist.
- I will require documented source evidence before approving unit conversions or tolerance thresholds.
- I will report pipeline completion and publication readiness as separate statuses.

## Evidence from today (optional)

![Profiling output preserving an explicit geography-status assumption for regional data](../assets/2026-10-05-bronze-source-profiling.png)

I reused this previously inspected repository image because today's only new accessible screenshot was a private Messenger conversation with a visible name and profile image, and it was unrelated to Data Engineering. The profiling image is safe and relevant because it shows a geography assumption that should become a warning or publication blocker until boundary compatibility is verified.

- [PondoLens profiling consolidation commit](https://github.com/ItsYangCoder/ph-government-budget-analysis/commit/ce79570abd66bb751f6b288ceab7965d82b84f91)
- [Profiling consolidation review on `dev`](https://github.com/ItsYangCoder/ph-government-budget-analysis/blob/dev/docs/profiling_consolidation_review.md)
- [Reused safe profiling image](https://github.com/ItsYangCoder/ftw-b12-journal/blob/main/assets/2026-10-05-bronze-source-profiling.png)

## Reflection (optional)

- Today's learning was less about writing another validation query and more about deciding what a failed result should mean.
- The profiling work already gives the team many counts and examples. The next level is connecting each finding to a controlled response so the pipeline does not either ignore everything or stop everything.
- I like the idea of failing at the smallest safe boundary because it keeps the system honest without making it fragile. A questionable rate should not become a clean zero, but it also should not automatically destroy unrelated verified amounts.
- This also changes how I think about dashboards. A quality dashboard should not only display red and green checks; it should tell the team which outputs remain usable and why another output is being withheld.
- I want to understand better how tools such as dbt tests or Delta expectations can record warnings, quarantines, and blocking failures while keeping one consistent rule definition.

## Mood or meme (optional)

- Not every red flag needs to stop the whole train, but every red flag needs an owner and a destination. 🚦
