# 🌱 New Dataset, New Team, Same Introvert
### OULAD setup, data discoveries, at ang plot twist na pare-pareho pala kaming nahihiya

**September 8, 2026 · FTW Data Engineering Journey**

> **Today's mood:** productive, medyo sentimental, at unti-unting nae-excite sa bagong simula. 💛

Ngayong araw, inayos namin ang foundation ng OULAD project: folder structure, documentation, instructions sa bawat file, branch names, at paghahati ng tasks. Binalikan din namin ang source profiling para malinaw kung ano talaga ang hahawakan ng bawat isa.

Pero habang sine-set up ko ang project, may isa pa pala akong ina-adjust: sarili ko sa bagong team. Bagong dataset, bagong workflow, bagong mga kausap. Parang sabay na project setup at social setup ang ganap ko today. Hahahaha.

<!-- Add Rhea's supplied OULAD setup screenshot here after upload. -->

---

## 🧱 Paano namin binuo ang foundation

Ang project namin ay [OULAD Data Engineering Pipeline](https://github.com/ItsYangCoder/oulad-data-engineering-pipeline), gamit ang **Open University Learning Analytics Dataset**. May data tungkol sa students, module presentations, assessments, registrations, at activities sa Virtual Learning Environment o VLE.

Ang goal: makabuo ng maayos na data pipeline na puwedeng gamitin para pag-aralan ang assessment performance, withdrawal, demographics, at student engagement.

### 1. Inayos ang layers at responsibilities

Ito ang flow na sinusundan ng project:

**Source CSV → Bronze / Raw → Silver / Clean → Gold / Mart → Analytics**

| Bahagi | Gamit sa project namin |
|---|---|
| Source | Dito nanggagaling ang seven CSV files sa Databricks Volume. |
| `open_university.oulad_bronze` | Original source records, kasama ang ingestion metadata. |
| `open_university.oulad_silver` | Planned cleaning, type conversion, at aggregation ayon sa documented rules. |
| `open_university.oulad_gold` | Planned dimensions, facts, at reporting view gamit ang dbt. |
| `open_university.oulad_quality` | Para sa validation work ng project. |
| Analytics + Metabase | Planned queries at dashboards para sagutin ang business questions. |

Mas naiintindihan ko na kung bakit may magkakahiwalay na layers. Kapag may kakaibang number sa dashboard, may malinaw kaming babalikan: source ba, cleaning rule ba, o join sa Gold?

### 2. May Bronze baseline na kaming pagbabasehan

Sa setup at profiling na binabalikan namin ngayon, na-load na ang seven source files. Ito ang recorded Bronze counts:

| Bronze table | Recorded rows |
|---|---:|
| `assessment_raw` | 206 |
| `courses_raw` | 22 |
| `student_assessment_raw` | 173,912 |
| `student_info_raw` | 32,593 |
| `student_registration_raw` | 32,593 |
| `student_vle_raw` | **10,655,280** |
| `vle_raw` | 6,364 |

May `ingestion_timestamp` at `ingestion_date` ang Bronze tables para may record kung kailan na-load ang data.

Ang initial ingestion approach namin ay `CREATE TABLE IF NOT EXISTS ... USING DELTA AS SELECT ... FROM read_files(...)`. Natutunan ko rin na ang pag-create ng table kapag wala pa ito ay **hindi automatic incremental loading**. Kailangan pa rin ng malinaw na strategy kapag may bagong batch.

At yes, **10.6 million rows** ang student VLE. Seven files lang pakinggan, pero hindi ibig sabihin maliit lang ang laman. 😂

### 3. Ginawang understandable ang repository

Pinalitan namin ang simpleng placeholders ng guides para alam ng future assigned member kung ano ang gagawin sa file.

May purpose, input, expected output, transformation steps, completion checks, at suggested branch name.

| Folder | Ano ang ilalagay |
|---|---|
| `src/sql/00_setup/` | Initialization at source inspection. |
| `src/sql/01_raw/` | Bronze ingestion scripts. |
| `src/sql/02_clean/` | Silver transformation code. |
| `dbt/models/` | Gold dimensions, facts, sources, at reporting view. |
| `src/sql/04_analytics/` | Business analysis queries. |
| `tests/` | Source, Silver, Gold, at business validation SQL. |
| `dbt/tests/` | Automated dbt checks. |
| `docs/` | Source findings, assumptions, pipeline plan, at schema documentation. |
| `.github/workflows/` | Guides para sa CI/CD implementation. |

Napag-usapan din namin ang `resources/` para sa deployment/job definitions kapag kailangan na. Sa current committed structure, nasa `dbt/models/` ang Gold work.

Ang realization ko: kapag may file na, hindi ibig sabihin alam na agad ng teammate ang dapat ilagay. Malaking tulong ang instructions na may example ng output at malinaw na “done when.”

---

## 🔎 Ang mga “ahhh, kaya pala!” moments ko sa OULAD

### Missing value does not automatically mean zero

May **173 missing scores**, at lahat ay sa TMA records. Hindi namin sila basta gagawing zero dahil magkaiba ang “walang recorded score” at “nakakuha ng zero.”

Ang agreed treatment: preserve the records, gawing SQL NULL ang missing placeholders, at huwag isama ang NULL scores sa average. Kasama pa rin ang records kapag submission o participation ang binibilang.

May **11 missing assessment dates**, lahat sa Exam records. Hindi rin namin huhulaan ang deadlines.

May source values ding `?`, kaya hindi sapat na SQL NULL lang ang hanapin sa profiling. Kailangan tingnan kung paano talaga nirerepresent ng source ang missing information.

### Source assessment, assumptions, at pipeline plan: magkakaugnay pero magkaiba

| Document | Tanong na sinasagot | Example |
|---|---|---|
| Source assessment | Ano ang nakita namin sa data? | May 173 missing TMA scores. |
| Assumptions | Ano ang agreed interpretation o treatment? | Unknown scores stay NULL. |
| Pipeline plan | Paano namin ipapatupad at iche-check iyon? | Normalize placeholders, preserve records, validate scored/missing counts. |

Dati parang magkakahalo sila sa isip ko. Ngayon mas malinaw kung saan ilalagay ang observation, decision, at implementation plan.

### Repeated keys need context bago mag-delete

Ito ang isa sa pinakamalaking discoveries:

| Student VLE profiling | Rows |
|---|---:|
| Bronze source rows | 10,655,280 |
| Unique daily interaction keys | 8,459,320 |
| Excess rows over those unique keys | 2,195,960 |

Ang daily key ay combination ng module, presentation, student, resource, at relative day.

Sa agreed plan namin, pagsasamahin ang records sa parehong key gamit ang **`SUM(sum_click)`**. Kung basta isang row lang ang ititira, puwedeng mawala ang recorded clicks.

Kaya ang expected Silver count ay **8,459,320**, pero kailangan mag-match pa rin ang total clicks sa typed Bronze values. **Expected output pa ito; kailangan pang i-implement at i-validate.**

Dito ko mas naintindihan ang grain: *ano ba talaga ang ibig sabihin ng isang row?*

### Ang “date” ay puwedeng relative day

Sa OULAD, may dates na bilang ng araw relative sa presentation start. Day 0 ang start; valid ang negative days kapag before start.

Hindi namin dapat gawing actual calendar dates kung wala namang supplied start date. At hindi rin puwedeng gawing day 0 ang unknown date, kasi may totoong meaning ang zero.

### Kailangan tugma ang design sa instruction ni sir

Ang agreed Gold scope ay **five dimensions at exactly two facts**:

- Dimensions: `dim_student`, `dim_course`, `dim_module_presentation`, `dim_date`, `dim_demographics`.
- Facts: `fact_assessments` at `fact_vle_interactions`.

May supporting **`vw_student_outcomes` view** para kasama sa enrollment reporting ang students kahit walang recorded assessment o VLE activity.

Isa pang lesson: kailangang i-aggregate separately ang dalawang facts bago pagsamahin sa enrollment-level reporting. Kapag detailed facts ang direktang pinag-join, puwedeng dumami ang rows at lumobo ang scores o clicks.

---

## 🗂️ From “ikaw dito” to clear tasks

Naayos din ang GitHub issues: may assigned owner, files na gagalawin, suggested branch, dependencies, at completion checklist. May priority labels na **P0, P1, at P2**, kasama ang area labels.

Ang target completion namin ay **Thursday, September 10, 2026**.

Examples ng branch names:

- `feature/clean-assessments`
- `feature/clean-students`
- `feature/clean-vle`
- `feature/build-dimensions`
- `docs/update-readme`

Mas malinaw na sa akin na **one task can include several related files**. Puwedeng kasama sa isang branch ang transformation, tests, at related documentation.

Ang goal ko sa pag-aayos nito: kapag binuksan ng teammate ang task niya, alam niya kung saan magsisimula, ano ang hinihintay niyang dependency, at paano niya masasabing tapos na.

<!-- Add Rhea's supplied GitHub Issues or Project board screenshot here after upload. -->

### Honest progress check

**Ready na ang foundation, recorded source profiling, documentation, file guides, at issue assignments.** Ang Silver implementation, dbt Gold models, dashboards, at working CI/CD ay susunod pang trabaho.

May documented naming mismatches din sa ilang Bronze loaders at checks na kailangan naming itugma sa actual Databricks tables bago mag-run mula sa fresh environment.

Reminder sa sarili ko: ang maayos na folder structure at existing files ay simula pa lang. Ang proof ay nasa working code, validated results, at repeatable runs.

---

## 💛 New team ulit… at medyo malungkot din pala ako

Honestly, medyo malungkot ako kasi bagong assigned team na naman ang makakasama ko.

Nami-miss ko ang dati kong ka-group. Nakapagpalagayan na kami ng loob, at kabisado na namin kung paano gumalaw ang bawat isa. May comfort sa ganung setup—mas natural nang magtanong, makipag-usap, at kumilos nang magkakasama.

Kaya may adjustment ulit ngayon. Getting-to-know stage na naman.

Pero nakakatuwa rin ang bagong team ko. Sinabi nila na **pinag-pray daw nila na maging ka-group ako.** 🥹

Nakakataba ng puso marinig iyon. Habang ako, may lungkot pa sa pag-miss sa dating team, sila pala masaya na makakasama nila ako.

Sa ngayon, medyo professional pa ang galaw namin. Ramdam mong nag-aadjust at nagpapalagayan pa ng loob. Naiintindihan ko naman—hindi naman automatic ang pagiging comfortable sa bagong group.

Tapos nung chinat ko sila, **nahihiya lang din pala sila sa akin.**

Ako na introverted at naghahanap din ng way para maging close kami: *hala, pare-pareho lang pala tayo?!* 😭😂

Kaya sabi ko, mag-ask lang sila nang mag-ask sa akin para matuto kami pareho at maging close rin. Hindi ko naman alam lahat. May mga bagay na mas naiintindihan ko kapag pinag-uusapan namin o kapag kailangan kong i-explain.

Sinabi ko rin na introverted akong tao.

**Sila rin daw.**

Ayon. Isang grupo ng introverts na naghihintayan lang pala kung sino ang unang magiging comfortable. Hahahahaha.

<!-- Add Rhea's supplied personal/team photo here if she chooses to include one. -->

Ang gusto kong mabuo sa team namin ay yung comfortable kaming magsabi ng “hindi ko gets,” “patingin naman,” o “may idea ako.” Sana habang ginagawa namin ang project, mas maging madali rin ang conversations namin.

Puwede ko palang ma-miss ang dati kong team habang binibigyan ng chance ang bagong team na maging close sa akin. Hindi naman kailangang mawala agad ang lungkot para maging excited sa bagong simula.

---

## 🌷 What I am taking with me

Today, mas naintindihan ko kung gaano karaming decisions ang kasama sa project setup: table grain, missing values, folder responsibilities, validation, at task dependencies.

At sa personal side, may simpleng reminder ako: **minsan, yung taong akala mong sobrang formal, nahihiya lang din pala tulad mo.**

Ang next step namin ay i-align ang Bronze names, simulan ang assigned transformations, at patunayan sa checks na tama ang outputs. Ang next step ko bilang teammate: patuloy na mag-open ng conversation at gawing welcome ang questions.

**Current status: OULAD foundation prepared. Team closeness loading…** 🌱💛

---

<details>
<summary>📚 Project notes na puwede kong balikan</summary>

- [Pipeline plan](https://github.com/ItsYangCoder/oulad-data-engineering-pipeline/blob/main/docs/pipeline_plan.md)
- [Source profiling results](https://github.com/ItsYangCoder/oulad-data-engineering-pipeline/blob/main/docs/source_assessment.md)
- [Documented assumptions](https://github.com/ItsYangCoder/oulad-data-engineering-pipeline/blob/main/docs/assumptions.md)
- [Assigned project issues](https://github.com/ItsYangCoder/oulad-data-engineering-pipeline/issues)

*Technical counts refer to the recorded current source batch. Planned outputs still need implementation and validation.*

</details>

