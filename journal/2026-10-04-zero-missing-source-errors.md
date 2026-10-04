# Journal - 2026-10-04 - Zero Is Not Missing Data

## Today in one sentence
- With no new Data Engineering commit today, I studied an existing PondoLens rule that protects meaning: zero, blank, source dash, invalid text, missing formula output, and source errors must not be cleaned into the same value.

## What I learned
- A zero is a real observation. It can mean that an amount, count, or rate was measured and the result was exactly zero. A blank means that the source cell did not contain a value. Replacing a blank with zero creates evidence that the publisher never supplied.
- A dash is also not automatically zero. In government PDF tables, a dash may mean not applicable, not reported, no value presented, or a publisher-specific convention. PondoLens preserves it as `source_dash` so its meaning can be reviewed instead of silently guessed.
- The shared converter makes these states explicit. Its numeric parser returns `missing` for `None` or an empty string, `source_dash` for `-`, `–`, or `—`, `source_excel_error` for values beginning with `#`, and `invalid_numeric` for text that does not follow an accepted numeric pattern. Only valid finite numbers receive the `ok` status.
- The parser intentionally validates comma grouping. It accepts values such as `1,234.50`, but it does not remove arbitrary characters and hope the result is a number. This prevents a label, malformed value, or copied note from becoming an incorrect amount.
- PondoLens also uses Python's `Decimal` for monetary parsing. This is important because binary floating-point values can introduce tiny rounding differences. For financial source data, decimal arithmetic makes the stored value and reconciliation behavior more predictable.
- The record builder keeps both the parsed fields and the source evidence. It stores `raw_value`, `original_formula`, `value`, `value_php`, and `value_status`. If the source is not a valid numeric observation, the parsed value remains empty while the original content and status remain traceable.
- The verified agency workbooks contain **127 original `#DIV/0!` cells**: 66 in the 2024 file and 61 in the 2025 file. They are preserved as undefined rates with raw source errors, not converted to zero. This is mathematically correct because division by zero does not produce a zero rate.
- Treating those 127 errors as zero would bias an average or utilization summary. A dashboard might make the affected rows look like poor performance when the real issue is an undefined denominator. Excluding them without a count would also hide a source-quality condition. The safer design is to keep the status, decide whether the row is analytically eligible, and report how many observations were excluded.
- The same principle applies to cached Excel formulas. A formula cell may exist while its cached result is missing. PondoLens records `missing_formula_cache` instead of pretending the formula evaluated successfully. A pipeline that cannot calculate Excel formulas should be honest about what it received.
- I learned that missingness is part of the data contract, not just a cleanup inconvenience. A Silver table can expose standardized statuses, while a Gold metric can define which statuses are accepted in its numerator, denominator, and completeness calculation.
- A useful metric should therefore report both its result and its coverage. For example, an agency utilization average is more trustworthy when it also shows the number of eligible observations, undefined rates, missing values, and source errors.

## Terms I am still learning
- **Missingness semantics** - the specific meaning of why a value is absent or unusable, rather than treating every empty-looking cell as the same condition.
- **Source state** - a label such as `ok`, `missing`, `source_dash`, or `source_excel_error` that describes what the source actually contained.
- **Undefined rate** - a ratio that cannot be calculated because its denominator is zero or unavailable.
- **Formula cache** - the last calculated result stored inside an Excel workbook for a formula cell.
- **Analytical eligibility** - a rule indicating whether a retained source record may be used in a specific analysis.
- **Coverage metric** - a count or percentage showing how much of the expected data was usable for a calculation.

## What confused me
- A dash can look like a simple visual substitute for zero, especially in a published table. However, the symbol alone does not prove its business meaning. I still need source-specific documentation or reconciliation evidence before translating it into a number.
- I used to think that replacing missing numeric values with zero made downstream SQL easier. It does make the column easier to aggregate, but it changes the question from “What did the source report?” to “What value did the pipeline assume?”
- There is also a difference between retaining an error and allowing it into a KPI. The raw and long-form layers should preserve the error for lineage, while analytical views may exclude it. The exclusion must remain visible through a status count or quality report.
- I still need to decide how the future dashboard should display incomplete metrics. A blank result, warning badge, coverage percentage, or separate quality panel can each be correct in different situations. The rule should be consistent and understandable to a non-technical viewer.

## One small next step
- [ ] Define one status-summary query grouped by dataset and `value_status`, including total rows, usable numeric rows, source dashes, missing values, formula-cache gaps, and source Excel errors.

## Git checkpoint
- [x] I created or updated a file
- [x] I wrote a commit
- [x] I pushed my changes

## Decisions or assumptions (optional)
- I will not replace blanks, dashes, undefined rates, or source errors with zero unless the publisher explicitly defines that interpretation.
- I will preserve the original raw value and formula beside the parsed value and status.
- I will let Gold metrics accept only documented statuses and will show excluded-state counts when they affect coverage.
- I will treat an empty parsed value as “not a usable number,” not as proof that the amount is zero.
- No new relevant screenshot was accessible today, so I reused the existing safe PondoLens project mark after inspecting it.

## Evidence from today (optional)
- [Shared typed-value parser](https://github.com/ItsYangCoder/ph-government-budget-analysis/blob/dev/src/pondolens/converters/common.py) - implements the separate statuses for missing values, source dashes, Excel errors, invalid numerics, and missing formula caches.
- [Data contracts and ingestion rules](https://github.com/ItsYangCoder/ph-government-budget-analysis/blob/dev/docs/long_form.md) - states that zero, blank, source dash, and undefined rate are distinct.
- [Recorded source validation](https://github.com/ItsYangCoder/ph-government-budget-analysis/blob/dev/docs/evidence/long_form_validation.md) - records the 127 preserved agency `#DIV/0!` cells and explicit PDF nonnumeric states.
- [PondoLens project status](https://github.com/ItsYangCoder/ph-government-budget-analysis/blob/dev/README.md) - identifies the current source ingestion and validation scope.

## Reflection (optional)
- The useful lesson for me is that cleaning data should remove accidental mess without removing information. Turning every unusual value into zero produces a tidy column, but the result is less truthful.
- Preserving source states also makes debugging more human. Instead of seeing a blank output and wondering what happened, I can trace whether the source was empty, used a dash, contained an Excel error, or lacked a cached formula result.
- I want the future PondoLens dashboard to make uncertainty understandable rather than invisible. A smaller metric with clear coverage is more useful than a complete-looking metric built from invented zeroes.

## Mood or meme (optional)
![PondoLens project mark with a magnifying glass, peso symbol, and rising bar chart](../assets/2026-10-01-pondolens-project-mark.png)

The inspected project mark fits this reflection because the magnifying glass represents checking what a value actually means before using it in an analysis.
