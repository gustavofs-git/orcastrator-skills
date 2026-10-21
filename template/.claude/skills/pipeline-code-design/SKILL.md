---
name: pipeline-code-design
description: Helps decide how to organize pipeline code without imposing a paradigm - one script per endpoint or a generic loader with config, I/O at the edges and pure transformations, one logger, base classes or functions, when to extract a shared utility. Use when setting up a repo's code structure, adding a second or third similar job, or when a review finds duplicated pipeline logic.
---

# Pipeline code design

Pipeline code tends to two failures: ten copy-pasted scripts that drift apart, or a framework built for one job. This skill lays out the options and their trade-offs; the user decides, and the decision becomes a rule in `docs/conventions.md` (and an ADR when it is expensive to reverse).

## When to use

- `/orca-init` in a greenfield repo: how will pipeline code be organized?
- Adding a job that looks like an existing one.
- A review finds the same logic in two places.

## Present options, don't impose

For each question, give two or three options with trade-offs and a recommendation tied to this repo. Examples:

| Question | Options |
|---|---|
| Many similar sources or endpoints | one script per endpoint · a generic loader plus one config block per endpoint · a mix: generic loader, custom hooks for the odd ones |
| Where I/O happens | I/O at the edges, transformations as pure functions over DataFrames or SQL · I/O mixed in (simpler for tiny jobs) |
| Logging | one shared logger with structured fields (run id, source, window, row counts) · per-job logging |
| Shape of shared code | functions in a `common/` module · base classes jobs inherit from |
| Configuration | config files per endpoint or job · code constants |

Recommend from the evidence: how many similar cases the spec or roadmap shows, what the team already uses, what the tests need.

## Rules

- **Do not generalize before the third similar case**, unless the spec already shows N identical cases (for example, eight endpoints of the same API with the same pagination). Then build the generic piece first.
- **I/O at the edges.** Pure transformation functions are testable with small synthetic inputs and no engine setup. Reading and writing live in thin functions around them.
- **Shared utilities come first.** Logger, API client, generic loader: each is a task **before** the tasks that use it, usually in a greenfield's foundation item. Never build a utility inside the first feature that needs it and copy it into the second.
- **One canonical helper.** Before writing a helper, search for an existing one. A near-duplicate is a review finding.
- **Config is data.** Validate config files at load time with explicit types; a typo in a config key should fail the run, not silently skip an endpoint.
- **Name by the domain**, not the mechanism: `orders_loader`, not `generic_runner_2`.

## How the decision travels

1. The decision is taken in `/orca-init` (greenfield) or in a spec (when a new pattern appears).
2. Expensive to reverse → ADR (`writing-adrs`).
3. The rule goes into `docs/conventions.md` with one example from the repo.
4. `/orca-plan` points tasks to the pattern ("pattern to follow" in the context package).
5. The `reviewer` flags code that duplicates an existing utility or skips the pattern.

## Red flags

- A base class with one subclass.
- A generic loader built for one endpoint "for later".
- The same pagination or retry logic in two jobs.
- Transformations that need a live connection to test.
- A utility created inside a feature task and then copied.

## Verification

- [ ] Options were presented with trade-offs; the user chose.
- [ ] The rule is in `docs/conventions.md` with an example.
- [ ] Shared utilities are planned as tasks before their users.
