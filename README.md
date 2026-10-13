# dbt Starter: Local Sales Mart

Build one small, local dbt project that turns fictional retail orders and products into a tested sales mart. This is a specification for a junior data engineer to implement in a separate repository; this repository contains no application code.

## At a glance

| Item | Value |
|---|---|
| Duration | Exactly 5 days × 6 hours = 30 hours |
| Target role | Junior data engineer |
| Required stack | Python 3.12, PostgreSQL 18, dbt Core 1.12.x, dbt-postgres 1.11.x, Docker Compose |
| Input | One dbt source group with 20 fictional orders and 6 products; hard limits are 100 orders and 20 products |
| Models | Two staging models, one sales mart, and one optional intermediate model only if it fits the 30-hour plan |
| Local database | PostgreSQL on `127.0.0.1:15433` |
| Required checks | dbt parse/build/test, exact mart row count and total, 15–20 distinct test cases |

## What you will build

Load the two small fixture tables with dbt seeds, stage them, join products to orders, and materialize one repeatable full-refresh sales mart. The required demo mart has exactly **20 order rows** and **441.00** in total net sales. It includes 18 completed orders and 2 cancelled orders; cancelled orders contribute zero net sales.

The project stays on one laptop. It has no UI, live API, cloud resource, or other remote runtime service. Do not add Spark, Kafka, Delta, or Airflow. Local dbt artifacts are allowed; do not host or style a documentation website.

## Local demo

After dependencies and the PostgreSQL image have been downloaded, run:

```text
docker compose up -d db
uv run dbt seed --target local
uv run dbt build --target local --full-refresh
uv run dbt test --target local
```

The database publishes only `127.0.0.1:15433`. The required demo and tests use fictional local fixtures and work without internet after initial downloads.

## Document index

| Document | Purpose |
|---|---|
| [01 Project overview](docs/01-project-overview.md) | Problem, outcome, scope, and job skills |
| [02 Users and roles](docs/02-users-and-roles.md) | Two roles and three user stories |
| [03 Functional requirements](docs/03-functional-requirements.md) | Five Must requirements and business rules |
| [04 Non-functional requirements](docs/04-non-functional-requirements.md) | Local performance, reliability, security, and hardware targets |
| [05 System architecture](docs/05-system-architecture.md) | One model-flow diagram and design questions |
| [06 Tech stack and setup](docs/06-tech-stack-and-setup.md) | Verified versions, local setup, commands, and port |
| [07 Data model](docs/07-data-model.md) | Source, grains, keys, schema, and exact fixture |
| [08 Pipeline specification](docs/08-pipeline-specification.md) | dbt model contract, tests, and expected results |
| [09 Testing strategy and cases](docs/09-testing-strategy-and-test-cases.md) | 19 unique test cases and coverage target |
| [10 DevOps, CI/CD, and quality](docs/10-devops-ci-cd-and-quality.md) | Git flow, one CI job, and Definition of Done |
| [11 Milestones and deliverables](docs/11-milestones-and-deliverables.md) | Shared five-day plan and five-minute demo |
| [12 Evaluation rubric](docs/12-evaluation-rubric.md) | 100-point rubric and achievable Must gates |
| [13 Interview preparation](docs/13-interview-preparation.md) | Ten dbt/data-engineering questions |
| [14 Glossary and resources](docs/14-glossary-and-resources.md) | Terms and official learning links |

## Working agreements

Use fictional data only. Follow the existing cohort's individual-work, Git, and AI-assistance policy: understand submitted work and disclose significant AI help to the trainer. The required interface is dbt's command line and SQL results, not a custom application.

## Change log

| Version | Date | Change |
|---|---|---|
| 1.0 | 2026-10-02 | First release: six-week VayuStream streaming and air-quality pipeline specification. |
| 1.1 | 2026-10-02 | Removed student-built frontend and cloud-deployment scope while retaining the six-week streaming/AQI project. |
| 1.2 | 2026-10-13 | Replaces the six-week scope with dbt Starter, an exactly 30-hour local PostgreSQL/dbt project. Grading now uses five Must requirements, 19 unique test cases, exact fixture results, and the revised 100-point rubric; former streaming, AQI, and extended test-catalogue gates no longer apply. |
