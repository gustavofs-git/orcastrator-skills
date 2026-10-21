---
name: incremental-loads-and-backfills
description: Designs correct loads - full vs incremental, cursors and watermarks, late data, deletes, idempotent writes, resuming after a failure - and runs backfills as deploys with a window, cost estimate, rollback point and idempotency proof. Use when a load becomes incremental, a spec mentions late data or reprocessing, or wrong data must be rewritten.
---

# Incremental loads and backfills

An incremental load is a promise: the table after many small runs equals the table a full reload would produce. Most incremental bugs break that promise quietly, with duplicates or gaps that no one notices for weeks.

## When to use

- A load is new, or moves from full to incremental.
- A spec mentions late data, updates, deletes or reprocessing.
- Wrong data was published and must be rewritten.
- Running any backfill.

## Full or incremental

Prefer full when the table is small enough that a full reload fits the time and cost budget. It is simpler and correct by construction. Go incremental when the spec gives a time or cost reason, with numbers.

## Designing an incremental load

1. **Cursor.** Which column says "new or changed since last run"? An update timestamp catches updates; an event timestamp does not. Write which one, and why.
2. **Watermark source.** Read the high-water mark from the target table (`max(updated_at)`), not from a separate state file that can disagree with the data. If a state file is needed, write it in the same transaction as the data.
3. **Late data.** Measure how late records arrive. Reprocess a lookback window (`watermark - 3 days`), sized from the measurement, not from a guess. Changing the lookback needs new measurements.
4. **Idempotent write.** Running the same window twice gives the same table. Use an upsert or merge by key, or overwrite whole partitions. Never append without a key.
5. **Dedup across batches.** Deduplicate against the table, not only inside the current batch. A record re-sent two days later lands in a different batch.
6. **Deletes.** Does the source tell you about deletes (a delete feed, a soft-delete flag)? If not, plan a periodic reconciliation, and write the trade-off.
7. **Failure halfway.** After a failed run, the next run must resume without gaps or duplicates. With an idempotent write and a watermark read from the table, this comes free.
8. **Concurrency.** An incremental run and a backfill on the same table must not overlap. State it as a **While** requirement.

## Switching full to incremental

Run both side by side for a shadow period. The incremental output goes to a shadow table; compare it with the full reload daily. Switch only after the difference is zero for the agreed number of days.

## Backfills are deploys

A backfill rewrites data consumers already read. Treat it like a release:
- **Window:** exact start and end.
- **Cost:** estimated before running (bytes scanned, run time, compute), and stated to the user.
- **Rollback point:** record the table version (snapshot id, time-travel timestamp, backup) before the first write, with the exact command to return to it.
- **Idempotency proof:** the requirement and test that show re-running the window is safe.
- **Dry run:** counts of what will change, in dev or as a read-only query, before the real run.
- **Batches:** by partition, so a failure leaves a clear point to resume from.
- **Consumers:** tell them what changes ("daily counts drop by about 0.8%") before, not after.

Writing the backfill code is a task in `/orca-plan`. Running it belongs to `/orca-ship`.

## Red flags

- Append-only writes with no key.
- A watermark in a file that is not updated with the data.
- Dedup only inside the current batch.
- A lookback window with no measurement behind it.
- A backfill without a recorded rollback point or a cost estimate.
- No answer to "what happens to deletes?"

## Verification

- [ ] Cursor, watermark source and lookback are written, with the measurement.
- [ ] A test re-runs the same window and gets the same rows.
- [ ] A test resumes after a failure without gaps or duplicates.
- [ ] Late updates and deletes have a stated behavior and a test or check.
- [ ] Every backfill has a window, a cost, a rollback point and a dry run.
