# Journal - 2026-10-06 - Idempotent and Reconciled Bronze Loading

## Today in one sentence

I learned that a Bronze load is not complete just because Delta tables were created; the same batch must be safe to rerun, preserve its lineage, and reconcile back to the Parquet source without missing or changing records.

## What I learned

- Today I moved from preparing source files to a real Databricks Bronze-loading workflow on the `feat/bronze-ingestion` branch. The notebook first verifies the uploaded extraction packages, then loads them into `pondolens.bronze`, and finally compares the loaded tables with the source Parquet files.
- The first important gate happens before any write. The notebook expects **18 extraction packages containing 21 Parquet tables**. It checks each manifest, required files, SHA-256 hashes, recorded row counts, source identifiers, processing keys, and extraction validation results. If one required check fails, loading stops. This protects Bronze from accepting an incomplete or changed upload.
- A **manifest** is more than a file list. In this workflow, it connects each Parquet table to its source ID, processing key, original-file hash, expected row count, and validation evidence. That information becomes the load plan instead of relying on handwritten table names.
- The Bronze load uses a deterministic `record_id` and a Delta merge that inserts only unmatched records. A deterministic ID is generated from stable source information, so the same source record receives the same identifier on every rerun. The verified rerun produced **zero candidate new rows**, which is concrete evidence that the same batch did not duplicate data.
- Zero new rows alone is not enough. A bad pipeline could also insert nothing because it read the wrong folder or skipped records. The notebook separately verifies:
  - the source row count against the manifest;
  - non-null and unique record IDs;
  - the source ID, processing key, and source-file hash;
  - the source and target schemas;
  - the loaded row count and distinct IDs;
  - all field values in both directions between Parquet and Delta.
- The two-way comparison matters. Checking only `source EXCEPT target` finds source rows that were lost, while checking `target EXCEPT source` finds unexpected rows in the selected processing revision. Both must be empty before I can say that the load preserved the batch.
- The verified result was **21 tables and 798,440 records/cells, all passing**. I need to describe that number carefully: many rows represent raw workbook cells or extracted measures, not 798,440 government financial transactions.
- The empty-table creation fix was also a useful Databricks lesson. The target Delta table is created from the source schema with zero data before the merge, using a supported append-mode write. This gives the merge a typed destination without preloading the batch.
- Bronze still is not Silver. The notebook preserves extracted values, source locations, raw JSON, parsed values, and quality flags. It does not yet standardize department names, choose comparable geographic vintages, or perform analytical joins.

## Terms I am still learning

- **Deterministic record ID** — an identifier calculated from stable source fields so the same record receives the same ID every time.
- **Idempotent load** — rerunning the same batch leaves the target in the same correct state instead of adding duplicates.
- **Reconciliation** — proving that the records written to the target agree with the expected source counts and content.
- **Processing key** — an identifier for one exact source-and-parser revision, used to separate repeatable extraction results.
- **Delta merge** — an operation that matches source and target rows and applies controlled inserts or updates.
- **Lineage** — metadata that connects a Bronze record to its source file, source ID, hash, processing revision, and original location.

## What confused me

- A zero-row rerun initially looks like the strongest success signal, but it does not prove completeness by itself. I still need the manifest checks and the two-way source-to-target comparison to rule out a silent skip.
- I am still deciding how a future corrected source should behave. If the source bytes change, the processing key should change and the new revision should remain distinguishable from the earlier Bronze revision. I do not want a correction to silently overwrite history.
- The batch includes reference and context data such as PSGC, population, and GRDP alongside 2024–2025 budget sources. They belong in Bronze, but their different vintages must remain visible so they are not treated as automatically comparable in Silver.

## One small next step

- [ ] Save a compact Bronze run summary with the run time, processing keys, 21 target tables, expected rows, actual rows, and candidate-new-row counts so the next run can be audited without reopening every notebook output.

## Git checkpoint

- [x] I created or updated a file
- [x] I wrote a commit
- [x] I pushed my changes

## Decisions or assumptions (optional)

- I will block loading when a manifest, checksum, required file, row count, lineage field, or extraction validation does not match.
- I will use deterministic record IDs and insert-only matching for the same Bronze revision instead of appending the same records again.
- I will treat **798,440** as verified records/cells across the extraction outputs, not as a count of financial transactions.
- I will keep Bronze focused on preservation and traceability; business standardization and cross-source joins belong in later layers.
- I will keep the implementation on `feat/bronze-ingestion` as branch evidence and will not claim it is already merged into `dev` or `main`.

## Evidence from today (optional)

![PondoLens source profile showing a documented geography status before Bronze ingestion](../assets/2026-10-05-bronze-source-profiling.png)

This previously inspected repository image shows how source assumptions such as geographic coverage are preserved as explicit metadata before loading. Today's exact Bronze reconciliation results are documented in the linked implementation commit and verified notebook output.

- [Bronze loading and verification commit](https://github.com/ItsYangCoder/ph-government-budget-analysis/commit/f024e097155718e590f29a51ac05bec188be7ce4)
- [Empty Bronze table creation fix](https://github.com/ItsYangCoder/ph-government-budget-analysis/commit/7df22ed36e74fdf2c8f9f1784acddf4286d40d26)
- [Databricks entry-notebook documentation](https://github.com/ItsYangCoder/ph-government-budget-analysis/commit/96546da2b3af98aab69afe5d0c457821a6688b45)
- [Reusable Bronze extraction pipeline](https://github.com/ItsYangCoder/ph-government-budget-analysis/commit/41c83353c7b7f914fcf08bea8b69772c81e0ad6c)

## Reflection (optional)

- The most important change in my thinking is that loading and verifying are one workflow. I should not celebrate a table creation before I can explain exactly what was loaded and prove that it matches the source.
- The implementation became easier to reason about when each notebook section had one purpose: setup, verify files, load, reconcile, and preview.
- I want to keep this habit in Silver: define the evidence for correctness before treating the transformed tables as ready for analysis.

## Mood or meme (optional)

- Today's rule: **zero duplicates is good; zero unexplained differences is better.** 🥉
