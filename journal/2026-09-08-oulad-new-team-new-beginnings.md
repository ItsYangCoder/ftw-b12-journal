# New Dataset, New Team, Same Introvert

*Setting up OULAD, organizing the work, and learning how to start again with a new team.*

**September 8, 2026 · FTW Data Engineering Journal**

Today felt like two kinds of setup happening at once: building the foundation of our OULAD project and getting comfortable with a new group.

On the technical side, we worked on the repository structure, file instructions, documentation, branch names, and task assignments. On the personal side, may adjustment din. I miss my old teammates, but I'm slowly getting to know the new people I'll be learning with.

This entry captures that stage of the project: may structure and direction na, but we still have implementation, testing, and a lot of conversations ahead of us.

## 1. Giving everyone a clear starting point

![Databricks repository tree with the assessment_clean.sql implementation guide open](../assets/oulad-project-setup.png)

*Our project in Databricks, with the folder structure on the left and the assessment-cleaning guide on the right. The file explains what to build; it is not yet the finished transformation.*

This screenshot sums up a big part of today's work. Hindi lang kami gumawa ng folders and empty files—we added instructions so each member can understand their assigned task.

The open file, `assessment_clean.sql`, includes the suggested branch, purpose, source, target, grain, transformation rules, and completion checks. For example, it tells the owner to preserve the 11 unknown Exam dates and validate the expected 206 assessment rows.

I realized how useful that is when working with a new team. Kapag binuksan nila ang file, they shouldn't have to guess what I meant or where to start.

We're building the [OULAD Data Engineering Pipeline](https://github.com/ItsYangCoder/oulad-data-engineering-pipeline) using the Open University Learning Analytics Dataset. It contains student, course, assessment, registration, and Virtual Learning Environment (VLE) data.

Our planned flow is:

**Source CSV → Bronze / Raw → Silver / Clean → Gold / Mart → Analytics**

| Layer | Responsibility in our project |
|---|---|
| Source | Seven CSV files stored in a Databricks Volume. |
| `open_university.oulad_bronze` | Preserve source records and ingestion metadata. |
| `open_university.oulad_silver` | Clean values, convert types, and apply documented aggregation rules. |
| `open_university.oulad_gold` | Build dimensions, facts, and a supporting reporting view using dbt. |
| `open_university.oulad_quality` | Support the project's validation work. |
| Analytics and Metabase | Answer the business questions through queries and dashboards. |

Mas clear na sa akin why these responsibilities are separated. If a result looks wrong, we have specific places to investigate: the source, the cleaning logic, or the Gold joins.

### Where the work belongs

We organized the files around those responsibilities:

| Folder | What belongs here |
|---|---|
| `src/sql/00_setup/` | Initialization and source inspection. |
| `src/sql/01_raw/` | Bronze ingestion scripts. |
| `src/sql/02_clean/` | Silver transformations. |
| `dbt/models/` | Silver source declarations, Gold dimensions, facts, and reporting view. |
| `src/sql/04_analytics/` | Business analysis queries. |
| `tests/` | Source, Silver, Gold, and business validation SQL. |
| `dbt/tests/` | Automated dbt tests. |
| `docs/` | Source findings, assumptions, pipeline plan, and schema documentation. |
| `.github/workflows/` | CI/CD implementation guides. |

We also discussed `resources/` for deployment and job definitions when needed. Gold transformations belong in `dbt/models/` in the current committed structure.

My takeaway: a useful project structure should help people understand their work. The folder names matter, pero equally important ang instructions inside the files.

## 2. Understanding the data before writing the transformations

The setup screenshot shows the guides, but the rules inside them came from the source profiling we reviewed.

The seven source files had already been loaded during the initial setup. These are our recorded Bronze baselines:

| Bronze table | Recorded rows |
|---|---:|
| `assessment_raw` | 206 |
| `courses_raw` | 22 |
| `student_assessment_raw` | 173,912 |
| `student_info_raw` | 32,593 |
| `student_registration_raw` | 32,593 |
| `student_vle_raw` | 10,655,280 |
| `vle_raw` | 6,364 |

Seven files sounds manageable, tapos makikita mo na 10.6 million rows ang student VLE alone. Hahaha.

Bronze includes `ingestion_timestamp` and `ingestion_date` to record when the rows were loaded. The initial loading approach uses `CREATE TABLE IF NOT EXISTS ... USING DELTA AS SELECT ... FROM read_files(...)`.

An important distinction: creating the table only when it doesn't exist is not automatic incremental loading. Kailangan pa rin ng separate plan for new batches and safe reruns.

### Missing is not the same as zero

We found 173 missing scores, all in TMA records, and 11 missing assessment dates, all in Exam records.

Our agreed rules preserve those records and keep unknown values as SQL NULL. Hindi namin dapat gawing zero ang missing score or invent an Exam deadline just to fill a blank.

NULL scores are excluded from averages, while the records can still count toward submission or participation analysis. Some source placeholders also use `?`, so checking only for SQL NULL would miss part of the problem.

### Repeated keys need an explanation

The VLE profiling showed 10,655,280 source rows but only 8,459,320 unique daily interaction keys—a difference of 2,195,960 rows.

The daily key combines module, presentation, student, resource, and relative day. Our agreed plan is to group by that complete key and calculate `SUM(sum_click)`.

Hindi enough na basta mag-remove ng repeated rows. Keeping an arbitrary row could discard recorded clicks. We expect fewer Silver rows after aggregation, but the total clicks must still reconcile with typed Bronze values.

The 8,459,320 Silver rows are an expected output, not a claim that the transformation has already passed validation.

