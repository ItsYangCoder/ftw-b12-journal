# Journal - 2026-10-31 - The Control Plane Manages; the Compute Plane Executes

## Today in one sentence
- I learned to separate Databricks management responsibilities from workload execution—and to avoid treating “stored in the control plane” as a complete description of where every workspace file physically lives.

## What I learned
- The simplest useful model is: **the control plane coordinates, while the compute plane calculates**. The control plane contains Databricks-managed backend services and the web application. It handles the workspace experience, APIs, identities, job definitions, scheduling, and orchestration. The compute plane provides the CPU and memory that execute Python, SQL, Spark transformations, and pipeline tasks.
- A scheduled notebook job crosses that boundary. I create the notebook and job configuration through workspace services. When the schedule fires, the control plane coordinates the run and sends work to an eligible compute resource. The compute plane then executes the notebook code, reads source data, performs transformations, and writes results. The control plane can keep job-run metadata after the temporary compute has stopped.
- The practice question I reviewed used the answer: “the notebook is stored securely and encrypted in the control plane, and the code executes in the compute plane.” That is a helpful exam mnemonic, but current Databricks documentation gives a more precise storage model. Workspace assets—including notebooks, SQL queries, dashboards, Git folders, libraries, Python files, and YAML files—belong to the **workspace file system**. Their physical storage depends on the workspace architecture.
- In a classic AWS workspace, workspace file-system and system data depend on both Databricks control-plane databases and a workspace storage bucket in the customer’s AWS account. In a serverless workspace, Databricks uses managed default storage. Therefore, “the control plane manages the notebook” is safer than assuming every notebook byte is literally stored only inside the control plane.
- The compute plane also has two important forms. With **classic compute**, clusters run in the customer’s cloud account and virtual network. With **serverless compute**, resources run in a Databricks-managed serverless compute plane in the same region as the workspace. Both execute workloads; the difference is who provisions and operates the underlying infrastructure.
- Serverless does not mean “no compute.” It means I do not provision or manage the compute resources myself. Databricks allocates, starts, scales, and updates them. This is why serverless can give faster startup and lower operational overhead, but it still has a compute plane where my code runs.
- Unity Catalog and Delta Lake answer different questions. Unity Catalog governs data and AI objects, permissions, lineage, storage credentials, and external locations. Delta Lake is a table and transaction layer. Neither is the place that “executes” notebook code. If an answer says code runs in Unity Catalog or Delta Lake, it is mixing governance or storage technology with execution infrastructure.
- A cluster driver is part of the compute plane, so an answer mentioning driver execution is not automatically false. The incorrect part of today’s distractor was the claim that the notebook is stored **unencrypted** in a workspace bucket. Good exam elimination requires judging the whole answer, not accepting it because one phrase is technically plausible.
- This architecture also helps me reason about failures. A notebook can remain safely stored even after a short-lived job cluster terminates. A failed Spark task is a compute-plane event, while schedule configuration, run history, and orchestration status are control-plane concerns. Logs and results can persist beyond the life of a single compute resource through workspace system storage.
- The inspected repository image shows a GitHub Actions deployment that validates a Databricks bundle and starts a Chinook job. The GitHub runner is outside this two-plane model, but once it calls Databricks, the platform’s control services accept and coordinate the request, and the configured Databricks compute executes the job. That distinction makes CI/CD troubleshooting more precise: deployment authorization can fail before compute starts, while transformation logic can fail after execution begins.

## Terms I am still learning
- **Control plane** - the Databricks-managed backend and web services that manage the workspace, APIs, identities, job configuration, and orchestration.
- **Compute plane** - the environment where data is processed and notebook, SQL, Spark, job, and pipeline code actually executes.
- **Classic compute plane** - compute resources deployed in the customer’s cloud account and network.
- **Serverless compute plane** - Databricks-managed compute resources allocated on demand without the user provisioning clusters in their cloud account.
- **Workspace file-system data** - user-created workspace assets such as notebooks, SQL queries, dashboards, Git folders, libraries, and small code or configuration files.
- **Workspace system data** - internal persistent information such as job-run results, notebook revisions, query plans, cached query results, and cluster logs.
- **Orchestration** - coordinating when a workload starts, which task runs next, what dependencies apply, and how run status is recorded.

## What confused me
- I initially treated “control plane” as the folder that physically contains notebooks and SQL queries. That is too literal. The control plane is mainly a responsibility boundary. Workspace assets have a separate storage model, and that model differs between classic and serverless workspaces.
- The quiz wording made the architecture look like only two boxes, but the current documentation also describes workspace storage and customer data storage. A better picture is management services, execution resources, workspace storage, and governed data storage working together.
- I still want to understand exactly which logs remain in workspace system storage versus which logs I should export to an external observability platform for long-term retention. The high-level architecture explains the boundaries but not every retention policy.

## One small next step
- [ ] Draw one PondoLens job run as four labeled steps—control request, compute execution, Unity Catalog authorization, and Delta table read/write—then identify where I would look first for an authentication failure versus a transformation failure.

## Git checkpoint
- [x] I created or updated a file
- [x] I wrote a commit
- [x] I pushed my changes

## Decisions or assumptions (optional)
- I will use “managed by the control plane” instead of “physically stored only in the control plane” when explaining notebooks, because the current workspace-storage architecture is more precise.
- I will remember the exam shortcut `control = coordinate` and `compute = calculate`, then check whether a question is asking about responsibility, network ownership, or physical storage.
- I did not claim that I deployed or changed a Data Engineering repository today. The latest visible PondoLens repository activity remains October 22, so this entry is a verified learning reflection based on today’s Databricks study.

## Evidence from today (optional)
- [Databricks high-level architecture](https://docs.databricks.com/aws/en/getting-started/high-level-architecture/) - current documentation for the control plane, classic and serverless compute planes, workspace storage, and workspace assets.
- [Databricks network reference architecture](https://docs.databricks.com/aws/en/security/network/deployment-architecture) - current separation between the control plane, classic compute plane, and serverless compute plane.
- [Databricks serverless compute](https://docs.databricks.com/aws/en/compute/serverless) - explains that Databricks allocates and manages on-demand compute for notebooks, workflows, and Lakeflow pipelines.
- [Latest visible PondoLens commit](https://github.com/ItsYangCoder/ph-government-budget-analysis/commit/d684dff932dea289a8435fa2d1666990797e500f) - October 22 evidence; no newer project commit was used as today’s activity.

![Successful CI deployment steps that validate a Databricks bundle and run a Chinook job](../assets/ci-cd-databricks-dev-success.png)

This privacy-safe repository screenshot shows the boundary in practice: GitHub Actions validates and submits the deployment, while the Databricks platform coordinates and executes the configured job. It contains no credentials, private URLs, personal information, or account identifiers.

## Reflection (optional)
- What felt easy today? The phrase “control coordinates, compute calculates” gave me a clear first test for eliminating obviously reversed answers.
- What felt difficult today? The simplified exam answer and the current product documentation use different levels of detail. I had to separate a useful mnemonic from a precise statement about workspace storage.
- What do I want to understand better next time? I want to trace one real scheduled PondoLens job from the API request through authentication, compute startup, table access, task execution, logging, and final run status.

## Mood or meme (optional)
- The control plane is the dispatcher; the compute plane is the crew doing the heavy work. The dispatcher can schedule the job, but it does not lift the data. 🚦
