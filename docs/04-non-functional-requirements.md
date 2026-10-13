# Non-functional requirements

These are acceptance targets for the student implementation, not claims that an application has been run or measured.

| ID | Area | Target and verification |
|---|---|---|
| NFR-LOC-01 | Local operation | Required build and tests use one laptop and one PostgreSQL 18 service; no cloud account, live API, or remote runtime dependency. |
| NFR-PERF-01 | Performance | On the reference laptop, after PostgreSQL reports healthy, the 20-order seed/build/test demo completes within 5 minutes. Record the result; do not assume it. |
| NFR-REL-01 | Repeatability | Two full-refresh builds from the same fixture produce 20 mart rows and total net sales `441.00` each time. |
| NFR-OFF-01 | Offline use | After Python/package and container-image downloads, `dbt seed`, `dbt build`, and `dbt test` succeed with external network access disabled. |
| NFR-SEC-01 | Privacy | Use fictional data only. Do not commit real passwords, personal data, or a populated local environment file. |
| NFR-SEC-02 | Network | PostgreSQL is published only on `127.0.0.1:15433`; it is not exposed on a public or wildcard interface. |
| NFR-MAINT-01 | Data types | Quantities are integers, dates are dates, and money is exact numeric with two decimal places; do not use binary floating-point money. |
| NFR-TEST-01 | Coverage | Any authored Python support package targets at least 80% line and 70% branch coverage. dbt SQL behavior is verified by the named model tests and acceptance cases. |

## Reference hardware and reliability

| Resource | Reference target |
|---|---|
| Host | 4 CPU cores and 8 GB RAM |
| Free disk | At least 5 GB for the checkout, database image, and local data |
| Database | One local PostgreSQL 18 container |
| Host port | `127.0.0.1:15433` |

The resource values are proposed acceptance conditions, not measured results. On a healthy local database, any failed seed/build/test command must return a nonzero result and preserve its failure in the command output. Re-running a successful full-refresh demo must not add duplicate mart rows.

## Security and local data

Store only the small fictional fixtures and generated local dbt/PostgreSQL artifacts. Keep credentials in local environment configuration with safe placeholders in examples. Generated artifacts and local database files are excluded from Git. The specification repository itself remains Markdown-only.

[Back to README](../README.md)
