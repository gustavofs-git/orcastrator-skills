# Discovery rules

## Depth

Start light. Escalate to full when any trigger below appears, and write which one.

| Light (the item extends what exists) | Full (any trigger) |
|---|---|
| pattern to follow | new or poorly documented external source |
| consumers and lineage of touched tables | new layer or new pattern |
| compatibility with current contracts | personal data |
| integration risk | incremental load or backfill |
| | table critical in cost or performance |
| | new library |

Full depth adds only what its triggers need. A new library does not require profiling a source.

## Gap map

One row per expected outcome in the brief.

| Outcome (from the brief) | What exists | Gap |
|---|---|---|
| sales per region and day | `fct_orders`, `dim_store` | Missing: the mart |
| correct region in history | `dim_store` keeps history | Constraint: the join must filter on validity dates |
| closed month never changes | — | Unknown: cancellation after close → product owner |

Tags:
- **Missing:** does not exist and must be built.
- **Unknown:** nobody knows yet. Becomes an open question with an owner and `blocks:`.
- **Constraint:** exists and limits the work: an ADR, a consumer's contract, a repo pattern, a CODEOWNERS rule.

An outcome fully served by what exists gets `—` in the Gap column. Say so; it may shrink the spec.

## Options

Only when there is more than one reasonable path. Each option gets two or three lines:

```
A. Extend mart_sales_by_channel with a region column
   + one model to maintain   − changes the grain of a table with consumers
B. New mart_sales_by_region following the same pattern   (recommended)
   + no consumer impact      − similar logic in two models
```

Recommend one and say why in one line. Do not decide; the spec does.

## Risk

| Risk | Typical signals |
|---|---|
| **High** | incremental load or backfill on a table with consumers; personal data; breaking contract change; wrong data already published; new external source |
| **Medium** | new model or job following an existing pattern; additive change with consumers |
| **Low** | additive change with no consumers yet, or internal only |

Write one line of justification: `Risk: High (incremental load + personal data)`. When signals disagree, take the highest and say why.

## Conflicts

Each line: what contradicts what, how it was resolved, and the date.

```
- ADR-0003 asks for a generic loader; the code has none → built in this item (confirmed 2026-10-22)
```

An unresolved conflict stays in the list with `unresolved`. The spec does not start while one remains.
