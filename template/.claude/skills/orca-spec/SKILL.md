---
name: orca-spec
description: Writes the specification of one work item in specs/<n>-<slug>/3-spec.md. Assumptions first, then EARS requirements R1..Rn, then the design (contract changes, load semantics, quality checks, privacy), reviewed by the architect-critic and approved by you. Run after /orca-discover.
argument-hint: <item id>
disable-model-invocation: true
effort: high
---

# orca-spec

Write what this work item must do (requirements) and how (design) before any code exists. The spec passes a gate: the user approves it, and `Status: approved` is written in the file.

Work item: $ARGUMENTS

Supporting files, read when the step needs them:
- [ears.md](ears.md): requirement patterns and rules (Step 3).
- [design-rules.md](design-rules.md): design rules and diagrams (Step 4).

## Read first

- `specs/<id>-*/1-brief.md` and `2-discovery.md`.
- ADR titles only: the first line of each file in `docs/adr/`. Open an ADR only if the discovery cites it.
- The contracts the discovery cites.
- `docs/review.md`, for what counts as Critical here.

Do not read code beyond what the discovery cites.

## Step 0: Check the gates

Stop and tell the user what is missing if any of these hold:
- the brief is `draft`, or has an unanswered open question with `blocks: spec`. Show its paste-ready text and say: "When you have the answer, run `/orca-brief <id> <answer>`."
- the discovery is missing or has an `unresolved` conflict;
- the item depends on an `[unclear]` fact in `docs/` with no answer. Ask about it now, one question with a guess, and update `docs/`.

**Spec already exists.** If `3-spec.md` is `approved` and the user or the build needs a change, edit it, set `Status: draft`, write what changed under the header, and go through the critic and approval again. If a new spec replaces it entirely, set the old one to `superseded`.

## Step 1: Propose a split, if needed

If the item bundles things that can be delivered separately (each has value alone and can reach production alone), propose splitting it into separate work items before writing anything. List each with its outcome and say why. Each new item gets its own folder in `specs/` and starts at `/orca-brief`. Creating tracker items is an external action: only after an explicit yes.

## Step 2: State assumptions

List what you assume, numbered, and wait for corrections before writing the rest:

```
I ASSUME THAT:
1. "Region" is the store's region, not the customer's (from the brief).
2. Valid sales are fct_orders rows with status not in ('cancelled', 'test').
3. Amounts are gross, in the currency stored in fct_orders.
Correct any of these before I write the requirements.
```

## Step 3: Write the requirements

Follow [ears.md](ears.md). Number them `R1..Rn`. Each one must become a test or a check. Cover the bad cases: the **If** pattern is where data bugs are caught (late data, nulls, duplicates, a failure halfway, a changed schema).

**Large spec:** approve in two passes in the same file. Write the requirements, set `Status: draft`, and ask for approval of the requirements alone. When the user approves, set `Status: draft (requirements approved <date>)` and continue with the design.

## Step 4: Write the design

Follow [design-rules.md](design-rules.md). Use the discipline skills the design needs: `data-contracts`, `data-modeling`, `incremental-loads-and-backfills`, `data-quality-checks`, `schema-evolution-and-deprecation`, `api-ingestion`, `pipeline-performance-and-cost`. A decision that is expensive to reverse, or that affects more than one work item or consumer, becomes an ADR (follow `writing-adrs`).

## Step 5: Fill traceability

One row per requirement: `| R | Test or check | Task |`.
- **Test or check:** the test name (with the R id in it, such as `test_r2_watermark_from_table`) or the runtime check.
- **Task:** leave `—`. `/orca-plan` fills this column.
- A requirement that existing code already satisfies gets `already covered: <test>`.

## Step 6: Run the architect-critic

Delegate to the `architect-critic` agent. Give it only file paths: `3-spec.md`, `1-brief.md`, `2-discovery.md`, `docs/` and `docs/adr/`. Never pass your reasoning or this conversation.

Passes depend on the risk recorded in the discovery:
- **Low or Medium:** one pass.
- **High:** after any change you make in response to the critic, run a second pass.

Discuss each problem with the user. Change the spec where you both agree. A problem you decide not to fix is written under Open questions with the reason.

## Step 7: Ask for approval and end the turn

1. Copy `.claude/orca/templates/spec.md` to `specs/<n>-<slug>/3-spec.md` and fill it in, with `Status: draft`. Remove the template comments.
2. Reply with:
   - the number of requirements and the contract changes, one line each;
   - the critic's verdict and what you changed because of it;
   - the open questions;
   - "Approve the spec?"
3. End the turn.

When the user approves explicitly in a later turn, set `Status: approved (<date>)` and reply with the next command, `/orca-plan <n>`. "Looks fine" or "ok I guess" is not an approval; ask again.

## The spec stays alive

If the build finds that the contract must change, the build stops with `BLOCKED`. The spec is updated, approved again, and only then does the build continue.

## Red flags

- A requirement with "the system", "should", or a vague word without a number.
- Implementation in a requirement ("use MERGE") instead of behavior.
- Code in the design other than the contract schema.
- Approving with an unanswered blocking question.
- Passing your reasoning to the critic.
- `Status: approved` written without an explicit approval in this conversation.

## Before you end the turn

- [ ] Assumptions were confirmed before the requirements were written.
- [ ] Every requirement follows ears.md and has a row in Traceability.
- [ ] Every **If** requirement names its failure decision in the design (fail, quarantine or continue with an alert).
- [ ] Every contract change states grain, keys, columns, NULL meaning, PII class and freshness.
- [ ] The critic ran the number of passes the risk requires.
