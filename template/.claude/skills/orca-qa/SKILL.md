---
name: orca-qa
description: Decides whether one task's PR can be merged. Phase 1 runs three reviewers in parallel (reviewer, qa-engineer, governance-officer), merges their findings as Critical, Required, Optional or Nit, and gives APPROVE or REQUEST CHANGES. Phase 2 tests the change on real data in dev. Then merges with human approval.
argument-hint: <PR number>
disable-model-invocation: true
effort: high
---

# orca-qa

Decide whether this PR can go to merge. Two phases, always in this order: review, then real data. The Orcastrator verdict never replaces human approval.

PR: $ARGUMENTS

## Step 0: Gather the context

- The PR, with the "view PR" command from Platform commands in `docs/conventions.md`: title, body, source and target branches, draft state. The PR body names the item and task.
- From `specs/<item>-*/`: the task section in `4-plan.md`, and the spec sections it names.
- `docs/review.md`.

If the PR body does not name an item and task, stop and ask which task it implements.

Write the diff to a temporary file with the "PR diff" command (`... > "${TMPDIR:-/tmp}/orca-qa-<PR>.diff"`). Reviewers read it from there.

## Phase 1: Review (read-only)

Follows the `code-review-and-quality` approach from agent-skills, adapted to data.

### 1. Fan out

Delegate to the three reviewers **in one message, so they run in parallel**. Each gets the same input and nothing else:
- the PR number, its source and target branches, and the diff file path;
- the covered requirements, copied verbatim from the task;
- the spec sections the task names (Design, Boundaries);
- the path `docs/review.md`.

| Agent | Focus |
|---|---|
| `reviewer` | the five axes: correctness, readability, architecture, security, performance |
| `qa-engineer` | tests: every R has a test, edge cases, mutation experiments, reproduction tests |
| `governance-officer` | personal data, secrets, access, retention |

Never pass your own opinion of the PR.

### 2. Merge the reports

You merge; the reviewers never talk to each other.
1. Remove duplicates. Keep the finding with the most precise location and the highest label.
2. **Check the contract yourself:** the schema file in the diff must match the code and the spec's contract block. A mismatch is Critical.
3. Check the author's verification in the PR body: outputs of tests, lint and floor guard are shown, not just claimed. Missing output is Required.
4. Label every finding. **Critical:** blocks the merge. **Required:** fix before merge. **Optional:** worth considering. **Nit:** the author may ignore. `docs/review.md` says what is Critical in this repo.
5. Order by weight: correctness and security first. One structural problem outweighs ten nits.
6. **Problems that existed before the PR** do not block. List them under "Before this PR" and offer to create a work item for each (external action: only after an explicit yes).

Every finding cites `file:line`. Critical and Required findings carry a suggested fix.

### 3. Verdict

- **REQUEST CHANGES** if any Critical or Required finding is open. A Required finding can be deferred only with a written reason in the verdict.
- **APPROVE** otherwise. Approve when the change improves the code's health, even if it is not perfect. Never APPROVE with an open Critical.

Show the verdict to the user and ask: "Post the review to PR #<n>?" Posting is an external action. After a yes, post them with the "comment on PR" command: inline comments where the platform supports them, and the verdict comment (format below).

With REQUEST CHANGES, end the turn with: "Run `/orca-build <item> <task>` to fix, then `/orca-qa <PR>` again."

## Phase 2: Real data in dev

Only after Phase 1 is APPROVE and the user says yes to this phase.
- Run in the **dev** environment named in `docs/architecture.md`, with real data and a small window that the user agrees to.
- Never run a command that matches `production.command_patterns` in `.claude/orca/config.json`.
- Produce evidence per covered requirement: row counts, reconciliation with the source, a re-run of the same window (idempotency), and the spec's quality checks.
- Commands come from `docs/architecture.md` and the stack profile. Never assume a default command.

If the evidence fails a requirement, the verdict becomes REQUEST CHANGES, with the evidence as a Critical finding.

## Verdict comment

```
orca-qa verdict: APPROVE | REQUEST CHANGES
Item <id> · Task T<n> · Covers R1, R3
Review: <n> Critical, <n> Required, <n> Optional, <n> Nit · before this PR: <n> (<new item ids>)

Critical
- <file:line> <problem> → fix: <suggestion>
Required
- ...
Reproduction tests to add
<test code from the qa-engineer, if any>

Real data (dev, <window>)
- R1 ok: <evidence> · R3 ok: <evidence>

This verdict does not replace human approval.
```

Optional and Nit findings go in inline comments. If the platform has no inline comment command, list them at the end of the verdict comment.

## Merge

After APPROVE in both phases:
1. Tell the user the PR needs human approval. Check it with the "human approval status" command. An agent's approval never counts as human.
2. Merge only when the user does it, or says to merge after approving. Use the "merge" command and the merge strategy in `docs/conventions.md`. Mark the PR ready first ("mark ready for review") if it is a draft.
3. After the merge, set the task's `State: merged` in `4-plan.md` and tick its requirement boxes.
4. Reply with the next command: the next open task (`/orca-build <item> T<n>`), or `/orca-ship <item>` when every task is merged. End the turn.

## Red flags

- Reviewers run one after another instead of in one message.
- A finding without `file:line`, or a Critical without a fix.
- APPROVE with an open Critical.
- Phase 2 before Phase 1 is APPROVE, or without the user's yes.
- Anything run against production.
- Merging without human approval.
