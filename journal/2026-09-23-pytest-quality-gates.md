# Journal - 2026-09-23 - Pytest and CI-Style Quality Gates

## Today in one sentence
- I learned that a successful test run is strongest when it is part of a small verification chain: tests, repository checks, formatting, linting, a focused commit, and a pushed branch that another person can review.

## What I learned
- I ran the project's test suite with `pytest -q`, and the terminal evidence showed **63 tests passed in 13.52 seconds**. This gave me a quick regression signal: the behaviors covered by those tests still worked after the configuration and documentation changes.
- I also verified the repository with `git diff --check`, `ruff format --check .`, and `ruff check .`. These commands answer different questions. Pytest checks behavior, `git diff --check` catches whitespace problems in the patch, Ruff's format check confirms consistent formatting, and Ruff's lint check catches suspicious or inconsistent Python code.
- The output showed **87 files already formatted** and **all lint checks passed**. I realized that "the tests passed" does not automatically mean "the change is ready." A change can behave correctly while still containing formatting problems, lint errors, or a messy diff.
- The verification work was committed as `6646931` with the message `test: verify pytest gates and document validation`, then pushed to `chore/issue-7-pytest-ci-verification`. This is useful because a reproducible branch gives reviewers something concrete to inspect; a screenshot alone is only supporting evidence.
- I am starting to see pytest as more than a command I run before a presentation. In a data pipeline, tests can guard assumptions about configuration, ingestion behavior, schemas, transformations, and failure handling. CI can then repeat those checks consistently whenever code is proposed for integration.
- An earlier NYC Mobility pull request already documented an offline pytest CI foundation with 15 passing tests. Today's 63-test result appears to come from a broader production working copy, so I should not present the two counts as if they were the same repository state.

## Terms I am still learning
- **Regression test** - a test that checks whether behavior that worked before still works after a new change.
- **Quality gate** - a check that must pass before code is treated as ready for review, merge, or deployment.
- **Linting** - automated analysis that finds code-quality problems without running the complete application.
- **Formatting check** - a non-mutating check that confirms files already follow the project's formatting rules.
- **Continuous Integration (CI)** - an automated process that runs repeatable checks when code changes are pushed or proposed.
- **Deterministic check** - a check that should give the same result when the code and environment are the same.

## What confused me
- I initially treated pytest, Ruff, and CI as if they were interchangeable. They support the same quality workflow, but they test different things: pytest exercises expected behavior, Ruff checks Python style and common code issues, and CI is the environment that runs these checks automatically.
- I still need to confirm how the 63 local tests map to the exact tests executed by the repository's GitHub Actions workflow. A green local run is strong evidence, but local and CI environments can differ in Python version, dependencies, environment variables, and available services.
- The terminal also warned that Git had automatically chosen the commit author identity. That did not stop the commit, but it is a reminder to configure the correct Git name and email before future work so authorship remains accurate.

## One small next step
- [ ] Compare the GitHub Actions pytest command and Python version with the verified local commands, then record any differences in the validation document.

## Git checkpoint
- [x] I created or updated a file
- [x] I wrote a commit
- [x] I pushed my changes

## Decisions or assumptions (optional)
- I will describe the result as **local verification plus a pushed review branch**, not as a merged or deployed production change.
- I will keep behavioral tests, formatting checks, linting, and diff validation as separate gates because each catches a different class of problem.
- I will not publish the full terminal screenshot because it contains a local username, filesystem paths, an email address, and a repository URL that are not necessary to explain the learning.
- I am using an existing safe CI/CD screenshot from this journal repository only as a visual comparison of the automated pipeline goal, not as evidence that today's NYC branch was deployed.

## Evidence from today (optional)
- Local verification shown in the reviewed screenshot: `63 passed in 13.52s`, `87 files already formatted`, and `All checks passed!`
- Verified local commit: `6646931` — `test: verify pytest gates and document validation`
- Verified pushed branch: `chore/issue-7-pytest-ci-verification`
- NYC Mobility's earlier pytest CI foundation: [Pull Request #22](https://github.com/ItsYangCoder/nyc-mobility-ingestion-pipeline/pull/22)
- Journal template followed: [entry-extended.md](https://github.com/ItsYangCoder/ftw-b12-journal/blob/main/templates/entry-extended.md)

## Reflection (optional)
- What felt easy today? Once I read the commands as a sequence, the verification flow made sense: run tests, inspect the patch, check formatting, check linting, then commit and push.
- What felt difficult today? The harder part was deciding what each green result actually proves. A passing formatter does not prove pipeline logic, and a passing local test suite does not prove that Databricks or a live data source will behave the same way.
- What do I want to understand better next time? I want to trace one test from its Python function, through pytest discovery, into GitHub Actions, and then explain exactly what pipeline risk it prevents.

## Mood or meme (optional)
- Sir. Myk: "basic lang yang group project nyo". Me looking at the code:

![Successful GitHub Actions deployment to Databricks, used as a safe example of the automated pipeline goal](../assets/basic_meme.jpg)
image from google
