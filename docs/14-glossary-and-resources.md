# Glossary and resources

## Glossary

| Term | Simple definition |
|---|---|
| dbt project | A set of configuration, models, and tests that dbt executes |
| Model | A named SQL transformation that creates a relation |
| Source | A declared input relation outside the model DAG |
| Seed | A small CSV file loaded into the database by dbt |
| Staging model | A close-to-source model that casts and standardizes fields |
| Intermediate model | A reusable transformation between staging and a final mart |
| Mart | A shaped dataset intended for analysis |
| Grain | What one row represents |
| `source()` | dbt reference to a declared source table |
| `ref()` | dbt reference to another model |
| Materialization | How dbt creates a model in the database |
| Full refresh | Rebuild the selected model's result instead of appending new rows |
| Incremental model | A model that processes a subset of data on later runs |
| Primary key | Column or columns that identify one row |
| Unique test | Test that a key has no duplicate values |
| Not-null test | Test that a required value is present |
| Relationship test | Test that a key resolves to a key in another relation |
| Accepted-values test | Test that a column contains only listed values |
| Singular test | A custom dbt data test for a specific business condition |
| Adapter | dbt component that connects to a database engine |
| Profile | Local connection configuration selected for a dbt target |
| Fixture | Small deterministic data used in tests and demos |
| Idempotent run | Repeating an operation leaves the same intended result |
| PostgreSQL | Open-source relational database used by this local project |
| Docker Compose | Tool for defining and operating the one local database service |

## Official documentation

- dbt documentation: <https://docs.getdbt.com/docs/introduction>
- dbt sources: <https://docs.getdbt.com/docs/build/sources>
- dbt seeds: <https://docs.getdbt.com/docs/build/seeds>
- dbt data tests: <https://docs.getdbt.com/docs/build/data-tests>
- dbt commands: <https://docs.getdbt.com/reference/dbt-commands>
- PostgreSQL 18 documentation: <https://www.postgresql.org/docs/18/>
- Docker Compose documentation: <https://docs.docker.com/compose/>
- Python documentation: <https://docs.python.org/3.12/>
- uv documentation: <https://docs.astral.sh/uv/>
- pytest documentation: <https://docs.pytest.org/>
- Ruff documentation: <https://docs.astral.sh/ruff/>
- GitHub Actions documentation: <https://docs.github.com/actions>

## Five recommended readings

1. dbt sources: <https://docs.getdbt.com/docs/build/sources>
2. dbt seeds: <https://docs.getdbt.com/docs/build/seeds>
3. dbt data tests: <https://docs.getdbt.com/docs/build/data-tests>
4. dbt build command: <https://docs.getdbt.com/reference/commands/build>
5. dbt incremental models, for the optional ungraded exercise: <https://docs.getdbt.com/docs/build/incremental-models>

[Back to README](../README.md)
