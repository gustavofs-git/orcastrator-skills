# Stack profile: dbt / SQL warehouse

Loaded after `docs/architecture.md` names `dbt` as a profile. The commands in `docs/architecture.md` always win over the typical forms below; use these only to recognize and fill them. Requires dbt 1.8 or later for unit tests.

## Commands (typical forms)

| Purpose | Command |
|---|---|
| focused build and test | `dbt build --select <model>` |
| unit tests of one model | `dbt test --select "<model>,test_type:unit"` |
| suite | `dbt build` (or the CI selector the repo uses) |
| parse / compile check | `dbt parse` |
| lint | `sqlfluff lint <paths>` |
| children (consumers) | `dbt ls --select <model>+ --resource-type model` |
| parents | `dbt ls --select +<model> --resource-type model` |
| slim CI | `dbt build --select state:modified+ --defer --state <artifacts path>` |
| docs from contracts | `dbt docs generate` |
| source freshness | `dbt source freshness` |

Always pass the target the command needs (`--target dev`). Never a target that matches `production.command_patterns` in `.claude/orca/config.json`.

## Contract

The model's YAML is the contract. Enforced contracts work on `table`, `view` (no constraints) and `incremental` models; not on Python models, ephemeral models or snapshots.

```yaml
models:
  - name: mart_sales_by_region
    description: Daily sales by store region, at the region valid on the sale date.
    config:
      materialized: incremental
      unique_key: [region_name, sale_date]
      on_schema_change: append_new_columns   # contracted incremental models need append_new_columns or fail
      contract:
        enforced: true
      meta:
        grain: one row per region per sale date
        pii: none
        freshness: D-1 by 08:00
    columns:
      - name: region_name
        data_type: varchar
        description: Store region valid at sale time; UNKNOWN if the store is unmatched.
        constraints:
          - type: not_null
      - name: sale_date
        data_type: date
        constraints:
          - type: not_null
      - name: gross_amount
        data_type: numeric(18,2)
        description: Gross amount in the store currency.
        config:
          meta:
            pii: none
```

- Give numeric types a precision and scale; dbt does not compare them.
- Column-level `meta` goes under `config:` (dbt 1.10, backported to 1.9). Follow the repo if it uses top-level `meta`.

## Tests

| Level | dbt form |
|---|---|
| pure transformation | unit tests (`unit_tests:` in YAML, `given` input rows, `expect` output rows) |
| component | `dbt build --select <model>` on fixtures in a dev schema |
| contract | the enforced contract above |
| integration | slim CI in a dev or CI schema |
| runtime checks | data tests (`unique`, `not_null`, `relationships`, `accepted_values`, custom) with `severity` |

Unit test shape:

```yaml
unit_tests:
  - name: test_r3_store_switching_region_keeps_old_sales
    model: mart_sales_by_region
    overrides:
      macros:
        is_incremental: false
    given:
      - input: ref('fct_orders')
        rows:
          - {order_id: 1, store_id: 7, sale_ts: 2026-02-10 10:00:00, gross_amount: 100.00, status: paid}
          - {order_id: 2, store_id: 7, sale_ts: 2026-04-10 10:00:00, gross_amount: 50.00, status: paid}
      - input: ref('dim_store')
        rows:
          - {store_id: 7, region_name: South, valid_from: 2020-01-01, valid_to: 2026-03-01}
          - {store_id: 7, region_name: Southeast, valid_from: 2026-03-01, valid_to: 9999-12-31}
    expect:
      rows:
        - {region_name: South, sale_date: 2026-02-10, gross_amount: 100.00}
        - {region_name: Southeast, sale_date: 2026-04-10, gross_amount: 50.00}
```

- `expect` lists only the columns the test is about.
- With `is_incremental: true`, `expect` is the rows to be inserted or merged, not the final table; add `input: this` for the current contents.
- Larger fixtures can live as CSV files in the test paths' `fixtures/` folder.
- **Seeds are reference data**, not test data, unless the repo says otherwise.
- Keep unit tests out of production builds (the repo's selector or `--exclude-resource-type`).

## Incremental models

- `is_incremental()` filter on the cursor column, minus the lookback window: `where updated_at >= (select max(updated_at) from {{ this }}) - interval '3 days'` (adapt the syntax to the adapter).
- `unique_key` set, so re-runs merge instead of duplicating.
- Test both modes with the `is_incremental` override.

## Floor guard patterns (dbt)

Flag in a diff, unless the spec names the change:
- a test's `severity` changed from `error` to `warn`; `warn_if` / `error_if` thresholds loosened;
- a `where:` added to a data test; a test set to `enabled: false` or removed from YAML;
- `contract: enforced` set to `false`; `on_schema_change` set to `ignore` or `sync_all_columns` on a contracted model;
- a new `select distinct`, `qualify row_number() ... = 1` or `group by` that only removes duplicates;
- a new `coalesce` over a column that was failing a test;
- an `--exclude` added to the CI selector.
