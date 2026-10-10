# Journal - 2026-10-10 - Proposed Silver Rules Need Evidence

## Today in one sentence

I learned that a profiling finding becomes a trustworthy Silver rule only after the source meaning is verified, the transformation is implemented, and a test proves the expected behavior; writing the rule down is an important step, but it is not execution evidence.

## What I learned

- Today's real repository evidence is a consolidation document on the PondoLens `dev` branch. It brings the team's profiling results together and turns them into candidate rules for the Silver layer. The document deliberately labels the rules **PROPOSED** and the unresolved checks **PENDING**. It also states that no Databricks execution or source verification has been completed for this consolidation yet.
- That status language matters. A finding describes what profiling observed. A proposed rule describes how the pipeline may respond. An implemented rule is transformation code that actually performs that response. A verified rule has test or run evidence showing that the behavior works on known cases. These are different stages, and collapsing them would make the documentation sound more complete than the pipeline really is.
- I can express the progression as: **finding → proposed treatment → pending source check → implemented transformation → executable test → approved rule**. The document currently strengthens the first three stages. It does not claim the last three.
- The consolidated findings show why one generic `fillna(0)` step would be unsafe. In `agency_actuals`, the profiling found 809 dash placeholders, 294 NULL amounts, 494 NULL expense classes, 70 NULL particulars, and 12 rate values that could not be parsed. Those values may represent different conditions. A true zero, a blank cell, a printed dash, a structural NULL, and an invalid numeric string should not all become the number zero.
- A useful Silver pattern is to keep the typed value and add an explicit status such as `valid`, `missing`, `source_placeholder`, `not_applicable`, `invalid_numeric`, or `review_required`. This preserves meaning while still giving analytics a predictable column type.
- The same principle applies to comparisons. Only 102 of 1,008 agency groups produced a class-total comparison in the profiling review. If a difference is NULL because one side is missing, that is not a passing comparison. A quality check should distinguish **equal**, **different**, and **not comparable** instead of treating NULL as zero or success.
- Unit normalization also needs evidence. The proposal suggests preserving the source amount, original unit, and conversion factor, then converting thousands or millions only after the source unit is verified. An unknown or conflicting unit should block an affected peso metric. Multiplying first and documenting later could create a clean-looking but materially wrong result.
- The proposed numeric types also carry a lesson. Amounts may use `DECIMAL(38,12)`, percentages `DECIMAL(18,8)`, counts `BIGINT`, and years `INT`, but a declared type is not a proof that every source value fits. Precision, scale, parsing, and overflow still need tests with actual values.
- Bronze and Silver have different responsibilities. Bronze should retain the source representation and ingestion lineage. Silver can standardize names, types, units, statuses, and business grain, but it should still point back to the raw value, source path, source row or page, revision, original unit, Bronze ingestion time, and transformation flags.
- I also learned that a **publication blocker** is broader than a failed technical job. The pipeline may run successfully while a metric remains unsafe to publish because its unit, hierarchy, denominator, revision, stage, scope, cutoff date, or geography boundary is unresolved.
- The geography example makes this concrete. The profiling image records an explicit assumption about 18 regions, NIR being separate, and Region IX including Sulu. That assumption belongs in lineage and review metadata until a compatible crosswalk is verified. A successful name join would not prove that population, GRDP, and spending describe the same boundaries.
- This connects to my ETL and ELT review. PondoLens is not explained well by choosing only one label. Some source-specific reshaping can happen before data reaches Bronze, while the more important semantic standardization is designed after loading, during Bronze-to-Silver work. What matters most is not the acronym; it is knowing which guarantees exist at each boundary and where the evidence for those guarantees lives.
- The strongest takeaway is that documentation can be a quality gate when it preserves uncertainty. Words such as `PROPOSED`, `PENDING`, and `BLOCKED` are not signs of weak work. They prevent an assumption from quietly becoming production logic.

## Terms I am still learning

