# How to write `docs/`

`docs/` is the project's memory, not an inventory. The test: **if new code follows the patterns, `docs/` does not need to change.**

## What goes in, what stays out

**Write:**
- organization patterns: layers, where each kind of thing lives;
- naming conventions;
- architecture decisions **with the reason**;
- technology standards;
- exact commands.

Each pattern comes with one short example taken from this repo.

**Do not write:**
- lists of tables, models or files;
- every dependency;
- implementation details;
- the `.claude/` folder;
- secrets, connection strings, internal hosts or IPs.

| Bad (inventory) | Good (pattern) |
|---|---|
| `stg_shop__orders.sql`: orders staging<br>`stg_shop__customers.sql`: customers staging<br>... (80 models) | **Staging** (`models/staging/<source>/`): one model per source table, named `stg_<source>__<entity>`. Only renames, casts and filters junk; never joins. Example: `stg_shop__orders.sql`. |

## Shape

- One subject per file. At most 200 lines; no minimum.
- Short lines. One fact or rule per line, so each line can carry its own marker.
- Commands are written exactly as they run, never as a tool name alone.

## Markers

Every fact carries one marker, at the end of its line:

| Marker | Use when | Example |
|---|---|---|
| `[detected: <file>]` | read literally from a file; the path is the proof | `targets: dev, ci, prod [detected: profiles.yml.example]` |
| `[confirmed]` | the user confirmed it, including your interpretations | `prod is production; only deploy.yml writes to it [confirmed]` |
| `[unclear]` | nobody confirmed it | `D-1 or intraday? [unclear: product decides]` |

Rules:
- **Interpretation is not fact.** "92% of the last 200 commits use Conventional Commits" is a fact. "The team standard is Conventional Commits" is an interpretation: it is `[confirmed]` only after the user confirms it, otherwise `[unclear]`.
- **Guesses never fill the document.** If the user does not answer or says "I don't know", the line is `[unclear: <question> · <who decides>]`.
- A count or a path you computed (`0 unit tests`) is `[detected: <what you searched>]`.

## What goes where

| File | Subject |
|---|---|
| `product.md` | what the product is, for whom, what it is not, domains and owners |
| `architecture.md` | stack, repo topology, layers, environments, deploy pipeline, commands, stack profile, security and governance, operations |
| `conventions.md` | code organization, naming, git and PR process, tracker, where contracts and ADRs live |
| `testing.md` | test levels, test data, local vs CI, runtime checks |
| `review.md` | what this repo adds to the review: what is Critical here, paths to ignore, warnings, rules from incidents |
| `roadmap.md` | first work items and their order (greenfield) |
| `build-log.md` | one entry per delivered work item (written by `/orca-ship`) |
| `adr/NNNN-<title>.md` | one decision each (follow the `writing-adrs` skill) |
| `sources/<source>.md` | one external source each (written by `source-scout`) |

If something fits no file, it is probably inventory or implementation detail. Leave it out.

## Changing an existing file

- Keep the sections and examples the user wrote.
- Add rather than replace. When a rule changes, change the line and say why.
- At the end of the file, keep a `## Changes` list with one line per update: `<YYYY-MM-DD> — <what changed> — <why>`.
- A marker changes only for a reason: `[unclear]` → `[confirmed]` when the user answers; `[unclear]` → `[detected: <file>]` when the code now shows it.
