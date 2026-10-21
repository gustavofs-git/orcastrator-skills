---
name: data-modeling
description: Designs tables and their relations - grain, keys, layers, facts and dimensions, history (SCD), event vs processing time, timezones and NULL meaning. Use when designing a new table or model, choosing a grain, joining to a dimension that keeps history, or when counts double or drift after a join.
---

# Data modeling

Most data bugs are modeling bugs: a join that changes the grain, a dimension joined without its validity dates, a timestamp in the wrong timezone. Decide these on paper before writing SQL.

## When to use

- Designing a new table or model.
- Joining to a dimension that keeps history.
- Choosing between event time and processing time.
- Counts that double, or totals that drift after a join.

## Step 1: Grain first

Write the grain as a sentence before anything else: "one row per order line", "one row per store per day". Then:
- name the key that enforces it, and test it for uniqueness;
- for every join, ask: "can this join return more than one row per left row?" If yes, the grain changed. Aggregate first, or filter to one row.

## Step 2: Place it in a layer

Follow the layers in `docs/architecture.md`. Typical shapes:

| Layer | Does | Never does |
|---|---|---|
| Raw / bronze / staging | lands or renames, types, filters junk; one table per source entity | joins, business logic |
| Cleaned / silver / intermediate | deduplicates, conforms, joins within a domain | serves consumers directly (unless the repo says so) |
| Serving / gold / marts | facts, dimensions and aggregates for consumers | reads raw data directly |

A new layer or a new pattern is an ADR (see `writing-adrs`).

## Step 3: Facts and dimensions

- **Fact:** an event or measurement at a declared grain, with foreign keys and additive measures.
- **Dimension:** descriptive attributes of an entity.
- Keep measures additive where possible. Store ratios as numerator and denominator.

## Step 4: History

| Need | Pattern |
|---|---|
| only the current value matters | overwrite (type 1) |
| facts must use the value valid at the time | keep history with `valid_from`, `valid_to` (type 2) |

Joining to a type 2 dimension **always** filters on validity: `fact.event_ts >= dim.valid_from and fact.event_ts < dim.valid_to`. Without it, every change in the dimension duplicates the fact.

## Step 5: Time

- **Event time:** when it happened. **Processing time:** when you loaded it. Business questions use event time. Incremental cursors often use update or processing time. Say which column is which.
- Store timestamps in UTC unless the repo says otherwise. Convert at the edge, for the consumer.
- A "day" needs a timezone. Test the boundary (23:30 local) and daylight-saving days.

## Step 6: NULL and unknowns

- Decide what NULL means per column: unknown, not applicable, not arrived yet.
- A fact row whose dimension key has no match: keep it with an explicit unknown member (`UNKNOWN`, `-1`), and count those rows in a check. Never drop it silently with an inner join.

## Step 7: Partitioning

Partition by what consumers filter on and what late data rewrites, usually the event date. Too fine (per hour on small data) creates small files; too coarse makes every rewrite expensive.

## Red flags

- A table without a one-sentence grain.
- A join to a history table with no validity filter.
- `DISTINCT` added after a join to "fix" duplicates.
- An inner join that can drop facts with missing dimensions.
- A date derived from a timestamp without stating the timezone.
- Ratios stored only as ratios.

## Verification

- [ ] Grain stated and its key tested for uniqueness.
- [ ] Every join checked for grain change.
- [ ] History joins filter on validity dates.
- [ ] Event time, processing time and timezone are explicit.
- [ ] Unmatched keys go to an unknown member and are counted.
