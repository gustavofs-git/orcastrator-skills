## Delivery
<!-- Appended to 4-plan.md by /orca-ship. Keep only the rows that apply; say in one line why the others were skipped.
     Tick each line with its evidence when it completes. -->
Skipped: <rows and why, e.g. "backfill: new table, no history to load">

### Rollback
<!-- Any write to an existing table. Recorded BEFORE the write. -->
- [ ] Table version before delivery: <table> → <snapshot id | timestamp | backup name>
- [ ] Command to return to it: `<exact command>`

### Write-audit-publish
<!-- New table, or changed logic in an existing table. -->
- [ ] Written to the audit location: <table or branch>
- [ ] Audited: <quality checks from the spec> → <results>
- [ ] Published: <how: swap, merge, branch publish> → <evidence>

### Backfill
<!-- New load, new column with history, or a fix with wrong data already written. -->
- [ ] Window: <from> → <to> · expected cost: <value> · environment: <name>
- [ ] Idempotency proven by: R<n> (<test or qa evidence>)
- [ ] Run: <command> → <rows written, duration, cost>

### Consumers
<!-- Any contract changed. From lineage. -->
- [ ] Consumers: <list, with owners>
- [ ] Breaking change: deprecation window <from> → <to> · removal item <id> | not breaking
- [ ] Notice posted: <where, link>

### Post-delivery checks
<!-- New or changed load. -->
- [ ] Load 1 (<date>): freshness <value> · volume <value> · quality <result>
- [ ] Load 2 (<date>): ...
- [ ] Load 3 (<date>): ...
