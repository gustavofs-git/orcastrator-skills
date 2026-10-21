---
name: pipeline-performance-and-cost
description: Makes pipelines faster and cheaper by measurement - run time, bytes scanned and cost per run first, then the one bottleneck (missing partition pruning, skew, shuffle, small files, full reloads, exploding joins), then re-measure and keep or revert. Use when a job is slow or expensive, a spec has a cost or time target, or a review flags an unbounded scan.
---

# Pipeline performance and cost

Measure, fix one thing, measure again. An optimization without a before-and-after number is a guess that adds complexity.

## When to use

- A job is slow, expensive, or near its time window.
- A spec has a cost or run-time target.
- A review flags a scan without a partition filter, or a `MERGE` over the whole table.

## Step 1: Measure

Get the numbers before touching code:
- run time per step;
- bytes or rows scanned, and partitions read;
- compute used and cost per run (from the warehouse's query history, the Spark UI, or the platform's billing);
- data volume per run.

Write them down. They go in the PR and, for a work item about cost, in the build log.

## Step 2: Find the bottleneck

| Symptom | Likely cause |
|---|---|
| scans the whole table every run | no partition filter; filter on a derived expression the engine cannot prune |
| one task takes most of the time | skew on a join or group-by key (NULLs or one hot value) |
| large shuffle | joins and aggregations on unaligned keys; a broadcast that should be one, or one that should not |
| many tiny files, slow reads | too-fine partitioning; many small appends; no compaction |
| run time grows every day | full reload of a growing table; a lookback window that grows |
| row count explodes mid-pipeline | a join that multiplies rows (see `data-modeling`) |
| Python step is slow | row-by-row UDFs instead of built-in functions |

Fix the biggest one first. Only one change at a time, so the next measurement tells you what it did.

## Step 3: Fix, then verify

- Make the change. The tests must still pass: a faster wrong answer is a regression.
- Measure again with the same input size. Keep the change if the gain is real; revert it if it is not.
- Common fixes: filter on the partition column directly; incremental instead of full (`incremental-loads-and-backfills`); pre-aggregate before a join; salt or isolate a skewed key; compact small files; replace a UDF.

## Step 4: Guard against regression

- Put the target in the spec as a number ("about 4 min per run").
- Where the repo allows, a check or CI job that flags scans without partition filters or cost above a threshold.
- Record the measured cost in the PR's Testing section and, for cost work, in `docs/build-log.md`.

## Red flags

- "Optimized" with no before or after number.
- Several optimizations in one change.
- A cache or materialization added without measuring the read pattern.
- A faster version that changes the output.
- An unbounded scan or backfill with no cost estimate.

## Verification

- [ ] Before and after numbers, on the same input size.
- [ ] Tests pass after the change.
- [ ] The cost or time target in the spec is met, or the gap is written.
