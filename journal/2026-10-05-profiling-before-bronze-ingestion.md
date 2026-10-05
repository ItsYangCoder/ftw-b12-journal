# Journal - 2026-10-05 - Profiling Before Bronze Ingestion

## Today in one sentence

I learned that preparing data for Bronze ingestion is not just checking whether a file opens; it is documenting enough physical and business context so extraction can happen without guesswork.

## What I learned

- Today's Bronze-ingestion preparation clarified the boundary between **profiling** and **transforming**. Profiling describes the source as it exists: filename, format, sheets or pages, header position, year coverage, units, row grain, totals, and unusual labels. Transformation changes that source into a new structure. I can profile a workbook deeply without rewriting its values or creating Silver logic too early.
- A good handoff acts like a small source contract. For every source, I should be able to answer: Where did it come from? Which original file is authoritative? How should Python open it? Which sheet or page contains the real table? What does one row represent? Which fields look like keys? Which totals are controls rather than observations? What uncertainty must remain visible?
- I need to keep three gates separate:
  1. **Readable** — Python can open the file and enumerate its structure.
  2. **Extractable** — I can preserve the original values and provenance in a repeatable output.
  3. **Analytically ready** — the grain, units, mappings, scope, and controls are understood well enough for comparison or aggregation.

  Passing the first gate does not prove the other two.
- The current PondoLens ingestion code reinforces this separation. It snapshots original bytes, calculates a SHA-256 hash, records workbook cells or PDF page/table candidates, and deliberately reports that analytical mappings are still pending. That means Bronze can be technically successful while a geography or measure still requires review.
- The screenshot is a concrete example. The value `18_regions_NIR_separate_IX_includes_Sulu` carries several decisions in one status field: the expected region count, separate treatment of NIR, and where Sulu is grouped. It is useful provenance, but it is not yet a join key. Before I compare budget, population, and development indicators, I still need to confirm that their geographic vintages use compatible boundaries.
- Preserving originals unchanged is more than a storage habit. It gives me a stable reference when parsing rules change, a checksum target for reproducibility, and a way to distinguish a source correction from a code correction. If I manually clean the only copy, I lose that audit trail.
- This also changes how I should treat blockers. A missing header row, an unexplained unit, or an uncertain regional convention should be recorded explicitly. Guessing may make the pipeline run, but it makes the result harder to defend.

## Terms I am still learning

- **Source profiling** — inspecting a file's structure and meaning before building permanent extraction rules.
- **Grain** — what one row represents, such as one agency-year-measure or one region-year-indicator.
- **Schema-on-read** — keeping raw data close to its source form and applying structure when it is read.
- **Provenance** — the information that connects an extracted value back to its original file, sheet or page, row, and parser version.
- **Control total** — a source-provided total used to check extraction; it should not automatically be added again as a detail row.
- **Geographic vintage** — the version of regional boundaries and classifications that applies to a dataset at a point in time.

## What confused me

- I am still working out when a compact field such as `geography_status` is enough and when it should become a separate mapping table with one explicit rule per region.
- The phrase "18 regions" sounds precise, but it does not by itself prove that two sources define those regions the same way. I still need evidence for how NIR, Region IX, Sulu, nationwide values, and Central Office records are handled in each source.
- I also need to be careful not to turn today's profiling notes into claims that extraction or reconciliation has already been completed. The source can be documented before the corresponding pipeline run exists.

## One small next step

- [ ] Create one compact source-profile record for the geography-bearing input that lists its original filename, file type, sheet or page, header row, year coverage, row grain, unit, control totals, geographic convention, and unresolved questions.

## Git checkpoint

- [x] I created or updated a file
- [x] I wrote a commit
- [x] I pushed my changes

## Decisions or assumptions (optional)

- I will keep original files unchanged and treat manual corrections as a separate, traceable step.
- I will not label a source "analysis-ready" merely because Python can open it.
- I will preserve uncertain geography as metadata or a blocker instead of silently normalizing it.
- I will use the existing repository's nine-source layout and current ingestion rules as implementation evidence, while treating today's team assignment and screenshot as preparation evidence rather than proof of a completed pipeline run.

## Evidence from today (optional)

![Cropped data profile showing repeated geography_status values for an 18-region convention](../assets/2026-10-05-bronze-source-profiling.png)

The inspected screenshot shows a repeated `geography_status` value documenting an 18-region convention, separate NIR treatment, and the placement of Sulu. It contains no credentials, private URL, local username, or personal workspace path.

- [Setup and nine-source input contract](https://github.com/ItsYangCoder/ph-government-budget-analysis/blob/dev/docs/setup.md)
- [Structural ingestion code that preserves originals and records provenance](https://github.com/ItsYangCoder/ph-government-budget-analysis/blob/dev/src/pondolens/ingestion.py)
- [Data contracts and ingestion rules](https://github.com/ItsYangCoder/ph-government-budget-analysis/blob/dev/docs/long_form.md)

## Reflection (optional)

- The practical lesson for me is that the handoff before coding is part of data engineering. Clear source notes reduce the temptation to hide assumptions inside Python.
- The difficult part is not opening Excel or PDF files. It is deciding what must remain raw, what can be standardized later, and what evidence is required before different sources can be joined.
- Next time, I want to make the boundary between "extracted correctly" and "safe to analyze" visible in both documentation and pipeline status.

## Mood or meme (optional)

- Bronze lesson for today: copy the bytes, carry the context. 🥉
