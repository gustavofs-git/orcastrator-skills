---
name: orca-plan
description: Turns one approved spec into tasks T1..Tn in specs/<n>-<slug>/4-plan.md. One task is one PR; each covers at most 3 requirements and carries its own context package. Marks tasks that can run in parallel only with proof. Run after /orca-spec is approved.
argument-hint: <item id>
disable-model-invocation: true
effort: high
---

# orca-plan

Split the approved spec into tasks the build can do without the chat. One task is one PR. Each task carries exactly what its implementer needs to read.

Work item: $ARGUMENTS

Read [task-rules.md](task-rules.md) before Step 2.

**Read-only until the end.** This stage reads the spec and the code tree and writes exactly two things: `4-plan.md`, and the Task column of the spec's Traceability table. Nothing else.

## Read first

- `specs/<id>-*/3-spec.md`.
- `docs/conventions.md` and the commands in `docs/architecture.md`.
- The code tree (`git ls-files`), and the files the spec names as patterns to follow.
- The stack profile named in `docs/architecture.md`, for test commands.
- `specs/<id>-*/4-plan.md`, if it exists.

Do not read other items' specs.

## Step 0: Check the gates

Stop and say what is missing if any of these hold:
- the spec Status is not `approved (<date>)`;
- a requirement has no `R` id. Say: "Fix the spec first: run `/orca-spec <id>`.";
- the spec is vague where a task depends on it (an undefined key, an unstated failure decision). Ask the question. Do not guess.
- an open question has `blocks: plan`.

**Plan already exists.** Never overwrite tasks that have `State: PR #<n>` or `State: merged`. If the spec changed, add, change or remove only `State: open` tasks, and say what changed. If the existing plan is for different work, stop and ask.

## Step 1: Decide whether to split

A small change is one task. Split only when the change does not fit one reviewable PR (about 300 changed lines as a guide) or covers more than 3 requirements.

## Step 2: Slice vertically

In data, a slice is a thin path from the source to the table, tested end to end. Not "all the schemas, then all the loaders".
- Shared utilities (logger, API client, generic loader) come first, as their own task, before the tasks that use them.
- The core is validated early. High-risk tasks go early.
- The last task wires everything together (for example, the job in the orchestrator).
- **No orphan code.** Every task connects to what exists or to what a later task uses.
- **Only work that becomes a PR.** Writing the backfill code is a task. Running the backfill, notifying consumers and publishing belong to `/orca-ship`.

Write each task following [task-rules.md](task-rules.md).

## Step 3: Check coverage

Every requirement ends in exactly one of these states:
- covered by a task (`Covers: R1, R4`);
- `already covered: <test>`, when existing code meets it;
- deferred with a reason. A deferred requirement becomes an open question or a new work item.

Then fill the **Task** column of the spec's Traceability table with the task id, `already covered: <test>`, or `deferred: <reason>`. Change nothing else in the spec. This does not reset its approval.

## Step 4: Mark parallel tasks

A task gets `(P)` only when all four conditions in task-rules.md hold. Otherwise it gets `Depends on: T<n>` with the reason.

## Step 5: Write the plan and end the turn

1. Copy `.claude/orca/templates/plan.md` to `specs/<n>-<slug>/4-plan.md` with `Status: draft`. Fill it in and remove the template comments.
2. Reply with:
   - a table: task, outcome, covers, depends on;
   - "Why I split it this way", in two or three lines;
   - requirements already covered or deferred;
   - "Approve the plan?"
3. End the turn.

When the user approves explicitly in a later turn:
1. Set `Status: approved (<date>)`.
2. If `tracker.mirror_tasks` is `true` in `.claude/orca/config.json`, offer to create one tracker item per task. This is an external action: only after an explicit yes. Write each tracker id next to its task.
3. Reply with the next command: `/orca-build <n>` for a single task, or `/orca-build <n> T1`.

## Red flags

- A task described with function, class or file names instead of a domain outcome.
- A task covering more than 3 requirements.
- `(P)` on two tasks that touch the same file, table or schema file.
- A requirement in no task, not covered and not deferred.
- A task that runs a backfill or notifies consumers.
- Changing anything in the spec besides the Task column.

## Before you end the turn

- [ ] Every requirement is covered, already covered, or deferred with a reason.
- [ ] Every task has all the fields in task-rules.md, with requirements copied verbatim.
- [ ] Every `(P)` passes the four conditions; every other task has `Depends on` with a reason.
- [ ] The Task column in the spec's Traceability is filled.
- [ ] The plan says why it is split this way.
