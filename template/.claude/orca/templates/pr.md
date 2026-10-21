<!-- Title: <type>(<scope>): <outcome> (<item> T<n>), following docs/conventions.md -->

## What changed
<!-- Two or three lines, in domain language. -->

## Task
Item: <item id> · Task: T<n> · Plan: specs/<n>-<slug>/4-plan.md

## Requirements covered
<!-- Ids only. -->
R1, R3

## Contract
<!-- File and change: "src/<pkg>/schemas/bronze_shop_orders.yml created" or "models/marts/mart_x.yml: column added". Or "unchanged". -->

## Testing
<!-- Last lines of each output; never "tests pass" alone. -->
```
<verify command>
<output>
<suite, lint, floor guard outputs>
```
- R1: <test name> · R3: <test name>
- Self-review: done by the author agent in its own context (no fresh-context reviewer). <what it fixed>

## Risk
<!-- What could go wrong in data terms: grain, consumers, cost. Low / Medium / High and why. -->

## How to revert
<!-- Data: the table version to return to, and the reprocessing plan. Code: revert this PR. -->

## Noticed, not touched
<!-- Out-of-scope problems seen while working. "—" if none. -->
