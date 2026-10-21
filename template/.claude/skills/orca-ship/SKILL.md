---
name: orca-ship
description: Delivers one work item to the people who consume its data, after every task is merged. Builds a delivery checklist from what the spec changed (write-audit-publish, backfill, rollback, consumers, post-delivery checks), runs it with you, records CHANGELOG and build log, and closes the item.
argument-hint: <item id>
disable-model-invocation: true
effort: low
---

# orca-ship

Merging is not delivering. Run this once per work item, after all its tasks are merged, to get the data to its consumers safely.

Work item: $ARGUMENTS

## Step 0: Check the gates

From `specs/<id>-*/`:
- Every task in `4-plan.md` has `State: merged`. Confirm each with the "PR state" command from Platform commands in `docs/conventions.md`.
- Every requirement has evidence: a test in a merged PR, or real-data evidence in an `orca-qa verdict` comment. Requirements marked `already covered` or `deferred` count as handled.
- An open question with `blocks: ship` has an answer.

If any fails, stop and list what is missing.

**Resuming.** If `4-plan.md` already has a `## Delivery` section, continue from its first unchecked line.

## Step 1: Build the checklist from the spec

Read `3-spec.md` (contract changes, load semantics, quality checks) and the lineage of the changed tables. Copy only the rows that apply from `.claude/orca/templates/delivery-checklist.md`:

| Row | Applies when |
|---|---|
| Write-audit-publish | a new table, or changed logic in an existing table |
| Backfill | a new load, a new column with history, or a bug fix with wrong data already written |
| Rollback | any write to an existing table |
| Consumers | any contract changed |
| Post-delivery checks | a new or changed load |

Skip the rest and say why in one line. An additive column with no history and no backfill may need only "notify the consumers of `silver.orders`". A new ingestion gets every row.

Append the checklist to `4-plan.md` as a `## Delivery` section, so the delivery is recorded next to the tasks.

## Step 2: Run the checklist with the user

One row at a time. Each row says what will run, where, and its expected cost, and waits for a yes.
- **Environments** are the ones `docs/architecture.md` names. Run in dev first when the row allows it.
- **Production:** any command that matches `production.command_patterns` in `.claude/orca/config.json` needs an explicit yes for that exact command. Prefer the repo's deploy pipeline over running it by hand.
- **Rollback first:** record the table version (snapshot id, time-travel timestamp or backup name) and the exact command to return to it, before any write to an existing table.
- **Backfill:** state the window, the expected cost, and the requirement that proves idempotency. Follow `incremental-loads-and-backfills`.
- **Consumers:** list who reads the changed tables. A breaking change needs a deprecation window and a notice with the date (follow `schema-evolution-and-deprecation`). Posting the notice is an external action: show the text, then post after a yes.
- **Post-delivery checks:** freshness, volume and quality on the first loads. If the first loads have not run yet, leave those lines unchecked with their dates. The item stays open; run `/orca-ship <id>` again after them.

Tick each line in `## Delivery` with its evidence (a count, a snapshot id, a link) as it completes.

## Step 3: Record

Delegate to the `tech-writer` agent with: the item id and title, the PR numbers, the "view PR" command, the spec path, and the telemetry file `.orca/telemetry/sessions.jsonl` if it exists. It writes the `CHANGELOG.md` entry and the `docs/build-log.md` entry, with the item's cost or "unknown".

## Step 4: Close the item and end the turn

When every line in `## Delivery` is checked:
1. Close the item in the tracker with a short summary comment (the "close work item" command). This is an external action: show the comment, then close after a yes.
2. Reply with what was delivered, the CHANGELOG line, the build-log line, and any follow-up work item.
3. End the turn.

## Red flags

- Shipping with a task not merged or a requirement without evidence.
- A write to an existing table before its rollback version is recorded.
- A backfill without a stated window and cost.
- A breaking change shipped without a deprecation window.
- Closing the item with post-delivery checks still unchecked.
- Any production command run without a yes for that command.
