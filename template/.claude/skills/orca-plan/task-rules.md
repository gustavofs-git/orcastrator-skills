# Task rules

## Format

Every task in `4-plan.md` has this shape:

```markdown
## T2 Orders ingested twice in the same window do not duplicate or leave gaps (P)
State: open
Covers: R1, R3
Depends on: —

### Requirements (copied verbatim from 3-spec.md)
- [ ] R1 When the same time window is ingested twice, the orders loader shall produce the same rows in `bronze.shop_orders`.
- [ ] R3 If a run fails mid-window, then the orders loader shall resume on the next run from the last stored order, without gaps or duplicates.

### Context to load (only this)
- specs/<n>-<slug>/3-spec.md, sections "Design" and "Boundaries"
- contract of bronze.shop_orders: src/<pkg>/schemas/bronze_shop_orders.yml
- src/<pkg>/io/api_client.py (pattern to follow)
- tests/fixtures/shop/

### Touches
- src/<pkg>/layers/bronze_writer.py
- src/<pkg>/schemas/bronze_shop_orders.yml
- tests/ingestion/

### Verify
<focused test command from docs/architecture.md>
```

## Fields

- **Title:** the outcome in domain language. "Orders ingested twice do not duplicate." Never a function, class or signature; those belong to the build.
- **State:** `open`, then `PR #<n>` when the build opens the PR, then `merged`.
- **Covers:** requirement ids only, at most 3.
- **Depends on:** task ids with the reason ("T1: needs the watermark reader"), or `—`.
- **Requirements:** each covered requirement, copied verbatim from the spec. Never paraphrase.
- **Context to load:** the only files the implementer reads besides `CLAUDE.md` and the stack profile. Sections of the spec, the contract, the pattern to follow, fixtures. Put the most important item last.
- **Touches:** every file and resource the task will create or change: code, schema files, tests, config, tables. The parallel check reads this list.
- **Verify:** the focused test command, exactly as `docs/architecture.md` and the stack profile write it. Never an assumed default command.

File paths appear only in Context to load and Touches.

## Parallel tasks: `(P)`

A task gets `(P)` only when all four conditions hold:
1. It does not depend on the result of another pending task.
2. It does not touch the same files or resources as another pending task: the same model, schema file, table or config.
3. It does not wait for another task's approval.
4. What it needs from the environment already exists, or is created inside the task.

The interfaces the spec defines between components are what let tasks run in parallel without merge conflicts. If the spec does not define the interface, the tasks are not parallel.

## Size

| Signal | Action |
|---|---|
| more than 3 requirements | split |
| more than about 300 changed lines expected | split |
| two independent subsystems in one task | split |
| "and" in the title joining two outcomes | probably two tasks |
| one requirement, one file | do not split further |

Steps inside a task (TDD cycles, commits) never become tasks.