### Observations, decisions, and implementation are different

| Document | What I should write |
|---|---|
| Source assessment | What we observed, such as the 173 missing TMA scores. |
| Assumptions | Our agreed interpretation or treatment, such as preserving unknown scores as NULL. |
| Pipeline plan | How to implement the rule and validate the result. |

I used to mix these up. Ngayon, mas clear kung saan ilalagay ang finding, decision, and implementation steps.

### Model the meaning of the data

OULAD uses relative-day fields: day 0 means presentation start, and negative days can be valid. We shouldn't invent calendar dates without actual start dates, or turn an unknown date into day 0.

We also aligned the Gold scope with sir's instructions: five dimensions and exactly two facts.

- Dimensions: `dim_student`, `dim_course`, `dim_module_presentation`, `dim_date`, and `dim_demographics`.
- Facts: `fact_assessments` and `fact_vle_interactions`.

The supporting `vw_student_outcomes` view will keep the full enrollment population in reporting, including students without recorded activity. For enrollment-level summaries, the two facts must be aggregated separately before joining them. Otherwise, puwedeng ma-multiply ang rows and measures.

## 3. Turning the plan into visible team tasks

![OULAD Pipeline Progress board with Backlog, Ready, In progress, In review, and Done columns](../assets/oulad-project-board.png)

*The saved Project board snapshot shows 16 tasks in Backlog, 3 in Ready, and 1 in In review. No tasks are marked Done in this screenshot; these are captured statuses, not a live progress report.*

After organizing the files, we organized the work itself.

Each GitHub issue has an owner, relevant files, a suggested branch, dependencies, and completion checks. We also added P0, P1, and P2 priority labels so the team can see which tasks need attention first.

The screenshot shows why this helps. The course-cleaning task is already in review, while other tasks are still waiting or ready to be picked up. Mas madaling makita where we are kaysa puro updates scattered across messages.

Some suggested branches are `feature/clean-assessments`, `feature/clean-students`, `feature/clean-vle`, and `feature/build-dimensions`.

One task can include its transformation, tests, and related documentation on the same branch. Hindi kailangan ng bagong branch for every individual file.

Our target completion is **Thursday, September 10, 2026**. The board helps us track that goal, but moving a card alone doesn't prove the code works. We still need validation results and review before calling a task done.

### What is actually ready?

The foundation, recorded source profiling, documentation, file guides, and issue assignments are in place. The board snapshot shows some work moving into review, but the complete Silver layer, dbt Gold models, dashboards, and working CI/CD still need implementation or verification.

There are also documented naming mismatches between some Bronze loaders and checks. Kailangan naming align those with the actual Databricks tables before running from a fresh environment.

A complete folder structure is a starting point. Working code, accurate results, and repeatable runs are the next proof we need.

## 4. A new team, and mixed feelings

Honestly, I feel a little sad about being assigned to another team.

I miss my previous group. Nakapagpalagayan na kami ng loob, and we already knew how each person worked. We had that comfortable rhythm where asking questions and coordinating didn't feel awkward anymore.

Now we're back in the getting-to-know stage. May adjustment ulit, and I miss that familiarity.

But my new teammates told me something that made me happy: they had prayed that I would be assigned to their group.

Nakakataba ng puso, honestly. While I was still missing my old group, they were already happy to have me with them.

![AI-generated illustration of Rhea with four appreciative teammates around a shared data-engineering workspace](../assets/new-team-grateful-message-composite.png)

*An AI-generated illustration representing how welcomed I felt by my four new teammates—not a photo of our actual group.*

Our interactions still feel a little formal and professional. Nagpapalagayan pa kami ng loob, and I understand that. Being comfortable with new people takes time.

Then I messaged them and found out they were just shy around me too.

Meanwhile, I was also wondering how to become closer to them because I'm introverted. So apparently, pare-pareho lang pala kaming nahihiya. Hahaha.

I told them to keep asking me questions para we can learn together and get more comfortable with each other. I don't know everything either. Sometimes, explaining something or working through a question helps me understand it better too.

When I said I was introverted, they told me they were too.

Ayon, a group of introverts figuring out how to start the conversation.

I want us to feel comfortable saying “hindi ko gets,” asking for help, or sharing an idea. Hopefully, the more we work together, the more natural those conversations will become.

I can miss my old team and still look forward to getting close to this one. Hindi naman kailangang mawala agad ang lungkot before I can appreciate a new beginning.

## What I'm taking from today

The three images capture different parts of the same day: giving the project a clear structure, making team progress visible, and getting comfortable with the people behind the work.

Technically, I learned that setup involves decisions about grain, missing values, validation, and dependencies—not just creating folders. Personally, I learned that someone who seems very formal might just be shy too.

Next, we'll align the Bronze names, work through the assigned transformations, and validate the outputs. I'll also keep encouraging questions para we can learn together.

*The project has a starting point. So does the team.*

---

<details>
<summary>Project references</summary>

- [Pipeline plan](https://github.com/ItsYangCoder/oulad-data-engineering-pipeline/blob/main/docs/pipeline_plan.md)
- [Source profiling results](https://github.com/ItsYangCoder/oulad-data-engineering-pipeline/blob/main/docs/source_assessment.md)
- [Documented assumptions](https://github.com/ItsYangCoder/oulad-data-engineering-pipeline/blob/main/docs/assumptions.md)
- [Project issues](https://github.com/ItsYangCoder/oulad-data-engineering-pipeline/issues)

*Counts refer to the recorded current source batch. Screenshots show captured project states; planned outputs still need implementation and validation.*

</details>
