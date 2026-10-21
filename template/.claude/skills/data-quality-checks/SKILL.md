---
name: data-quality-checks
description: Designs runtime checks that prove the data is right on every load - uniqueness, nulls, reconciliation with the source, freshness, volume, schema drift - each with a severity that blocks publish or alerts, plus run metadata to diagnose failures. Use when a spec needs quality checks, a table gets new consumers, or a data issue was found by a person instead of a check.
---

# Data quality checks

Tests prove the code before merge. Checks prove the data on every load, in every environment, on real rows. A check without a severity and a destination is a log line nobody reads.

## When to use

- Writing the "Quality checks" part of a spec.
- A table gets a consumer that depends on it.
- A consumer found a data problem before a check did. That is a missing check; add it, and a line to `docs/review.md`.

## Step 1: Define "correct" before checking

From the spec and the contract: the grain and key, the expected relation to the source (equal counts? sums within 0.1%?), the freshness target, the allowed NULLs. A check that does not trace to one of these is noise.

## Step 2: Choose the checks

| Check | Catches | Typical severity |
|---|---|---|
| unique(key) | grain broken, duplicates | error |
| not_null(key and required columns) | lost joins, bad parsing | error |
| accepted values / ranges | bad codes, unit mix-ups | error or warn |
| relationships (foreign key exists) | orphan facts | warn, with a count |
| reconciliation with the source | lost or extra rows, wrong sums | error |
| freshness (max event or load time) | stalled loads | warn, then error past the SLA |
| volume vs recent loads | partial loads, upstream outages | warn |
| schema drift on input | the source changed | error at ingestion |
| distribution of a key measure | silent logic changes | warn |

Tie each check to a requirement where one exists: "reconciliation vs `fct_orders`, last 7 days: error (R2)".

## Step 3: Set severity

- **error:** the data must not be published (write-audit-publish stops) or the run fails.
- **warn:** published, and someone is alerted.
- Severity goes up when a check catches a real problem. It never goes down without a written reason in the spec. Lowering error to warn to get a green run is lowering the bar.
- Thresholds are numbers with units, decided in the spec, not tuned until the check passes.

## Step 4: Route alerts

Each warn and error goes to a named destination (channel, on-call, ticket queue) from `docs/testing.md`. If the repo has no alerting, the check result is at least in the run log with the run id, and that gap is `[unclear]` in `docs/`.

## Step 5: Record run metadata

Every load logs, in structured form: run id, source, window or watermark, rows read, rows written, rows quarantined, duration, and check results. When a check fails, this is what makes it diagnosable.

## Step 6: Verify the check itself

A check that cannot fail is useless. In dev or in a test, feed it a known-bad row (a duplicate key, a stale timestamp) and see it fire with the right severity.

## Service levels

Freshness and completeness are the data's service levels. Write them as numbers in `docs/product.md` or the contract ("orders D-1 complete by 08:00"), and make the freshness check enforce them.

## Red flags

- A check without severity, or with no alert destination.
- Severity lowered from error to warn in a PR.
- A tolerance widened until the run is green.
- Checks only on the final table, never at ingestion.
- No reconciliation with the source for a new load.
- A check nobody has ever seen fail.

## Verification

- [ ] Every check traces to the contract or a requirement.
- [ ] Every check has a severity and a destination.
- [ ] Reconciliation and freshness exist for every new load.
- [ ] Each new check was seen failing on a bad row.
- [ ] Runs log their metadata.
