# Orcastrator

A `.claude/` folder you copy into a data repo. It turns Claude Code into a small data team that one person runs: someone who interviews the requester, someone who maps the terrain, an architect and a design critic, a planner, data engineers, reviewers and QA, and someone who ships and records.

**Thesis:** one engineer with the right process, guardrails and telemetry can deliver what a data team delivers, and every step leaves evidence a reviewer can audit.

Focus: data engineering first, then analytics engineering and data platform work. Stack profiles ship for **dbt / SQL warehouse** and **PySpark** as examples. They are not requirements: `/orca-init` adapts them, or writes a new profile, for the stack and platform your repo actually uses (Azure DevOps instead of GitHub, Databricks with Scala instead of dbt, ...). Claude Code only.

It adapts [agent-skills](https://github.com/addyosmani/agent-skills) by Addy Osmani (MIT) to data work.

> Status: early. The skills, agents and templates are written and not yet proven end to end. See [What is not here yet](#what-is-not-here-yet).

## The reference scenario

> A company hands you the endpoints of an API you have never seen and asks for a data system on top of it: ingestion, modeling, quality, and tables analysts can trust.

Every example uses a neutral fictional `shop` API (orders, customers, stores).

## The flow

```
ONCE PER REPO
  /orca-init      detects greenfield or brownfield → docs/ + CLAUDE.md + .claude/orca/config.json

PER WORK ITEM (any size: one new column or a whole ingestion)      specs/<n>-<slug>/
  /orca-brief     → 1-brief.md       why, for whom, success          gate: explicit "yes"
  /orca-discover  → 2-discovery.md   what exists, gaps, risk         gate: conflicts resolved
  /orca-spec      → 3-spec.md        EARS requirements R1..Rn + design   gate: Status approved
  /orca-plan      → 4-plan.md        tasks T1..Tn with context packages  gate: plan approved

  PER TASK (one task = one PR)
    /orca-build   → draft PR: code + tests + contract together (data-engineer, own worktree, TDD)
    /orca-qa      → 3 reviewers in parallel → real data in dev → verdict → merge with human approval

  /orca-ship      → delivery checklist sized to the change, CHANGELOG, build log, close the item
```

Rules that hold everywhere:
- **Only you start a stage.** Stage skills cannot be invoked by the model on its own.
- **Each stage ends by writing a file and ending the turn.** The next stage reads the file, not the chat, so you can stop and resume anywhere.
- **Approvals count only when written in an artifact.**
- **Never assume.** Every fact in `docs/` is `[detected: <file>]`, `[confirmed]` or `[unclear]`. Later stages ask when they hit `[unclear]`.
- **The contract is executable schema next to the code** (dbt model YAML with an enforced contract, or a PySpark schema file plus a contract test), never a hand-written document.
- **Two words only:** a *work item* (whatever your tracker calls it) and a *task* (one PR). No flags: each stage detects the case and asks when it cannot tell.

## What is inside

```
template/.claude/
├── skills/
│   ├── orca-init/ orca-brief/ orca-discover/ orca-spec/
│   │   orca-plan/ orca-build/ orca-qa/ orca-ship/          stage skills (you run them)
│   └── data-contracts/ data-modeling/ pipeline-code-design/ data-testing/
│       data-quality-checks/ incremental-loads-and-backfills/
│       schema-evolution-and-deprecation/ api-ingestion/ data-debugging/
│       pipeline-performance-and-cost/ verify-against-docs/ writing-adrs/   discipline skills
├── agents/        source-scout, data-engineer, architect-critic, reviewer,
│                  qa-engineer, governance-officer, tech-writer
└── orca/
    ├── config.json   filled by /orca-init
    ├── references/   definition of done, glossary, stack profiles (dbt, pyspark)
    └── templates/    brief, discovery, spec, plan, ADR, source profile, PR, issue,
                      delivery checklist, PySpark schema, stack profile, docs/ templates
```

| Agent | Role | Model |
|---|---|---|
| `source-scout` | profiles an external source; writes a summary and synthetic fixtures | sonnet |
| `data-engineer` | implements one task in its own worktree; returns `BLOCKED` instead of guessing | sonnet |
| `architect-critic` | reviews the spec with a fresh context: at most 3 problems and GO or NO-GO | opus |
| `reviewer` | five-axis code review, adapted to data | opus |
| `qa-engineer` | test gaps, mutation experiments, reproduction tests, in a throwaway checkout | sonnet |
| `governance-officer` | personal data, secrets, access, retention | sonnet |
| `tech-writer` | CHANGELOG and build log | haiku |

No agent can start another agent. The only fan-out is the three reviewers in `/orca-qa`, merged by the main session. Every agent report is at most 30 lines.

## Install

1. Copy `template/.claude/` into the root of your data repo.
2. Open Claude Code there and run `/orca-init`.
3. For each work item, start with `/orca-brief <item>`.

You need Claude Code, git, and the CLI for your code host and tracker (`gh`, `az`, `glab`, ...). `/orca-init` records the exact commands in `docs/conventions.md`.

## Adopting it in a team

You do not need every stage on day one:
1. **Brief only.** Well-defined work items; no agent writes code.
2. **Discover, spec and ADRs**, with `/orca-qa` next to the human reviewer.
3. **Plan and build** on items that passed the earlier steps.
4. **Telemetry:** cost per item and per kind of work.

The artifacts are useful without the agents: a work item with testable requirements and a context package per task is a good handoff to a junior engineer too.

## What is not here yet

Referenced by the skills but not shipped yet:
- `.claude/settings.json` (permissions, sandbox, hooks, env);
- hooks: `bash_guard.py`, `path_guard.py`, `commit_gate.py`, `dod_check.py`, `cost_log.py` and the other guards;
- scripts: the tracker adapter, `floor_guard.py`, `orca-session`;
- `CLAUDE.md.example`, pre-commit, CI, PR and issue templates for `.github/`;
- evals and the golden run;
- demo and cost numbers. They will come from the first real project built with Orcastrator, not from estimates.

Until the hooks exist, the rules they enforce are stated in the skills and agents but not enforced by the harness.

## License

MIT. See [LICENSE](LICENSE).
