# Journal - 2026-10-22 - Data Quality Needs Explainable Health Metrics

## Today in one sentence
- I learned that a useful data-quality dashboard should show what was checked, how many records passed, and what remains unassessed instead of hiding eleven different Silver datasets behind one “healthy” score.

## What I learned
- [PondoLens PR #100](https://github.com/ItsYangCoder/ph-government-budget-analysis/pull/100) added a live `dq_silver_health` monitoring view for all 11 Silver datasets. It reports actual numerators, denominators, and percentages for identity completeness, business-key uniqueness, measurement availability, primary-value validity, mapping, and supported expected-key coverage.
- The numerator and denominator matter because a percentage alone can hide scale. Completion-rate availability, for example, is 657 available measurements out of 855 expected observations, or about 76.84%. That is an availability metric, not a final quality grade. The remaining 198 observations include recognized source missingness and should not be silently converted to zero or described as invalid.
- Different checks answer different questions. **Identity completeness** asks whether required IDs exist. **Business-key uniqueness** asks whether the same real-world observation appears more than once. **Measurement availability** asks whether the value is present. **Validity** asks whether a present value is allowed. **Mapping coverage** asks whether source locations resolve to standard geography. A dataset can pass one check and still need attention in another.
- The decision not to create one combined health score is important. Averaging 100% key uniqueness, 76.84% availability, and 100% mapping would create a number that looks precise but mixes unrelated meanings. Operators need the individual checks so they can decide whether a result is expected source behavior, a warning, or a blocker.
- The monitoring design keeps legitimate cases separate: source markers, real zeros, missing expected keys, historical Negros Island Region exceptions, and the budget-only Nationwide mapping. This prevents a rule such as “all nulls are bad” from creating false alarms.
- [PondoLens PR #96](https://github.com/ItsYangCoder/ph-government-budget-analysis/pull/96) handles a different layer of observability. Its dbt `on-run-end` hook saves model and test outcomes into a separate monitoring schema with Delta history, run summaries, latest-node status, dataset summaries, and dashboard SQL. It preserves failed and skipped results and distinguishes a partial dbt selection from a complete run.
- This taught me the difference between **execution monitoring** and **data health monitoring**. Execution monitoring says whether dbt models and tests ran. Data health monitoring explains the condition of the resulting datasets. Both are needed: a command can succeed while a source is incomplete, and a failed command can leave previously valid tables unchanged.
- PR #100 recorded 77 passing offline tests and successful compilation of 19 models with 235 data-test definitions. At merge time, connected execution was still listed as pending. A later shared dashboard screenshot displayed a recorded build covering 11 datasets with 11 passed, 0 failed, and 0 warnings. I should treat that as evidence of the configured checks, not as independent proof that every official source is complete.
- The safe screenshot below shows why per-dataset reporting matters. The production catalog has 11 Silver tables feeding multiple Gold facts and dimensions. A single top-level green status would not tell me whether a problem came from completion rates, hospital counts, population, poverty, regional budgets, or the shared region crosswalk.

## Terms I am still learning
- **Observability** - the ability to understand what a data system did and what state its data is in using recorded evidence.
- **Numerator** - the number of records that satisfy a check.
- **Denominator** - the total number of records eligible for that check.
- **Expected-key coverage** - whether all independently expected business keys are present in the dataset.
- **False positive** - a quality alert that reports a problem even though the source condition is legitimate.
- **Unassessed** - a check that cannot yet make a defensible claim because no reliable independent reference exists.
- **On-run-end hook** - dbt logic executed after a command finishes, often used to record results or metadata.

## What confused me
- I still need to understand which dashboard statuses are blocking and which are informational. A lower availability percentage can be legitimate when the source publishes missing markers, while one duplicate business key may be serious enough to block downstream use.
- I also need to confirm how cancelled or abruptly terminated Databricks jobs appear. PR #96 notes that an end hook may not run after cancellation or early termination, so the monitoring tables and the Databricks job status must be read together.
- Some coverage checks remain intentionally unassessed because there is no independent reference for budget-detail completeness or crosswalk alias uniqueness. I need to avoid turning an unknown into a pass just to make the dashboard look complete.

## One small next step
- [ ] Recompute one dataset’s health metrics directly from its Silver table—starting with completion rate’s 657 available out of 855 observations—and compare the numerator, denominator, percentage, and status with `dq_silver_health`.

## Git checkpoint
- [x] I created or updated a file
- [x] I wrote a commit
- [x] I pushed my changes

## Decisions or assumptions (optional)
- I will keep each health dimension visible instead of calculating one combined quality score.
- I will preserve recognized source markers and legitimate zeros as separate states rather than treating them as generic failures.
- I will label unsupported coverage checks as unassessed, not passed.
- A green dashboard means the configured checks passed for the recorded run; it does not guarantee complete or independently verified source data.
- Monitoring tables should store useful result metadata without copying raw source records or sensitive error text.

## Evidence from today (optional)
- [PR #100 — Silver health metrics for the dashboard](https://github.com/ItsYangCoder/ph-government-budget-analysis/pull/100)
- [PR #96 — persisted dbt quality results for monitoring](https://github.com/ItsYangCoder/ph-government-budget-analysis/pull/96)
- [PR #100 merge commit](https://github.com/ItsYangCoder/ph-government-budget-analysis/commit/d684dff932dea289a8435fa2d1666990797e500f)
- ![Production Gold and Silver table inventory showing eight Gold tables and eleven monitored Silver datasets](../assets/2026-10-22-silver-health-coverage.png)
- This inspected screenshot from my shared project files shows the production table inventory that the health view covers. It contains no credentials, private URLs, account details, or personal information.
- I did not embed the newer dashboard screenshot because its browser chrome exposed a private Databricks workspace URL and account context. Its visible summary was used only as supporting text evidence.

## Reflection (optional)
- My main takeaway is that monitoring should make a result easier to question, not harder. Showing 657 out of 855 gives me something concrete to verify; showing only a green badge asks me to trust the implementation.
- The project is moving from “we ran tests” toward “we can explain what the tests measured.” That is a meaningful production-readiness improvement because future failures can be traced to a specific dataset and quality dimension.

## Mood or meme (optional)
- One green score is comforting; eleven explainable checks are useful. 🔎

