# Conventions
<!-- Filled by /orca-init. Each rule with one short example from this repo, and a marker. -->

## Code organization
<!-- Where each kind of code lives; config-driven or scripted; shared utilities and when to use them; logging. -->

## Naming
<!-- Tables, models, files, columns. Example: "staging models: stg_<source>__<entity> [detected: models/staging/]" -->

## Contracts
<!-- Where each table's executable contract lives, as the stack profile defines it. Examples: dbt model YAML with `contract: enforced`; a schema file next to a Spark job plus a contract test. -->

## Git
<!-- Integration branch (task branches start from it), branch naming, commit format, merge strategy. -->
- integration branch:
- branch name:
- commit message:
- merge:

## Pull requests
<!-- PR template, required reviewers, CODEOWNERS paths that need a human. -->

## Tracker
<!-- Which tracker, how tickets are organized, definition of done, whether plan tasks are mirrored. -->

## Platform commands
<!-- The exact commands Orcastrator stages use for this repo's code host and tracker. Confirmed with the user.
     Example values are for GitHub (gh). On Azure DevOps, GitLab, Bitbucket or Jira, write that platform's CLI or REST call.
     An operation with no command: "manual: <who does it>". -->
| Operation | Command |
|---|---|
| view PR (title, body, source and target branch, draft) | e.g. `gh pr view <PR> --json title,body,headRefName,baseRefName,isDraft` |
| PR diff | `git fetch origin <target> <source> && git diff origin/<target>...origin/<source>` |
| open draft PR | e.g. `gh pr create --draft --base <target> --title "<title>" --body-file <file>` |
| read PR comments | e.g. `gh pr view <PR> --comments` |
| comment on PR | e.g. `gh pr comment <PR> --body-file <file>` |
| human approval status | e.g. `gh pr view <PR> --json reviewDecision` |
| mark ready for review | e.g. `gh pr ready <PR>` |
| merge | e.g. `gh pr merge <PR> --squash` (strategy from Git above) |
| PR state (open, merged) | e.g. `gh pr view <PR> --json state` |
| view work item | e.g. `gh issue view <id>` |
| create work item | e.g. `gh issue create --title "<title>" --body-file <file>` |
| comment on work item | e.g. `gh issue comment <id> --body-file <file>` |
| move work item | e.g. `gh issue edit <id> --add-label <state>` |
| close work item | e.g. `gh issue close <id> --comment "<text>"` |

## Decisions
<!-- Where ADRs live and how they are numbered. Default: docs/adr/NNNN-<title>.md -->

## Changes
<!-- <YYYY-MM-DD> — <what changed> — <why> -->
