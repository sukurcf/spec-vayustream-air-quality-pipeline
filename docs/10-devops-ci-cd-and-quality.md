# DevOps, CI/CD, and quality

## Git workflow

Use a small GitHub Flow: branch from `main`, make focused commits, open a pull request, fix review/CI findings, then merge when required checks pass. Keep database state, local credentials, generated dbt artifacts, and test mutations out of Git. No implementation workflow or Compose file belongs in this specification repository.

## One CI job

CI uses **one Python 3.12.x environment only**; do not add a Python version matrix. Hosted CI may download packages and start an isolated PostgreSQL 18.6 service. That CI service is not a project runtime dependency.

| Step in the job | Check | Pass condition |
|---|---|---|
| Prepare Python 3.12.x and the locked environment | Install declared tools and dbt adapter | Lock resolves successfully |
| Lint Python support code, if present | Ruff | No lint or format findings |
| Run Python helper tests, if present | pytest with line/branch coverage | At least 80% line and 70% branch coverage for the named package |
| Start isolated PostgreSQL 18.6 | Use the local target and fixture | Only one database is used |
| Validate dbt project | Parse, seed, full-refresh build, and test | All nine required dbt test nodes pass |
| Check exact demo result | Inspect mart row count and total | 20 order rows and `441.00` net sales |

CI MUST NOT depend on a live API, hosted database, chart, dashboard, or generated documentation website.

## Local environment values

Use local placeholders in examples; keep real local values in an ignored environment file.

| Name | Example | Secret |
|---|---|---|
| `POSTGRES_HOST` | `127.0.0.1` | No |
| `POSTGRES_PORT` | `15433` | No |
| `POSTGRES_DB` | `dbt_starter` | No |
| `POSTGRES_USER` | `dbt_user` | No |
| `POSTGRES_PASSWORD` | `local-placeholder` | Yes |
| `DBT_TARGET` | `local` | No |

## Definition of Done

- The five Must requirements and all linked test cases are satisfied.
- The clean fixture builds `mart_sales` with 20 rows and `441.00` net sales.
- All nine required dbt tests pass on clean data and the applicable injected-defect tests fail.
- The local demo works after downloads with external access blocked.
- PostgreSQL binds only to `127.0.0.1:15433`; no secret or generated local state is committed.
- Any authored Python support package meets the documented 80% line/70% branch coverage target.
- Documentation matches the implementation and the five-minute demo is reproducible.

[Back to README](../README.md)
