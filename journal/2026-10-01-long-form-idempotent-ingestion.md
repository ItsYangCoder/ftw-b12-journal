# Long-Form Data and Idempotent Local Ingestion

## Today in one sentence

I learned that converting wide government reports into long-form tables is not enough by itself; the pipeline also has to preserve meaning, reject unsafe records, and produce the same current database state when unchanged inputs are run again.

## What I learned

Today’s PondoLens work moved beyond collecting source files. The `dev` branch now contains source-specific conversion, validation, tests, and local SQLite publication for nine DBM and PSA originals: seven Excel workbooks and two regional PDFs.

The sources cannot safely share one generic parser. Agency actuals, GAA appropriations, sector expenditure, regional obligations, population projections, and GRDP use different layouts and business meanings. The pipeline uses separate converters but writes a shared traceable schema with fields such as source hash, year, entity, measure, unit, value state, source table, page, row, and column.

A **wide** report often stores years, regions, or measures across many columns. A **long-form** table turns those repeated columns into rows. A simplified transformation looks like this:

```text
wide:  department | allotment | obligation | disbursement
long:  department | measure      | value
       DepEd      | allotment    | ...
       DepEd      | obligation   | ...
       DepEd      | disbursement | ...
```

The long version is easier to filter, validate, union within the same contract, and load into SQL. It is also naturally larger because department names and other keys repeat on every measure row. A larger row count or file size does not automatically mean duplication.

That explains why nine original files produced eighteen CSV outputs. Some originals contain more than one dataset type. For example, GAA budget lines are separated from control totals; agency summary records are separated from unresolved detail candidates; and population values are separated from growth measures. These supporting or control rows should not be added into financial totals just because they are stored as CSV.

The recorded conversion processed 1,489,811 rows across the output datasets. All nine sources were reported as `PASS`, with zero duplicate keys and zero failed implemented checks. An unchanged rerun reported `REUSED=True`. The repository’s current test suite also recorded 19 passing tests. These results verify the implemented source controls, but they do not mean cross-source analysis is ready.

I also understood idempotency more precisely. The pipeline does not reuse an output just because the folder already exists. Its run fingerprint includes the converter version, code fingerprint, dependency versions, input SHA-256 hashes, configuration, and input mode. It then verifies stored output checksums. If those conditions match, conversion can be reused. SQLite is still rebuilt as a current snapshot, so the rerun does not append another copy of the same rows.

If a source changes, its hash changes and the run receives a different ID. If a source has duplicate business keys, unknown units, or failed reconciliation checks, the candidate rows remain available for review but that source partition is not published as valid. This is safer than deleting suspicious rows merely to make the run pass.

The department-mapping review shows why the next layer remains gated. It contains 74 agency/fund entries: 67 have normalized-name candidates and seven have no candidate match. Every row is still marked `REVIEW_REQUIRED`. A suggested name match is not proof that two records have the same entity, budget scope, or year coverage.

![PondoLens Philippine government budget analytics project mark](../assets/2026-10-01-pondolens-project-mark.png)

*I used this inspected PondoLens project image because the available terminal and chat screenshots exposed local usernames, machine names, or people’s identities. The technical run results are preserved as text evidence instead.*

## Terms I am still learning

- **Wide format** — a table where repeated dimensions such as years or measures are spread across columns.
- **Long format** — a table where each observation and measure is represented as a row with explicit dimension columns.
- **Business key** — the set of fields that should uniquely identify one record at its declared grain.
- **Idempotency** — rerunning the same operation with unchanged inputs produces the same final state without duplicate effects.
- **Fingerprint** — a repeatable identifier derived from code, configuration, dependencies, and input hashes.
- **Reconciliation check** — a comparison proving that converted detail agrees with an expected source total or control.
- **Current snapshot** — the database state representing the latest accepted source partitions rather than an append-only copy of every rerun.

## What confused me

I initially wondered why there were eighteen CSVs when there were only nine original files, and whether similarly named outputs were accidental duplicates. The answer is that one source can intentionally produce separate analytical, control, and review datasets. File count alone cannot determine duplication; the source hash, dataset name, year, and business key provide the real identity.

I also need to remember that `PASS` has a limited meaning. It means the implemented source-specific checks passed. It does not approve the department crosswalk, fund scope, regional boundaries, or the comparability of GAA appropriations with agency obligations.

My open question is how much mapping logic should be automatic. Normalized names can generate candidates, but renamed departments, abbreviations, autonomous regions, and legislative councils require evidence beyond string similarity.

## One small next step

Review the seven `no_match` department rows against the official 2024 and 2025 source labels, assign a candidate only when entity and scope match, and record the reason in `review_notes` before approving any cross-source join.

## Git checkpoint

- [x] I created or updated a journal file.
- [x] I wrote a commit containing the journal and inspected project image.
- [x] I pushed the commit to `main`.
- [ ] I approved the unresolved department mappings for cross-source analysis.

## Decisions or assumptions (optional)

- Keep raw Excel and PDF files unchanged and outside Git.
- Use source-specific converters instead of forcing every report through one generic parser.
- Keep analytical records, source controls, and review candidates in separate datasets.
- Treat `REUSED conversion` as valid only when the recorded fingerprint and output checksums match.
- Publish only passing source partitions into the current SQLite snapshot.
- Keep `ready_for_cross_source_analysis` false until department scope and regional-boundary reviews are complete.
- Treat Parquet, dbt Silver/Gold, and Databricks deployment as next stages, not completed work.

## Evidence from today (optional)

- [PondoLens `dev` branch](https://github.com/ItsYangCoder/ph-government-budget-analysis/tree/dev)
- [Validated long-form pipeline and mapping-review commit](https://github.com/ItsYangCoder/ph-government-budget-analysis/commit/ff34f546e2f793feb1fcbac01d9ced8fcf2eae00)
- [Merged documentation PR #8](https://github.com/ItsYangCoder/ph-government-budget-analysis/pull/8)
- [Long-form conversion and publication implementation](https://github.com/ItsYangCoder/ph-government-budget-analysis/blob/dev/src/pondolens/long_form.py)
- [Regression tests for repeatability and financial controls](https://github.com/ItsYangCoder/ph-government-budget-analysis/blob/dev/tests/test_long_form.py)
- The reviewed conversion record: nine source files passed, 1,489,811 output rows, zero duplicate keys, zero failed implemented checks, and reuse on an unchanged run.
- The reviewed mapping CSV: 74 entries, 67 normalized-name candidates, seven unmatched entries, and all rows still requiring review.

## Reflection (optional)

The satisfying part is seeing messy Excel and PDF inputs become queryable tables without pretending that every row is already comparable. The difficult part is accepting that a successful conversion is only one gate. The pipeline is more trustworthy because it can say “source tables are ready” while still saying “cross-source analysis is blocked.”

## Mood or meme (optional)

**More rows do not always mean more data; sometimes they mean the same data finally has a clear shape.**
