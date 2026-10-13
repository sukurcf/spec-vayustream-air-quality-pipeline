# Users and roles

## Roles

| Role | Responsibility |
|---|---|
| Junior data engineer | Implements the dbt project, runs the local workflow, and explains the model grains and tests |
| Trainer/reviewer | Checks the documented inputs, results, test evidence, and five-minute demo |

## Personas

| Persona | Goal | Main concern |
|---|---|---|
| Mira, junior data engineer | Build a repeatable sales mart using SQL and dbt | Keep the model small enough to understand and finish in 30 hours |
| Dev, trainer | Evaluate the same deterministic contract for each learner | Verify exact results and meaningful failure tests without a custom UI |

## Ownership and permissions

| Action | Junior data engineer | Trainer/reviewer |
|---|---|---|
| Edit the separate implementation repository | Yes | Review |
| Start/stop the local PostgreSQL service | Yes | May observe |
| Run dbt seed/build/test | Yes | May request evidence |
| Inspect local mart rows and test results | Yes | Review |
| Change the grading contract | No | Clarify through a documented spec revision |

## User stories

| ID | Story | Requirement links |
|---|---|---|
| US-01 | As a junior data engineer, I want one source and clear staging models so I can trace input columns into a mart. | FR-DBT-01, FR-DBT-02 |
| US-02 | As a junior data engineer, I want dbt tests to reject broken keys and business data so bad sales rows are not accepted silently. | FR-DBT-03, FR-DBT-04 |
| US-03 | As a trainer, I want a clean local build with exact row and sales totals so I can review the project consistently. | FR-DBT-05 |

[Back to README](../README.md)
