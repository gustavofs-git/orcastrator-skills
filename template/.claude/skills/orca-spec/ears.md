# EARS requirements

EARS (Easy Approach to Requirements Syntax) gives five sentence patterns. Every requirement uses one of them.

| Pattern | Form | Use for |
|---|---|---|
| Ubiquitous | The `<subject>` shall `<response>`. | a rule that always holds |
| Event-driven | **When** `<trigger>`, the `<subject>` shall `<response>`. | a reaction to an event |
| State-driven | **While** `<state>`, the `<subject>` shall `<response>`. | a rule while a state lasts |
| Unwanted behavior | **If** `<condition>`, **then** the `<subject>` shall `<response>`. | the bad case: failure, invalid data, delay |
| Configured feature | **Where** `<feature is configured>`, the `<subject>` shall `<response>`. | a rule that applies only when something is configured |

Patterns can combine: *While* `<state>`, *when* `<trigger>`, the `<subject>` shall `<response>`.

## Examples in data

- R1 When the same time window is ingested twice, the orders loader shall produce the same rows in `bronze.shop_orders`.
- R2 If the API returns HTTP 429, then the shop API client shall wait for the `Retry-After` interval and retry up to 5 times.
- R3 While a backfill is running, the incremental orders job shall not start.
- R4 The `bronze.shop_customers` table shall not store customer emails in plain text.
- R5 Where a store filter is configured, the orders loader shall ingest only orders from those stores.

The **If** pattern forces you to write what happens to late data, a NULL key, a duplicate, a failure halfway, a changed schema. Most data bugs live there.

## Rules

- **Concrete subject.** The loader, the orders job, the `silver.orders` table, CI. Never "the system". For a process step, the responsible role ("the item owner").
- **One behavior per requirement.** An "and" joining two responses means two requirements.
- **Only `shall`.** A recommendation is not a requirement; it goes in the design.
- **Nothing vague without a number.** "Near real time", "fast", "complete", "clean", "deduplicated" only with their definition: freshness in hours, the uniqueness key, the tolerance.
- **Behavior, not implementation.** "Use MERGE" or "add a DISTINCT" is design.
- **Group by subject if it helps** (Load correctness, Privacy, Source behavior). A group is never named after an EARS pattern.
- **Keywords stay in English** (`When`, `While`, `If ... then`, `Where`, `shall`) even if the project writes artifacts in another language. Only the variable parts are translated.
- **Canonical ids.** `R1..Rn` live in the spec. Everywhere else (plan, tests, PRs) only the id appears. Where the text is needed (the task's context package), it is copied verbatim, never paraphrased.
- **Ids never change.** A removed requirement keeps its id with `(removed: <reason>)`; a new one takes the next free number.

| Wrong | Right |
|---|---|
| After each run, the table shall... | **When** a run finishes, the `silver.orders` table shall... |
| ...shall attribute it to `UNKNOWN` **and** count it in a quality check. | ...shall attribute it to `UNKNOWN`. (The count is a check; it goes in the design.) |
| The system shall keep data fresh. | The orders job shall publish data no older than 36 hours. |
| The loader shall use an upsert by `order_id`. | When an order arrives twice, the loader shall keep one row per `order_id`. |

## Checks every requirement must pass

- It is testable: a test or a check can fail it.
- It describes behavior, not implementation.
- It has a concrete subject and a single `shall`.
