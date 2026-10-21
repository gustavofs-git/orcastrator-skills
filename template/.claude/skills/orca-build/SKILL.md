---
name: orca-build
description: Implements one task of an approved plan. Hands the task's context package to the data-engineer agent, which works in an isolated worktree with TDD and opens a draft PR with code, tests and contract together. Also fixes a PR after /orca-qa asks for changes.
argument-hint: <item id> [task id]
disable-model-invocation: true
effort: medium
---

# orca-build

Get one task implemented by the `data-engineer` agent. You prepare its input and record the result; it does the work in its own worktree, with an empty context, reading only the task's context package.

Arguments: $ARGUMENTS (item id, then an optional task id such as `T2`, or the task's tracker id when tasks are mirrored).

## Step 0: Find the task and the mode

Read `specs/<id>-*/4-plan.md`.
- The plan Status must be `approved (<date>)`. Otherwise stop: "Approve the plan first (`/orca-plan <id>`)."
- **Which task.** If the arguments name one, use it. If they do not and the plan has one task, use that. If they do not and the plan has several, stop and list the open tasks.
- **Dependencies.** Every task in its `Depends on` must have `State: merged`. Otherwise stop and say which one is missing.

The task's `State` decides the mode:

| State | Mode |
|---|---|
| `open` | **New:** implement the task and open a draft PR. |
| `PR #<n>` | **Fix:** read the latest `orca-qa verdict` comment on PR #n (the "read PR comments" command from Platform commands in `docs/conventions.md`). If it says REQUEST CHANGES, fix on that PR's branch. If it says APPROVE, or there is none, stop: "Nothing to fix; run `/orca-qa <n>`." |
| `merged` | Stop: the task is done. |

## Step 1: Prepare the agent's input

The agent's worktree starts from the remote default branch. It cannot see files that exist only in this checkout, such as `specs/`. So copy into its prompt, verbatim:
1. the task section from `4-plan.md` (title, Covers, Requirements, Context to load, Touches, Verify);
2. the spec sections the task's "Context to load" names (usually Design and Boundaries), from `3-spec.md`;
3. from `docs/conventions.md`: the integration branch, branch naming, commit format, and the Platform commands "open draft PR" and "comment on PR";
4. the stack profile name from `docs/architecture.md`;
5. in fix mode: the PR number, its source branch, and the full verdict comment (findings and any reproduction tests to add).

Do not add your own summary or reasoning. Do not add other files. If a file in "Context to load" exists only in this checkout and not on the remote integration branch, include its content in the prompt and say so.

## Step 2: Delegate and wait

Delegate to the `data-engineer` agent with that prompt, and wait for its report. Do not implement anything yourself in this checkout.

## Step 3: Record the result

| Report | Do |
|---|---|
| `DONE` with a PR number | In `4-plan.md`, set the task's `State: PR #<n>`. |
| `FIXED` | Leave the State as it is. |
| `BLOCKED: <question>` | Leave the State as it is. Show the question. If the answer changes the contract or a requirement, the spec must change first (`/orca-spec <id>`), then build again. |

## Step 4: Report and end the turn

Reply with the agent's report as is (at most 30 lines), then the next command: `/orca-qa <PR>`, or what must happen to unblock. End the turn.

## Red flags

- Implementing or fixing code in this checkout instead of delegating.
- Paraphrasing the requirements or the verdict instead of copying them.
- Passing the whole spec, the chat or your reasoning to the agent.
- Building a task whose dependencies are not merged.
- Marking a task `merged`; only `/orca-qa` does that, after the merge.
