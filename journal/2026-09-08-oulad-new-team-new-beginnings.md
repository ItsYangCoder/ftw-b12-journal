# New Dataset, New Team, Same Introvert

*Setting up OULAD and getting comfortable with a new team.*

**September 8, 2026 · FTW Data Engineering Journey**

> **Today's mood:** productive, a little sentimental, pero looking forward to this new team.

Today, we worked on the OULAD project foundation: folder structure, documentation, file instructions, branch names, and task assignments. Binalikan din namin ang source profiling para clear sa bawat member kung anong data at rules ang hawak nila.

While setting up the project, I'm also adjusting to a new team. New dataset, new workflow, and new people to work with. Sabay ang technical setup at getting-to-know stage namin today. Hahaha.

![OULAD project setup](../assets/image%20(1).png)

---

## Setting up the project foundation

We're building the [OULAD Data Engineering Pipeline](https://github.com/ItsYangCoder/oulad-data-engineering-pipeline) using the **Open University Learning Analytics Dataset**. Kasama sa data ang students, module presentations, assessments, registrations, and activities in the Virtual Learning Environment or VLE.

Our goal is to build a reliable pipeline for analyzing assessment performance, withdrawal, demographics, and student engagement. Kailangan understandable ang flow para kaya naming i-explain kung saan nanggaling ang results.

### 1. Defined the layers and their responsibilities

We organized the project around this flow:

**Source CSV → Bronze / Raw → Silver / Clean → Gold / Mart → Analytics**

| Layer | Project responsibility |
|---|---|
| Source | Seven CSV files stored in the Databricks Volume. |
| `open_university.oulad_bronze` | Original source records with ingestion metadata. |
| `open_university.oulad_silver` | Planned cleaning, type conversion, and aggregation using documented rules. |
| `open_university.oulad_gold` | Planned dimensions, facts, and reporting view using dbt. |
| `open_university.oulad_quality` | Project validation work. |
| Analytics + Metabase | Planned queries and dashboards for the business questions. |

The reason for separating layers makes more sense to me now. Kapag may unexpected number sa dashboard, we can trace it back: source issue ba, cleaning rule, or a join in Gold?

### 2. Reviewed the Bronze baseline

The seven source files had already been loaded during setup. Binalikan namin these recorded Bronze counts as the baseline for the next steps:

| Bronze table | Recorded rows |
|---|---:|
| `assessment_raw` | 206 |
| `courses_raw` | 22 |
| `student_assessment_raw` | 173,912 |
| `student_info_raw` | 32,593 |
| `student_registration_raw` | 32,593 |
| `student_vle_raw` | **10,655,280** |
| `vle_raw` | 6,364 |

Each Bronze table includes `ingestion_timestamp` and `ingestion_date`. These record when the data was loaded, kaya may ingestion history kami at the row level.

Our initial ingestion approach uses `CREATE TABLE IF NOT EXISTS ... USING DELTA AS SELECT ... FROM read_files(...)`. One thing I learned: **creating a table if it doesn't exist is not automatic incremental loading**. Kailangan pa rin naming define how to handle new batches and reruns.

Also, **10.6 million rows** sa student VLE alone. Seven files sounds manageable until you see the actual row counts. Hahaha.

### 3. Made the repository easier to work with

We replaced simple placeholders with file guides. Para when a member opens their assigned file, may starting point na sila.

Each guide explains the purpose, input, expected output, transformation steps, completion checks, and suggested branch name.

| Folder | Contents |
|---|---|
| `src/sql/00_setup/` | Initialization and source inspection. |
| `src/sql/01_raw/` | Bronze ingestion scripts. |
| `src/sql/02_clean/` | Silver transformation code. |
| `dbt/models/` | Gold dimensions, facts, sources, and reporting view. |
| `src/sql/04_analytics/` | Business analysis queries. |
| `tests/` | Source, Silver, Gold, and business validation SQL. |
| `dbt/tests/` | Automated dbt checks. |
| `docs/` | Source findings, assumptions, pipeline plan, and schema documentation. |
| `.github/workflows/` | CI/CD implementation guides. |

We also discussed `resources/` for deployment and job definitions when needed. Sa current committed structure, Gold work lives in `dbt/models/`.

My takeaway here: having a file doesn't automatically make the task clear. Mas helpful kapag may expected output and specific completion criteria, especially while we're still learning the workflow.

---

## What I learned from the data

### Missing value does not automatically mean zero

We found **173 missing scores**, all from TMA records. Hindi puwedeng basta zero ang ipalit because an unknown score and an actual zero mean different things.

Our agreed treatment is to preserve the records, convert missing placeholders to SQL NULL, and exclude NULL scores from averages. Kasama pa rin sila when counting submissions or participation.

There are also **11 missing assessment dates**, all from Exam records. We keep those unknown deadlines as NULL; wala kaming reliable date na puwedeng ipalit.

Some source values use `?` for missing information. Kaya checking only for SQL NULL isn't enough—we need to inspect the source representations too.

### Separating findings, decisions, and implementation

| Document | Question it answers | Example |
|---|---|---|
| Source assessment | What did we observe in the data? | 173 missing TMA scores. |
| Assumptions | What interpretation or treatment did we agree on? | Unknown scores stay NULL. |
| Pipeline plan | How will we implement and validate it? | Normalize placeholders, preserve records, validate scored/missing counts. |

I used to mix these up. Ngayon, mas clear na sa akin where to document an observation, a decision, and the steps needed to implement it.

### Understanding repeated keys before removing rows

The student VLE profiling gave us another important finding:

| Student VLE profiling | Rows |
|---|---:|
| Bronze source rows | 10,655,280 |
| Unique daily interaction keys | 8,459,320 |
| Excess rows over those unique keys | 2,195,960 |

