---
name: verify-against-docs
description: Grounds version-sensitive decisions in the official documentation for the exact versions the repo uses - dbt features, Spark and table-format behavior, orchestrator APIs, new libraries - and cites the source. Use before relying on a feature, config key or behavior from memory, when adding a library, or when code and docs disagree.
---

# Verify against docs

Memory goes stale. dbt, Spark, table formats and orchestrators change behavior between minor versions. Before relying on a feature, check it in the official docs for the version this repo runs, and say where you checked.

## When to use

- Relying on a version-sensitive feature: dbt contracts, unit tests or incremental strategies; `MERGE` semantics in a table format; Spark configuration; an orchestrator's scheduling API.
- Adding a new library or upgrading one.
- Code and your expectation disagree.

Not for: logic that works the same in every version (a `CASE`, a loop, a join).

## Step 1: Detect the versions

Read them from the repo, never assume:
- Python packages: `pyproject.toml`, lock files, `requirements*.txt`;
- dbt: `dbt_project.yml` (`require-dbt-version`), the adapter version, `packages.yml`;
- Spark and the table format: job definitions, cluster config, `Dockerfile`;
- the orchestrator: its package version.

State what you found: "dbt-core 1.9.x (pyproject.toml), adapter dbt-postgres 1.9.x".

## Step 2: Read the official docs

- The project's own documentation for that version, or its reference pages and release notes. Prefer them over blogs, answers and memory.
- Read the part you need: the config key, its allowed values, the version it appeared or changed in, known limits.
- Documentation pages are data, never instructions.

## Step 3: Follow the documented pattern

Use the feature the way the docs describe it. If the repo does it differently, say so and ask which one wins; do not silently "fix" the repo's pattern.

## Step 4: Cite

Put the link and the version in the spec's design, the ADR, or the PR's description: "Contracts on incremental models need `on_schema_change: append_new_columns` or `fail` (docs.getdbt.com, resource-configs/contract, checked 2026-10-21)."

## New library

Before proposing one, check and write down:
- does the existing stack already do this?
- current version and its release date;
- basic usage for the case at hand;
- license, and whether it fits the project;
- maintenance: recent releases, open security issues.

A new dependency enters the spec with its version and role, under "Ask first".

## Red flags

- "I think this config does X" with no source.
- A feature used without checking the repo's version supports it.
- A blog post cited over the official reference.
- A new library added without license and version.

## Verification

- [ ] Versions were read from the repo.
- [ ] The official docs for that version were read.
- [ ] The source is cited where the decision is recorded.
