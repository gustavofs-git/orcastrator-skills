# Orcastrator glossary

Terms every stage uses. Read this when a word in a skill or artifact is unclear.

## Units of work

- **Work item.** What you run `/orca-brief` on, of any size and whatever your tracker calls it. One folder in `specs/`.
- **Task.** One piece of a work item's plan (`T1`, `T2`, ...). One task is one PR.
- **Item folder.** `specs/<n>-<slug>/`. `<n>` is the tracker id (or `001`, `002`, ... with no tracker); `<slug>` is 2–4 lowercase English words.
- **Stage.** One of the eight `orca-` commands. Each stage writes one artifact and ends the turn; the next stage reads the file, not the chat.

## Markers in `docs/`

| Marker | Meaning |
|---|---|
| `[detected: <file>]` | read literally from that file |
| `[confirmed]` | the user confirmed it, including interpretations |
| `[unclear]` | nobody confirmed it; comes with the question still to answer |

A stage that depends on an `[unclear]` fact asks before going on. When it gets an answer, it updates `docs/` and the marker becomes `[confirmed]`.

## Status values

| Artifact | Values |
|---|---|
| `1-brief.md` | `draft` · `confirmed (<date>)` |
| `2-discovery.md` | no status; each conflict records its resolution and date |
| `3-spec.md` | `draft` · `draft (requirements approved <date>)` · `approved (<date>)` · `superseded` |
| `4-plan.md` | `draft` · `approved (<date>)` |
| each task in `4-plan.md` | `State: open` · `State: PR #<n>` · `State: merged` |

An approval counts only when it is written in the artifact. It is never inferred from an earlier conversation.

## Open questions

- **Open question.** Something nobody could answer yet. Has an owner, a `blocks:` value and paste-ready text.
- **blocks.** The earliest stage that cannot proceed without the answer, or `not blocking`.

## Requirements and tasks

- **R1..Rn.** Requirement ids, defined in `3-spec.md` in EARS form. Everywhere else only the id appears; the task's context package copies the text verbatim.
- **T1..Tn.** Task ids, defined in `4-plan.md`.
- **Covers.** The requirement ids a task implements, at most 3.
- **already covered: <test>.** A requirement that existing code already meets.
- **(P).** A task that can run in parallel; it passes the four conditions in `orca-plan/task-rules.md`.
- **Context package.** The part of a task that tells its implementer exactly what to read, what it touches and how to verify. The implementer reads nothing else.
- **Integration branch.** The branch task branches start from and merge into, named in `docs/conventions.md`.

## Contracts, tests and checks

- **Contract.** A table's grain, keys, columns, types, NULL meaning, PII class and freshness, as executable schema next to the code, in the form the stack profile defines (for example a dbt model YAML with an enforced contract, or a schema file plus a contract test). Never a hand-written document.
- **Stack profile.** `.claude/orca/references/stacks/<name>.md`: this repo's commands, test levels, contract pattern and floor guard patterns. The shipped dbt and pyspark profiles are examples; `/orca-init` adapts them or writes a new one.
- **Platform commands.** The table in `docs/conventions.md` with the exact commands for this repo's code host and tracker (open PR, comment, merge, close item, ...). Stages use it instead of assuming `gh`.
- **Test.** Proves the code is right, before merge, on synthetic fixtures.
- **Check.** Proves the data is right, on every load, on real data, with a severity (`error` or `warn`).
- **Floor guard.** Detects a diff that lowers the bar to get green (a new skip, a lowered severity, a `DISTINCT` hiding duplicates).

## Review

- **Labels.** `Critical` (blocks merge) · `Required` (fix before merge) · `Optional` (worth considering) · `Nit` (author may ignore).
- **Verdict.** `APPROVE` or `REQUEST CHANGES`, posted by `/orca-qa` as a comment starting `orca-qa verdict:`. It never replaces human approval.
- **BLOCKED.** What the `data-engineer` returns, with a question, when it cannot go on without a decision.