The daily key combines module, presentation, student, resource, and relative day. Kailangan complete ang combination to identify the intended daily interaction.

Our agreed plan is to group records with the same key using **`SUM(sum_click)`**. Kapag arbitrary row lang ang itinira, we could lose recorded clicks.

The expected Silver count is **8,459,320**, while total clicks must still match the typed Bronze values. **Planned output pa ito; implementation and validation are still pending.**

This helped me understand grain better: *what exactly does one row represent?* Kailangan clear iyon before deciding how to handle repeated keys.

### Working with relative days

OULAD has date fields expressed as days relative to the presentation start. Day 0 means the start, and negative days are valid kapag before the presentation.

We shouldn't invent calendar dates without a supplied start date. Hindi rin puwedeng gawing day 0 ang unknown date because zero already has a specific meaning.

### Keeping the model aligned with the requirements

We aligned the Gold scope with sir's instructions: **five dimensions and exactly two facts**.

- Dimensions: `dim_student`, `dim_course`, `dim_module_presentation`, `dim_date`, `dim_demographics`.
- Facts: `fact_assessments` and `fact_vle_interactions`.

We also planned a supporting **`vw_student_outcomes` view**. This keeps the complete enrollment population in reporting, kasama ang students without recorded assessment or VLE activity.

Another lesson: aggregate the two facts separately before combining them for enrollment-level reporting. Kapag directly joined ang detailed facts, multiple matches can inflate row counts, scores, or clicks.

---

## Organizing the team workflow

We organized the GitHub issues with an owner, assigned files, suggested branch, dependencies, and a completion checklist. May **P0, P1, and P2** priority labels, plus area labels para easier to identify the type of work.

Our target completion is **Thursday, September 10, 2026**.

Some of the suggested branch names:

- `feature/clean-assessments`
- `feature/clean-students`
- `feature/clean-vle`
- `feature/build-dimensions`
- `docs/update-readme`

I also learned that **one task can include several related files**. The transformation, tests, and related documentation can stay on the same task branch. Hindi kailangan ng separate branch for every file.

I wanted each task to give the assigned member a clear starting point. Para alam nila what to work on, which dependencies to wait for, and what evidence they need before marking it done.

<!-- Add Rhea's supplied GitHub Issues or Project board screenshot here after upload. -->

### Honest progress check

**The foundation, recorded source profiling, documentation, file guides, and issue assignments are ready.** Pending pa ang Silver implementation, dbt Gold models, dashboards, and working CI/CD.

There are also documented naming mismatches in some Bronze loaders and checks. Kailangan naming align those with the actual Databricks tables before running from a fresh environment.

Reminder to myself: a complete folder structure is only the starting point. Kailangan pa rin ng working code, validated results, and repeatable runs.

---

## A new team, and mixed feelings

Honestly, I feel a little sad about being assigned to a new team again.

I miss my previous group. Nakapagpalagayan na kami ng loob, and we already knew how each person worked. We had that comfortable rhythm where asking questions and coordinating felt natural.

Now, we're back in the getting-to-know stage. May adjustment ulit, and I miss the familiarity of working with people I was already comfortable with.

But my new teammates made me smile. They told me **they prayed that I would be assigned to their group.**

Nakakataba ng puso, honestly. While I was still missing my old group, they were already happy to have me on their team. That made this transition feel a little easier.

Our interactions still feel a bit formal and professional for now. Nagpapalagayan pa kami ng loob, and I understand that. Being comfortable with a new group takes time.

Then I messaged them and found out that **they were just shy around me too.**

Meanwhile, I was also trying to figure out how to become closer to them because I'm introverted. So apparently, pare-pareho lang pala kaming nahihiya. Hahaha.

I told them to keep asking me questions para we can learn together and become more comfortable with each other. I don't know everything either. Sometimes, explaining something or discussing a question helps me understand it better too.

When I told them I'm introverted, they said they were too.

Ayon, a group of introverts trying to get comfortable with one another. At least now we know why everyone seemed a little reserved. Hahaha.

<!-- Add Rhea's supplied personal/team photo here if she chooses to include one. -->

I want us to feel comfortable saying “hindi ko gets,” asking for help, or sharing an idea. Hopefully, as we work through the project, those conversations will start to feel more natural.

I can miss my old team and still look forward to getting close to this one. May lungkot pa, but I'm also glad that my new teammates were honest about how they felt.

---

## What I'm taking from today

Today helped me understand how many decisions go into setting up a project: table grain, missing values, folder responsibilities, validation, and task dependencies. Mas clear na sa akin why these need to be discussed before everyone starts coding.

On the personal side, I learned that someone who seems very formal might just be shy too. Minsan, kailangan lang may unang mag-open ng conversation.

Next, we'll align the Bronze names, implement the assigned transformations, and validate the outputs. As a teammate, I'll keep encouraging questions para we can learn together and get more comfortable working as a group.

*OULAD foundation prepared. Getting comfortable with the team is still a work in progress, pero we're getting there.*

---

<details>
<summary>Project references</summary>

- [Pipeline plan](https://github.com/ItsYangCoder/oulad-data-engineering-pipeline/blob/main/docs/pipeline_plan.md)
- [Source profiling results](https://github.com/ItsYangCoder/oulad-data-engineering-pipeline/blob/main/docs/source_assessment.md)
- [Documented assumptions](https://github.com/ItsYangCoder/oulad-data-engineering-pipeline/blob/main/docs/assumptions.md)
- [Assigned project issues](https://github.com/ItsYangCoder/oulad-data-engineering-pipeline/issues)

*Technical counts refer to the recorded current source batch. Planned outputs still need implementation and validation.*

</details>

