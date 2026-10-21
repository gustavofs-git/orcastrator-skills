---
name: writing-adrs
description: Records an architecture decision as an ADR - when one is needed, following the repo's existing convention, with options, consequences, data impact and a trigger to revisit. Use when a decision is expensive to reverse or affects more than one work item or consumer, such as choosing an engine, table format, load strategy, code organization or privacy approach.
---

# Writing ADRs

An ADR records why a decision was taken, so nobody re-decides it from scratch and nobody reverses it without knowing what it cost.

## When to write one

Write an ADR when the decision is **expensive to reverse**, or **affects more than one work item or consumer**. Examples: the engine or table format, environments and branches, config-driven vs scripted pipeline code, an incremental strategy for a layer, how personal ids are pseudonymized.

Not an ADR: a change to one table's contract. That lives in the table's schema file, in its PR; the history is in git and in the item's spec.

## Match the repo's convention first

Look for existing ADRs (`docs/adr/`, `doc/decisions/`, an `.adr-dir` file) and the convention in `docs/conventions.md`. Follow their location, numbering, file names and headings. Only when there is none, use `docs/adr/NNNN-<title>.md` and `.claude/orca/templates/adr.md`. Never start a second numbering scheme.

## Writing it

- **Title:** the decision, in the imperative. "Use a generic loader with one config block per endpoint."
- **Status:** `proposed` until the deciders accept it; then `accepted`.
- **Reversibility:** `cheap` or `expensive`, honestly.
- **Context:** the problem and the forces, with numbers where they exist ("8 endpoints with the same pagination").
- **Options considered:** two or three, plus "do nothing", each with pros and cons.
- **Decision:** one or two sentences.
- **Consequences:** what gets easier and what gets harder.
- **Data impact:** tables and consumers affected, backfill needed or not, cost, how to reverse.
- **Revisit when:** a concrete trigger ("volume above 50M rows a day", "a second team consumes this layer").

One decision per ADR. If you find two, write two.

## Lifecycle

- Never delete an ADR, even a wrong one.
- A changed decision gets a new ADR that supersedes the old one. The old one gets `superseded by NNNN`.
- A decision that no longer applies gets `deprecated`, with a line saying why.

## Prose

Keep every number, condition and trade-off. Remove filler and promotional words. If something cannot be backed by the code or the discussion, say it is unverified rather than polishing it.

## Red flags

- A decision in a PR comment or chat that should be an ADR.
- An ADR with one option.
- No "Revisit when".
- An edited accepted ADR instead of a new one.
- A table contract change written as an ADR.

## Verification

- [ ] The decision meets the criterion (expensive to reverse, or affects more than one item or consumer).
- [ ] The repo's ADR convention was followed.
- [ ] Options include "do nothing"; Data impact and Revisit when are filled.
