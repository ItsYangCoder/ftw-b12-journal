# Journal - 2026-10-24 - Shared Dimensions Connect Facts Without Changing Their Grain

## Today in one sentence
- While refining the PondoLens data-model animation, I learned that a dimensional model shows relationships and analytical context, not the physical direction in which data travels through the pipeline.

## What I learned
- PondoLens uses a **fact constellation** because Population and Regional Expenditure are different kinds of measurements with different row meanings. Population has one row per year, location, and age group. Regional Expenditure has one row per year, location, and budget particular. Keeping them as separate facts protects both grains.
- Year and Location are **shared dimensions**. Both facts store `year_id` and `location_id`, so they can describe measurements for the same period and place. In the presentation graphic, “Region” is the audience-friendly label; the current Gold table is `dim_location`, which also distinguishes region, national, Central Office, and Nationwide records.
- Age Group applies only to `fact_population`. Agency applies only to the expenditure side, although the current technical dimension is `dim_particular`, which includes departments, agencies, and other budget entries together with their category. The simplified labels help the audience, but I should know the more precise table names behind them.
- The relationship lines do not mean that dimension tables contain the measures or literally send records into a fact table during analysis. A fact row carries foreign keys that reference its dimensions. The dimensions answer questions such as *which year, which location, which age group, and which budget particular?* The facts hold measurements such as `population_count` and `obligation_amount`.
- I also learned why shared keys are not enough by themselves. Population contains multiple age-group rows for one region-year, while expenditure can contain multiple agencies or particulars for the same region-year. If I join the raw facts directly on Year and Region, each obligation can repeat across age groups and inflate totals.
- For a regional per-person comparison, I must first aggregate Population to one row per year and location by summing the expected non-overlapping age groups. I must separately aggregate obligations to the intended level, such as one selected agency per year and location. Only then should I join the prepared results through Year and Location.
- The animation can still use motion to explain the model, but the narration must say “these dimensions describe and connect the facts,” not “the data flows from these boxes.” Data flow belongs to the Python → Google Drive → Databricks or Bronze → Silver → Gold diagram.

![Simplified PondoLens constellation showing shared Region and Year dimensions connected to Population and Regional Obligations facts](../assets/2026-10-24-fact-constellation-relationships.jpg)

This clean presentation graphic shows the two facts and their related dimensions. It is intentionally simpler than the complete Gold model, which also contains the regional-indicator fact and indicator dimension.

## Terms I am still learning
- **Fact table** - a table that stores measurable events or values at a clearly defined grain.
- **Dimension table** - a table that gives descriptive context to facts, such as year, location, age group, or agency.
- **Foreign key** - a value in a fact row that refers to the identifier of a related dimension record.
- **Conformed dimension** - a shared dimension used consistently by more than one fact table; Year and Location play this role in PondoLens.
- **Fact constellation** - several fact tables connected through shared dimensions instead of forcing every measurement into one wide table.
- **Fan-out join** - a join that multiplies rows because both sides contain several records for the same join key.

## What confused me
- Arrows are useful in an animation, but an arrow can accidentally suggest data movement or a one-way dependency. In a dimensional model, the important message is the key relationship and the one-to-many cardinality, not a physical flow direction.
- The presentation says “Agency,” while the actual model uses `dim_particular`. I need to avoid implying that every particular is a government agency because the dimension can also contain funds and other classified budget entries.
- I still need to be careful when saying Region and Year are “common ground.” Matching keys make a join possible, but they do not prove that geographic boundaries are historically compatible. NIR and other boundary changes still require dataset-specific review.

## One small next step
- [ ] Add one speaker note under the model slide: “These lines show relationships, not pipeline flow. We aggregate each fact separately, then compare matching Year and Region records.”

## Git checkpoint
- [x] I created or updated a file
- [x] I wrote a commit
- [x] I pushed my changes

## Decisions or assumptions (optional)
- I will keep the presentation diagram focused on Population and Regional Obligations because those are the facts needed for the current per-person story; I will describe it as a simplified view rather than the entire Gold model.
- I will animate Region and Year as shared context for both facts, Age Group only for Population, and Agency only for Regional Obligations.
- I will not present the relationship animation as the ingestion sequence. The separate pipeline slide explains source preparation and Bronze–Silver–Gold processing.
- No newer PondoLens code commit was visible today, so this entry is a learning reflection based on the current documented model and today’s presentation work, not a claim that I changed the dbt models.

## Evidence from today (optional)
- [PondoLens data-model documentation](https://github.com/ItsYangCoder/ph-government-budget-analysis/blob/main/docs/data_model.md) - defines the fact constellation, each table’s grain, shared dimensions, and safe preparation before joining facts.
- [Population fact SQL](https://github.com/ItsYangCoder/ph-government-budget-analysis/blob/main/dbt/models/gold/fact_population.sql) - resolves Year, Location, and Age Group foreign keys and checks that mapping does not lose or multiply Silver records.
- [Regional expenditure fact SQL](https://github.com/ItsYangCoder/ph-government-budget-analysis/blob/main/dbt/models/gold/fact_regional_expenditure.sql) - resolves Year, Location, and Particular keys and rejects missing or multiplied dimension matches.
- [Today’s simplified model graphic](../assets/2026-10-24-fact-constellation-relationships.jpg) - a clean Library image inspected before use; it contains no account details, private URLs, credentials, or personal information.

## Reflection (optional)
- Fixing the animation made me look beyond whether the slide was visually attractive. A moving dot in the wrong direction can teach the wrong architecture, even when every box has the correct label.
- The easiest part was recognizing the shared Region and Year context. The harder part was explaining why that still does not make a direct fact-to-fact join safe.
- I now have a clearer mental model: dimensions describe, facts measure, grain protects totals, and aggregation makes comparisons possible.

## Mood or meme (optional)
- A good diagram should simplify the model without simplifying away the truth. 🧩
