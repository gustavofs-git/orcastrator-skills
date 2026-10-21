---
name: data-contracts
description: Defines and changes a table's contract as executable schema next to the code (for example a dbt model YAML with an enforced contract, or a schema file plus a contract test). Use when creating a table, changing columns, types, grain or keys, writing the contract block of a spec, or reviewing whether a PR's schema file matches its code.
---

# Data contracts

A table's contract is what its consumers can rely on: grain, keys, columns with types, what NULL means, partitioning, personal data class and freshness. In Orcastrator the contract is **executable and lives next to the code**. It is never a hand-written document, because a document goes stale the moment someone changes the table and forgets it.

## When to use

- Creating a table, or changing its columns, types, grain or keys.
- Writing the contract block of a spec.
- Implementing a task that touches a table.
- Reviewing a PR: does the schema file match the code and the spec?

Not for: deciding whether a change breaks consumers and how to migrate them (use `schema-evolution-and-deprecation`).

## Where the contract lives

| Stack | Contract | What fails when it diverges |
|---|---|---|
| dbt | the model's YAML: `config: contract: enforced: true`, every column with `data_type`, `description`, and `config: meta` for grain, PII and freshness | `dbt build` fails if the model's columns or types differ |
| PySpark | a versioned schema file next to the job (`.claude/orca/templates/schema.yml`), which the job uses to validate its output | a contract test fails if the output schema differs from the file |
| any other stack (Scala Spark, SQL scripts, ...) | what the repo's stack profile defines; if it defines nothing, propose options (a schema file plus a contract test, engine-enforced constraints) and let the user choose | the contract test, or the engine's enforcement |

Details and examples are in the stack profile (`.claude/orca/references/stacks/<profile>.md`). Readable docs are generated from the contract (`dbt docs`, or the profile's script), never written by hand.

## What every contract states

- **Grain:** one row per what. "One row per order line", not "orders".
- **Keys:** the columns that are unique together. Tested.
- **Columns:** name, explicit type, nullability, meaning. No schema inference in production.
- **NULL meaning:** "unknown", "not applicable" or "not yet arrived" are different things. Say which.
- **Partitioning or materialization**, when it matters to consumers (pruning, late data).
- **Time:** event time vs processing time; timezone (store UTC unless the repo says otherwise).
- **Personal data class**, per column, as `docs/architecture.md` defines the classes.
- **Freshness:** a number ("D-1 by 08:00", "within 2 h").

## Rules

1. **Contract first.** Write the contract block in the spec before the code. The build changes the schema file in the same PR as the code; the reviewer sees code, tests and contract together.
2. **Every observable behavior becomes a contract** once someone depends on it (Hyrum's law): column order, a NULL that was never documented, sort order, a timezone. Expose only what you mean to keep.
3. **Prefer addition over modification.** A new nullable column is safe. A renamed, retyped or removed column breaks consumers.
4. **Validate at the boundary.** Data from an external source is untrusted: check its shape and types when it enters, and decide what happens to a bad row (fail, quarantine, or continue with an alert). Inside the pipeline, trust the contract.
5. **Two changes to the same table** touch the same schema file. If they happen at the same time, git shows the conflict instead of two documents silently disagreeing.
6. **Specs describe the change, not the state.** Read the current contract from the schema file, never from an old spec.

## Contract test (when the stack does not enforce it)

A test that reads the schema file and compares it with the output of the job on a small synthetic fixture: same column names, same order if consumers rely on it, same types, same nullability. It fails if either side changes alone.

## Red flags

- A table with no contract, or a contract in a wiki page.
- `string` or JSON columns where the type is known, without a written reason.
- A schema file changed in a different PR from the code that needs it.
- Schema inference or automatic schema evolution turned on in production.
- A key that is not tested for uniqueness.
- "Deduplicated" or "clean" in a description without the key or rule.

## Verification

- [ ] The contract states grain, keys, types, NULL meaning, PII class and freshness.
- [ ] It is enforced by the engine or covered by a contract test, as the stack profile says.
- [ ] The schema file, the code and the spec's contract block agree.
- [ ] Personal data columns carry their class.