- **Profiling finding** — an observed fact about the current data, such as a count of dashes, NULLs, duplicates, or failed parses.
- **Proposed rule** — a candidate treatment derived from a finding; it is not yet proof that the treatment is implemented or correct.
- **Structural NULL** — an empty value caused by the shape or hierarchy of the source rather than an accidental missing measurement.
- **Value status** — a companion field that explains why a typed value is present or absent, for example `valid`, `source_placeholder`, or `invalid_numeric`.
- **Lineage** — metadata that lets me trace a Silver record back to its source file, location, revision, raw value, and transformation.
- **Publication blocker** — an unresolved condition that prevents a metric or dataset from being presented as decision-ready even if the code ran.
- **Data contract** — an agreed definition of a dataset's grain, schema, meaning, quality rules, and change expectations.
- **Three-valued comparison** — treating a check as pass, fail, or not comparable instead of forcing missing evidence into pass or fail.

## What confused me

- My first instinct was to read the consolidated rules as the team's final Silver specification because they are detailed and consistent. The document itself corrects that interpretation: every rule remains proposed until the listed source and execution checks pass.
- I still need a clear approval mechanism. A Markdown status is helpful, but I am not yet sure whether a rule should become approved through a reviewed pull request, a test manifest, a data-contract version, or a combination of those artifacts.
- Dashes remain context-dependent. A dash can mean unavailable, not applicable, intentionally blank, or a visual placeholder. I cannot decide its semantic status from the character alone; the workbook legend, neighboring rows, and source-specific convention matter.
- The regional GAA source may use amounts in thousands, but applying a `×1000` factor before the unit note is verified would turn an assumption into data. I need the source evidence and at least one known-value test before accepting that conversion.
- Geography is still the hardest cross-dataset issue. If one dataset separates NIR and another uses an older grouping, neither a string cleanup nor a successful join makes their regional metrics comparable.

## One small next step

- [ ] Review one high-impact proposed rule end to end: verify the regional GAA unit from the source, record the evidence, define one known input and expected normalized amount, and add an executable test before changing that rule from `PROPOSED` to `APPROVED`.

## Git checkpoint

- [x] I created or updated a file
- [x] I wrote a commit
- [x] I pushed my changes

## Decisions or assumptions (optional)

- I will keep Bronze values unchanged and make Silver transformations traceable back to them.
- I will not replace blanks, dashes, structural NULLs, and invalid numeric text with one default value.
- I will use an explicit status field when the typed value alone cannot preserve source meaning.
- I will treat a NULL comparison as **not comparable**, not as a passed equality check.
- I will preserve the original amount, original unit, and conversion factor alongside a normalized amount.
- I will block affected metrics when the unit, denominator, hierarchy, or geography boundary is unresolved.
- I will keep rules labeled `PROPOSED` until source evidence, implementation, and an executable test are available.
- I will not describe today's consolidation commit as a completed Databricks run or an implemented Silver pipeline.

## Evidence from today (optional)

![Bronze profiling output showing an explicit geography-status assumption for 18 regions, NIR, and Sulu](../assets/2026-10-05-bronze-source-profiling.png)

I reused this previously inspected repository image because today's newly accessible images were interview-preparation screenshots rather than technical pipeline evidence. The profiling output is directly relevant: it shows a source assumption that should remain visible in lineage and validation instead of being silently converted into a Silver guarantee. The image contains no credentials, private URLs, personal account details, or local file paths.

- [Today's consolidation commit on the PondoLens `dev` branch](https://github.com/ItsYangCoder/ph-government-budget-analysis/commit/ce79570abd66bb751f6b288ceab7965d82b84f91)
- [Profiling consolidation review](https://github.com/ItsYangCoder/ph-government-budget-analysis/blob/dev/docs/profiling_consolidation_review.md)
- [Reused safe profiling image in this journal repository](https://github.com/ItsYangCoder/ftw-b12-journal/blob/main/assets/2026-10-05-bronze-source-profiling.png)

## Reflection (optional)

- The useful work today was not pretending that the Silver layer was finished. It was making the gap between observation, decision, code, and evidence easier to see.
- I used to think a detailed rule list was close to implementation. Now I see that the detail mainly improves the handoff: it gives the next reviewer or developer something precise to verify and test.
- I also see why data engineering quality is partly about language. A status word can protect users from a false claim just as a failed test can protect the pipeline from bad code.
- The consolidation gives the project a stronger direction without overstating readiness. That feels like a more honest kind of progress: the unknowns are smaller, named, and connected to concrete checks.
- What I want to understand better next is how to represent proposed and approved rules in a versioned data contract so documentation, code, tests, and release decisions cannot drift apart.

## Mood or meme (optional)

- Documentation: “Here is the rule.” Evidence: “Show me the row.” 🔎
