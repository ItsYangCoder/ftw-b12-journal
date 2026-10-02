# Journal - 2026-10-02 - A Name Match Is Not Yet a Safe Join

## Today in one sentence
- With no new PondoLens commit today, I used the current department-mapping boundary to study why a normalized text match is only a candidate for a join, not proof that two government records describe the same entity and scope.

## What I learned
- The current PondoLens source pipeline has a deliberate stopping point. It can preserve and validate the nine supplied files, publish passing source partitions, and expose filtered query views, but it does not publish an automatic cross-source join. The repository currently records **1,489,811 long-form rows**, **55,393 passing reconciliation checks**, and **zero duplicate keys at the implemented grains**. Those results establish that the implemented source contracts passed; they do not establish that agency actuals and GAA budget lines are comparable.
- The department-mapping script makes that boundary concrete. It reads distinct `(year, entity)` pairs from `agency_department_financials` and compares them with `(year, department_code, department)` pairs from `approved_budget`. Its normalization step removes a trailing abbreviation such as `(DEPED)`, applies case folding, and collapses whitespace. This is useful for finding likely matches without hiding the original labels.
- I learned to separate **candidate generation** from **entity resolution**. Candidate generation reduces the manual search space. Entity resolution is the decision that two labels refer to the same real-world organization for a defined period and purpose. In the recorded review run, normalization produced candidates for 67 of 74 year-and-department rows, while 7 had no match. Every row still remained `REVIEW_REQUIRED`, which is the correct conservative state.
- A matching name can still conceal a different analytical scope. One file may report actual obligations or cash expenditure for a department/fund grouping, while the GAA contains approved appropriations by department code and fund. Joining them because their labels look alike can create a clean-looking but invalid utilization ratio. The join needs agreement on identity, year, measure, fund coverage, and reporting level.
- The `year` key is part of the identity check, not just a filter. Government organizations can be renamed, reorganized, created, or moved between departments. A mapping that is correct for FY2024 may need a different canonical name or validity period for FY2025. This is why a durable crosswalk should record when a relationship is valid instead of treating names as timeless.
- The script also protects human work by refusing to overwrite an existing review CSV. That small rule matters: rerunning candidate generation should not erase approved mappings, scope decisions, or review notes. The crosswalk is governed data, not a disposable intermediate file.
- A safe Silver-layer mapping would keep the source label and candidate beside the approved canonical department code, review status, scope decision, match method, notes, and validity dates. Gold joins should accept only approved rows and should fail or quarantine ambiguous and unmatched records. In simple terms, the pipeline should make uncertainty visible instead of guessing.

## Terms I am still learning
- **Entity resolution** - deciding whether records with different or similar labels refer to the same real-world organization.
- **Crosswalk** - a reviewed table that translates source-specific labels or codes into a shared canonical identifier.
- **Candidate generation** - producing plausible matches for review without declaring them correct.
- **Temporal validity** - the period during which a mapping is true, often represented by effective start and end dates.
- **Semantic join gate** - a rule that blocks a cross-source join until identity, scope, grain, and time compatibility have been approved.

## What confused me
- A normalized exact name feels strong because it is deterministic, but it only proves that two cleaned strings are equal. It does not prove that their measures, fund boundaries, or organizational definitions are equal.
- I still need a clear policy for reorganized bodies and renamed departments. The open design question is whether one canonical department code should span a rename, or whether separate organization versions should be linked through a parent identity with validity dates.
- Another unresolved point is where review evidence should live. The generated CSV is excluded with local outputs, but approved mappings need version history and review notes. Committing a small approved crosswalk or maintaining it as a governed table would make changes auditable without committing source data.

## One small next step
- [ ] Review the seven `no_match` rows from the recorded mapping output and document, for each one, whether it is a true unmatched organization, a rename, a scope mismatch, or a source-label issue.

## Git checkpoint
- [x] I created or updated a file
- [x] I wrote a commit
- [x] I pushed my changes

## Decisions or assumptions (optional)
- I will treat normalized-name matches as suggestions only. No candidate becomes joinable until its organization identity and analytical scope are reviewed.
- I will keep `ready_for_cross_source_analysis` false while the department crosswalk and geographic boundaries remain unresolved.
- I will preserve original source names and codes alongside canonical values so a result can be traced back to the source.
- I am reusing the existing safe PondoLens project mark because no new relevant image was accessible today; I did not substitute an unrelated screenshot.

## Evidence from today (optional)
- [PondoLens status and next steps](https://github.com/ItsYangCoder/ph-government-budget-analysis/blob/dev/README.md) - reviewed cross-source mapping and joins are still marked pending.
- [Department mapping candidate generator](https://github.com/ItsYangCoder/ph-government-budget-analysis/blob/dev/scripts/build_department_mapping.py) - shows year-aware normalized matching, `REVIEW_REQUIRED`, scope fields, and overwrite protection.
- [Data contracts and cross-source limits](https://github.com/ItsYangCoder/ph-government-budget-analysis/blob/dev/docs/long_form.md) - explains why PASS source tables still need identity, fund, hierarchy, and geography review.
- [Recorded source validation](https://github.com/ItsYangCoder/ph-government-budget-analysis/blob/dev/docs/evidence/long_form_validation.md) - records the verified row counts, checks, tests, and the false cross-source-readiness flag.
- [Current PondoLens `dev` checkpoint](https://github.com/ItsYangCoder/ph-government-budget-analysis/commit/0187274fe61f478ab52d9606dcad87c148e4b0d1) - no newer commit was present during this review.

## Reflection (optional)
- The useful lesson for me is that a strong data pipeline should be able to say “not ready” precisely. Passing parsers, reconciliations, and duplicate checks are meaningful progress, but they answer different questions from “Can these sources be joined?”
- This boundary is not a weakness in the pipeline. It prevents a technically successful transformation from becoming a misleading analytical result. The next improvement is therefore not a more aggressive fuzzy matcher; it is a small, reviewable decision process with explicit scope and time rules.
- I want the eventual Gold model to be boring in a good way: every joined department should have an approved mapping, a defined period, and a traceable reason for inclusion.

## Mood or meme (optional)
![PondoLens project mark with a magnifying glass, peso symbol, and bar chart](../assets/2026-10-01-pondolens-project-mark.png)

The magnifying glass fits today's reflection: finding a likely name is the beginning of verification, not the end.
