# Review rules
<!-- Written by /orca-init and maintained by you. /orca-qa reviewers read this file on every PR.
     It adds to the five review axes; it does not repeat them. -->

## Critical in this repo
<!-- A finding of this kind blocks the merge. Keep the defaults below unless the repo has a reason not to; add repo-specific ones. -->
- duplicates the grain of a table
- loses rows silently
- exposes personal data (in a table, a log or a fixture)
- breaks the contract of a consumer
- writes outside its environment
- adds cost without a limit (an unbounded scan or backfill)

## Paths to ignore
<!-- Generated or vendored paths reviewers skip. Example: "target/", "dbt_packages/" -->

## Warnings
<!-- Known risks reviewers must keep in mind. Example: "the local sample has unmasked personal data (ticket DATA-130) [confirmed]" -->

## Rules from incidents
<!-- Every data bug that reached a consumer becomes one line here.
     <YYYY-MM-DD> — <what happened> — <rule reviewers now check> -->
