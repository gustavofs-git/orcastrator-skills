# Definition of Done

The same bar for every task, every time. A task's requirements say "did we build this thing?". The Definition of Done says "is it finished to our standard?". A task is done only when both hold.

## Every task (every PR)

- [ ] A test fails without the change and passes with it.
- [ ] Tests, lint and floor guard are green, **with the output shown** in the PR.
- [ ] Every covered requirement has a test or check with its id in the name or description.
- [ ] The contract is unchanged, or updated **in the same PR**, as the spec says.
- [ ] No real data, credential or personal identifier committed (code, fixtures, logs, docs).
- [ ] An ADR exists if a decision is expensive to reverse or affects more than one work item or consumer.
- [ ] The PR links the item, names the task, and has Testing, Risk and How to revert.
- [ ] `/orca-qa` gave APPROVE, and a human approved the PR.

## Every work item (at `/orca-ship`)

- [ ] Every task is merged.
- [ ] Every requirement has evidence: a test, a check, or real-data evidence from `/orca-qa`.
- [ ] The delivery checklist built from the spec is complete, with evidence on each line.
- [ ] `CHANGELOG.md` and `docs/build-log.md` have their entries.
- [ ] `docs/` changed only if a pattern or decision changed.

## Red flags

- "Done, I just haven't run it yet." Unverified work is not done.
- "Tests pass" with no output.
- A different bar under deadline pressure.
- "Done" before human approval.

Adapted from `references/definition-of-done.md` in agent-skills (MIT, Addy Osmani).
