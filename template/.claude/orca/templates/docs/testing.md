# Testing
<!-- Filled by /orca-init. How this repo tests code (before merge) and checks data (at runtime). One fact per line, each with a marker. -->

## Test levels here
| Level | Proves | Here | Command |
|---|---|---|---|
| Pure transformation | logic over a few rows | | |
| Component | a model or job against a local engine | | |
| Contract | the output schema did not change silently | | |
| Integration | works on the real engine | | |
<!-- "Here" says what exists, or "none" with a marker. Example: "none [detected: no unit_tests in models/]" -->

## Test data
<!-- Where test data comes from: synthetic fixtures, downloaded samples, production clones, anonymized seeds. Where fixtures live. Any warning (e.g. a sample with unmasked personal data). -->
- fixture policy: <synthetic-only | anonymized-samples-allowed> (also in .claude/orca/config.json)

## Local and CI
<!-- What runs locally, what runs in CI, on which events. -->

## Runtime checks
<!-- Check framework, default severity, where alerts go, freshness and volume checks. -->

## Changes
<!-- <YYYY-MM-DD> — <what changed> — <why> -->
