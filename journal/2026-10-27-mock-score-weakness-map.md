# Journal - 2026-10-27 - A Mock Score Should Become a Weakness Map

## Today in one sentence
- My 39/45 practice result is useful evidence of progress, but its real value is showing me which Databricks decisions I still need to explain without relying on recognition or luck.

## What I learned
- A practice score is an aggregate, so it can hide the shape of my uncertainty. A correct answer that I guessed is not as reliable as a correct answer that I can justify. For the next review, I need to classify every wrong or uncertain item by domain, record why my choice was wrong, and write the rule that would let me answer a similar question with different wording.
- Today’s screenshots exposed three concrete areas worth reviewing. First, **triggered** and **continuous** pipeline modes describe execution behavior, not the dataset type: triggered mode processes available data and stops, while continuous mode keeps processing new data. Databricks now documents this under Lakeflow Spark Declarative Pipelines, although older reviewers may still use Delta Live Tables, `LIVE TABLE`, and `STREAMING LIVE TABLE` terminology. [Databricks pipeline modes](https://docs.databricks.com/aws/en/ldp/concepts/pipeline-mode)
- Second, a slow first query against a stopped SQL warehouse is a startup-latency problem. A larger warehouse or a wider scaling range addresses execution capacity or concurrency, but it does not directly solve cold startup. Databricks recommends serverless SQL warehouses for rapid startup and autoscaling, so the wording of the scenario matters as much as the feature name. [Databricks SQL warehouse types](https://docs.databricks.com/aws/en/compute/sql-warehouse/warehouse-types)
- Third, troubleshooting a slow notebook task begins with the specific job run and task details. The run page provides task status, duration, compute information, query history, and logs. This is more precise than opening the notebook alone because the problem belongs to one execution context. [Monitor Lakeflow Jobs](https://docs.databricks.com/aws/en/jobs/monitor)
- These questions connect to PondoLens even though I did not make a new repository change today. The project already uses Databricks, dbt, SQL, and CI, and PR #100 explicitly separates passing offline checks from connected Databricks execution. That is the same reasoning pattern the exam tests: identify what the evidence proves, then do not claim the next layer is complete. [PondoLens PR #100](https://github.com/ItsYangCoder/ph-government-budget-analysis/pull/100)

## Terms I am still learning
- **Confidence calibration** - checking whether my confidence matches my actual understanding; guessed-correct answers should still be reviewed.
- **Triggered pipeline** - a pipeline update that processes the data available for that run and then stops.
- **Continuous pipeline** - a pipeline that remains active and processes new data as it arrives, trading lower latency for ongoing compute use.
- **Cold-start latency** - the delay while stopped compute starts before a query can run; it is different from slow query execution after compute is ready.
- **Run context** - the exact job execution, task, compute, parameters, logs, and timing needed to diagnose what happened.

## What confused me
- The reviewer uses older Delta Live Tables names while current documentation uses Lakeflow Spark Declarative Pipelines. I still need to be comfortable mapping the old words to the current concepts without assuming every old syntax detail is still the recommended production pattern.
- My 39/45 score is 86.7%, but the number alone does not tell me whether the six misses are concentrated in one high-weight domain or spread across several topics. It also does not reveal how many correct answers were guesses.
- A classmate reported 55 items in 90 minutes, while the current preparation material describes 45 scored questions and allows additional unscored items. I should plan my pacing around the full timer and not depend on an exact visible item count.

## One small next step
- [ ] Retake one closed-book, 45-question mock in 90 minutes, then create a short error log for every wrong or guessed answer with four fields: domain, mistaken assumption, correct rule, and one PondoLens example.

## Git checkpoint
- [x] I created or updated a file
- [x] I wrote a commit
- [x] I pushed my changes

## Decisions or assumptions (optional)
- I will treat guessed-correct answers as review items instead of counting them as mastered.
- I will prioritize weak-topic review and a timed retake before the October 31 exam; I am not scheduling a review call right now.
- I am using current Databricks documentation as the source of truth while learning the older names that may still appear in practice questions.
- The image below is an existing safe repository screenshot. It illustrates a real Databricks deployment workflow, but it is not evidence of a new run today.

## Evidence from today (optional)
- Practice result discussed today: **39/45 (86.7%)**, with the Databricks Certified Data Engineer Associate exam scheduled for **October 31, 2026**.
- Today’s inspected review screenshots covered pipeline execution modes, SQL warehouse startup latency, and job-run troubleshooting. The original browser captures were not committed because they exposed a personal name and a game-session URL.
- Recent project evidence: [PondoLens PR #100](https://github.com/ItsYangCoder/ph-government-budget-analysis/pull/100) merged with 77 offline tests passing; its description still marks connected Databricks execution as unverified.
- Current project head inspected today: [`d684dff`](https://github.com/ItsYangCoder/ph-government-budget-analysis/commit/d684dff932dea289a8435fa2d1666990797e500f), dated October 22, 2026. I found no newer project commit to claim as today’s implementation work.

## Reflection (optional)
- The mock score made me nervous because the exam is close, but it also gave me a smaller and more useful problem than “review everything.” I can turn uncertainty into a list of specific rules and decisions.
- What felt easiest was recognizing concepts I have already seen in the project, especially validation, deployment steps, and the boundary between offline checks and connected execution.
- What felt difficult was distinguishing an answer I truly understand from one that only looked familiar. I want my next review to measure reasoning, not recognition.

## Mood or meme (optional)
- ![Successful Databricks deployment workflow showing setup, bundle validation, deployment, and a completed job run](../assets/ci-cd-databricks-dev-success.png)
- This existing screenshot reminds me that readiness is a chain of checks: setup, validation, deployment, execution, and evidence. One green result is useful, but it only proves the step that actually ran. 🧭
