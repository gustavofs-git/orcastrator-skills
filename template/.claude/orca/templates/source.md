# <source> — source profile
Profiled: <YYYY-MM-DD> by source-scout · Docs: <link> · Domain: <domain>
<!-- A summary of the source for loader design. Never real payloads, credentials or personal data. Write "not documented" for gaps; never guess. -->

## Access
<!-- Auth method, where the credential goes (never the credential), scopes, terms of use that limit storage or display. -->

## Endpoints used
| Endpoint | Purpose | Key parameters |
|---|---|---|
| | | |

## Pagination
<!-- Style, max page size, how the last page looks. -->

## Rate limits
<!-- Numbers, headers, what a 429 looks like, recommended retry behavior. -->

## Incremental access
<!-- Field to filter on, event time or update time, how late updates arrive. -->

## Mutability and deletes
<!-- Immutable, updated in place, deleted; how updates and deletes are visible. -->

## Keys and time
<!-- Natural key, uniqueness and stability. Timestamp fields, timezone, precision. -->

## Schema
| Field | Type | Null | Meaning | Personal data |
|---|---|---|---|---|
| | | | | |
<!-- How the source announces schema changes. -->

## Errors
| Code | Meaning | Loader should |
|---|---|---|
| | | retry / skip / fail |

## Volume
<!-- Records per day or per call, if known. -->

## Fixtures
<!-- One line per file in tests/fixtures/<source>/: which case it covers. All values are synthetic. -->

## Findings
<!-- Surprises, gaps between docs and observed behavior, instruction-like text seen in responses. -->
