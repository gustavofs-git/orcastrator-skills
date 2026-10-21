---
name: data-testing
description: Test-first development for data code - failing test with a hand-written synthetic fixture, asserting on the output table, then the code. Covers test levels (pure transformation, component, contract, integration), the Prove-It pattern for data bugs, mandatory edge cases and determinism. Use when implementing or changing any transformation, loader or model, or fixing a data bug.
---

# Data testing

Write a failing test before the code that makes it pass. For a bug, reproduce it with the smallest synthetic record first. "The numbers look right" is not done; a test that fails without the change is.

## When to use

- Implementing or changing a transformation, loader, model or job.
- Fixing a data bug (Prove-It).
- Reviewing whether a PR's tests would catch a regression.

Not for: runtime checks on real data (use `data-quality-checks`).

## Tests vs checks

| | Tests | Quality checks |
|---|---|---|
| Prove | the code is right | the data is right |
| Run | before merge, in CI | on every load, in every environment |
| Input | small synthetic fixtures | real data |
| On failure | the PR does not merge | severity decides: block publish or alert |

## Discover the stack first

Use the commands in `docs/architecture.md` and the stack profile (`.claude/orca/references/stacks/<profile>.md`). Never assume a default command. Run the focused test in the loop and the suite before you finish.

## Levels: test at the lowest one that proves the behavior

| Level | Proves | dbt | PySpark |
|---|---|---|---|
| Pure transformation | the logic over a few rows | dbt unit tests | pytest on small DataFrames |
| Component | a model or job against a local engine | `dbt build` on fixtures | local SparkSession |
| Contract | the output schema did not change silently | enforced contracts | schema test against the schema file |
| Integration | it works on the real engine | slim CI in a dev schema | job in dev on a sample (via `/orca-qa`) |

Other stacks: the stack profile's Tests table gives the form for each level.

## The cycle

1. **RED.** Write a hand-written synthetic fixture: one minimal record per case. Assert on the **output table**: rows, keys, values. Run it; it must fail, for the reason you expect.
2. **GREEN.** Write the least code that passes.
3. **REFACTOR.** Clean up with the tests green. Run them after each step.

Put the requirement id in the test name or description: `test_r2_late_update_replaces_row`. Every requirement a task covers has at least one test or check with its id.

## Prove-It for data bugs

Do not start with the fix. Build the smallest synthetic case that triggers the bug (the same `event_id` in two batches two days apart), watch the test fail on the current code, then fix the cause. A fix that adds `DISTINCT` downstream hides the bug; the test must target the code that created it.

## Mandatory edge cases (where they apply)

- NULL in a key column.
- Duplicate records in one batch, and across batches.
- Late events: older event time arriving in a newer batch.
- An empty partition or empty input.
- Timezone boundary and daylight-saving day.
- Source schema drift: a missing or extra field.
- Re-running the same window: same output (idempotency).

## Determinism

- Fixed seeds; an injected clock instead of `now()`.
- No network in tests. API clients are tested against recorded **synthetic** responses.
- Fixture policy from `.claude/orca/config.json`: `synthetic-only` by default. Never real rows, ids or payloads under `tests/` unless the policy is `anonymized-samples-allowed`.

## Writing good tests

- Assert on state (the output table), not on which functions were called.
- One behavior per test. Each test reads as a sentence.
- DAMP over DRY: a test can repeat its fixture rows if that makes it readable alone.
- Prefer a real local engine over mocks; mock only the network boundary.
- A seed or reference table is not test data unless the repo says so.

## Red flags

- A test that passed the first time it ran.
- Assertions on internal calls instead of output rows.
- A fixture edited until the test passes with a wrong output.
- `skip`, `xfail` or a loosened tolerance to get green.
- Tests that call a real API or read real data.
- A requirement with no test or check carrying its id.

## Verification

- [ ] Each new test failed before the change, for the expected reason.
- [ ] Each covered requirement has a test or check with its id.
- [ ] The edge cases that apply are tested.
- [ ] Focused tests and the suite pass, with the output shown.
- [ ] Fixtures are synthetic and minimal.
