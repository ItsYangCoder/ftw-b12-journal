# Ingestion Manifests as Data Contracts

## Today in one sentence

I learned that downloading official files is only the beginning: a reliable pipeline also needs a manifest that records what each file means, what one row represents, and what has actually been validated.

## What I learned

The new [Philippine Government Budget Analysis repository](https://github.com/ItsYangCoder/ph-government-budget-analysis) defines a reproducible FY2024–FY2025 analysis across agencies, sectors, and regions. The related `INGESTION_MANIFEST.csv` is useful because it turns a folder of mixed Excel and PDF files into an explicit ingestion plan.

The manifest does not contain the budget observations themselves. It contains **control metadata** about the inputs:

- `source_id` gives every source a short stable identifier such as `A24`, `G25`, or `SEC`;
- `source_group` separates agency actuals, approved GAA, sector actuals, regional actuals, population, and development indicators;
- `fiscal_year` records the period that the data describes;
- `relative_path` tells the pipeline where the downloaded file should be;
- `file_format` determines whether the ingestion logic needs an Excel or PDF reader;
- `intended_measure` records whether the file contains appropriations, allotments, obligations, expenditure, population, or another measure;
- `analysis_grain` states what one output row should represent;
- `content_validation` records how far the source has been checked.

The most important distinction is between **file presence** and **content validation**. A file can be present in the landing folder while its worksheet, unit, identifiers, or reporting period are still unknown. Treating `file_present_in_drive = yes` as “ready for analysis” would be a false positive.

The current manifest makes that distinction concrete. The 2024 and 2025 agency actuals and GAA workbooks are present, but their workbook structure still needs checking. The sector source has stronger evidence: the manifest records its 2024 and 2025 columns, million-peso unit, and the difference between actual, program, and proposed years. The regional PDFs have confirmed headings but still need extraction and reconciliation. The population source is explicitly labeled as a projection rather than a census count.

This is also why measure and grain must be stored before writing transformations. Agency obligations at an `agency-year` grain cannot be appended blindly to regional obligations at a `department-region-year` grain. Sector expenditures from a fiscal-statistics table are not automatically equivalent to SAOB obligations. The values may all be numeric, but their business meanings are different.

A manifest can later drive a repeatable Bronze ingestion process:

1. select rows whose files are present;
2. route each file to the correct parser;
3. preserve the source identifier and original filename;
4. validate the expected sheet, columns, unit, and grain;
5. quarantine failures instead of loading ambiguous data;
6. update the validation status only after the checks pass.

For stronger reproducibility, I would eventually add `retrieved_at`, `file_checksum`, `schema_version`, and `expected_columns`. A checksum would prove whether the downloaded bytes changed even if the filename stayed the same.

![AQ1 and AQ2 mapped to official source families, measures, and analysis steps](../assets/2026-09-30-question-source-grain-map.png)

*This inspected screenshot shows how AQ1 and AQ2 are mapped to specific source families and calculations. I used it because it contains no credentials or personal information and it makes the relationship between a question, its measures, and its grain visible.*

## Terms I am still learning

- **Ingestion manifest** — a control table describing which source files exist, how to read them, and how far they have been validated.
- **Data contract** — an explicit agreement about a dataset’s structure, meaning, quality rules, and ownership.
- **Control metadata** — information used to operate and audit a pipeline rather than answer the business question directly.
- **Grain** — what one row represents, such as one agency in one fiscal year.
- **Measure semantics** — the exact business meaning of a number, including whether it is an appropriation, allotment, obligation, disbursement, or expenditure.
- **Checksum** — a value calculated from file bytes that can reveal whether a file changed.

## What confused me

I was confused about whether adding a source to the manifest meant that the source was already usable. It does not. The manifest can confirm that a file exists while still marking its internal validation as pending.

I also need to keep the sector measure separate from the agency measure. The sector workbook is described as actual cash-based national government expenditure, while the agency source contains allotments and obligations. I should not combine them under one generic `actual_spending` column unless I first prove that their definitions are compatible.

An open question is how strict the first schema contract should be for the regional PDFs. PDF tables can change layout across editions, so a parser that only checks column positions may be fragile even when the visible headings look similar.

## One small next step

Validate the `A24` workbook end to end: record the exact worksheet, header row, unit, agency identifier, required columns, and row grain, then change its status from `pending_workbook_check` only if every item is confirmed.

## Git checkpoint

- [x] I created or updated a journal file.
- [x] I wrote a commit containing the journal entry and inspected image.
- [x] I pushed the commit to `main`.
- [ ] I validated the internal schema and units of the `A24` workbook.

## Decisions or assumptions (optional)

- Use FY2024–FY2025 as the current repository scope because that is the period stated in its README.
- Keep official filenames and relative source folders unchanged in the raw landing area.
- Treat `file_present_in_drive` and `content_validation` as separate states.
- Keep agency, sector, regional, population, and development measures in separate typed datasets until their grains and semantics are reconciled.
- Do not describe pending sources as analysis-ready.

## Evidence from today (optional)

- [New PH Government Budget Analysis repository](https://github.com/ItsYangCoder/ph-government-budget-analysis)
- [Initial repository commit](https://github.com/ItsYangCoder/ph-government-budget-analysis/commit/4b8727654563431389fdb701332d7c70687f1649)
- [DBM SAOB source page](https://www.dbm.gov.ph/index.php/statement-of-appropriations-allotments-obligations-disbursements-and-balances)
- [DBM FY2024 GAA source page](https://www.dbm.gov.ph/index.php/2024/general-appropriations-act-gaa-fy-2024)
- The reviewed `INGESTION_MANIFEST.csv`, which currently lists eight source records with their measures, grains, paths, validation states, and ingestion notes.
- The inspected AQ1/AQ2 mapping screenshot embedded above.

## Reflection (optional)

A folder can look organized and still be unsafe to automate. The manifest makes uncertainty visible instead of hiding it: “downloaded,” “validated,” and “ready to analyze” are different states. That feels like a small documentation decision, but it is also pipeline logic and a protection against confident-looking wrong results.

## Mood or meme (optional)

**Downloaded is not the same as validated.**
