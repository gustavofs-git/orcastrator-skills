---
name: schema-evolution-and-deprecation
description: Changes a table's schema without breaking its consumers - additive vs breaking changes, contract versions, impact analysis from lineage, expand-then-contract migrations, deprecation windows and notices, and handling source schema drift. Use when renaming, retyping or removing a column or table, when a source changes its schema, or when retiring a table.
---

# Schema evolution and deprecation

Once a table has consumers, every column, type and quirk is something someone depends on, including the ones you never documented. Changing them is a migration, not an edit.

## When to use

- Renaming, retyping or removing a column, or changing a table's grain.
- Retiring a table or a job.
- A source changed its schema.

## Step 1: Classify the change

| Change | Kind |
|---|---|
| add a nullable column; add a table | additive: safe |
| widen a type (int → bigint) | usually additive; check consumers that cast |
| rename, retype, remove a column; change grain or keys; change NULL meaning | **breaking** |
| change a column's meaning but not its name | **breaking**, and the most dangerous |

## Step 2: Find who depends on it

Use lineage (the stack profile's command) for consumers inside the repo. For consumers outside it (BI tools, other teams' jobs, exports), check the contract's owners and ask. List every consumer in the spec with its owner.

## Step 3: Migrate in phases (expand, migrate, contract)

Never change a column in place in one release.
1. **Expand:** add the new column next to the old one, filled with the same values. Both are valid.
2. **Migrate:** move every consumer you own to the new column. Add a CI rule that fails if a model in this repo still reads the old one.
3. **Announce:** notify every consumer of the removal date.
4. **Contract:** after the window, remove the old column in a separate release, as a separate work item created now with its date.

Each phase is its own release and can be reverted alone. The schema file changes in each phase's PR, and the contract version goes up when the break lands (v1 → v2).

## Deprecation window and notice

- **Default to advisory:** consumers migrate on their timeline, with a date. Use a hard deadline only when keeping the old shape costs more than the migration (security, wrong data).
- **Window length** is agreed with the consumers and written in the spec.
- **Notice:** what changes, the replacement, the removal date, how to migrate, and who to ask. Posted where the consumers will see it; posting is an external action that needs the user's yes.
- **You own the migration** of what you deprecate. Do not announce and leave consumers to figure it out.

## Source schema drift

When an external source changes its schema:
- Detect it at ingestion: compare the input with the expected schema on every run.
- Decide in the spec: fail the run, quarantine the rows, or continue and alert. Never turn on automatic schema evolution to make the error go away.
- An additive source change becomes a work item to decide whether to expose the new field.

## Removing a table

Verify zero readers first (lineage, query logs if available). Then remove the code, tests, contract and docs in one PR, and note it in the changelog.

## Red flags

- A rename or retype in the same release that removes the old column.
- A breaking change with no list of consumers.
- A removal date with no work item to do it.
- Automatic schema evolution enabled in production.
- A column whose meaning changed but whose name did not.

## Verification

- [ ] The change is classified; if breaking, every consumer is listed with an owner.
- [ ] Old and new shapes coexist for the agreed window.
- [ ] A CI rule stops new reads of the deprecated column.
- [ ] The removal is a separate work item with a date.
- [ ] The notice was posted (with the user's yes) and is linked in the delivery checklist.
