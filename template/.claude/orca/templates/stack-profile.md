# Stack profile: <stack name>
<!-- Written by /orca-init when no shipped profile fits (e.g. databricks-scala). Saved as .claude/orca/references/stacks/<name>.md.
     Every command and pattern carries a marker: [detected: <file>], [confirmed] or [unclear: <question> · <who decides>].
     See references/stacks/dbt.md and pyspark.md for filled examples. -->

Engine and versions: <e.g. Databricks Runtime 15.4 LTS, Spark 3.5, Scala 2.12> [ ]
Table format: <e.g. Delta> [ ]
Build tool: <e.g. sbt> [ ]

## Commands
| Purpose | Command |
|---|---|
| focused tests | |
| one test | |
| suite | |
| lint and format | |
| build / package | |
| run a job in dev | |
| consumers (lineage) | <command, or how to find readers: search for the table name, catalog lineage, orchestrator graph> |
| docs from contracts | <command, or "not set up"> |

Never a command that matches `production.command_patterns` in `.claude/orca/config.json`.

## Contract
<!-- Where a table's executable contract lives and what fails when the output diverges.
     Options when the repo has none: a schema file next to the job plus a contract test (like the PySpark profile);
     engine-enforced schema and constraints (e.g. table constraints in the catalog); a typed case class plus a schema test.
     The user chooses; an ADR if it is expensive to reverse. -->

## Tests
| Level | Form in this stack |
|---|---|
| pure transformation | |
| component | |
| contract | |
| integration | |
| runtime checks | |

<!-- Show one short test from this repo, or the agreed shape: synthetic input built by hand, explicit schema, assert on the output rows, R id in the name. Local engine setup (session, timezone UTC, few shuffle partitions). -->

## Fixtures
<!-- Where synthetic fixtures live and their format. Never real rows. -->

## Floor guard patterns
<!-- What lowering the bar looks like in this stack, flagged in a diff unless the spec names it:
     dedup added to hide a grain bug, casts or defaults over failing columns, filters that drop failing rows,
     schema auto-merge turned on, tests ignored or removed, thresholds loosened. Use this stack's syntax. -->
