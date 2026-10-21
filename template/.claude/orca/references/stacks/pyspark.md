# Stack profile: PySpark

Loaded after `docs/architecture.md` names `pyspark` as a profile. The commands in `docs/architecture.md` always win over the typical forms below; use these only to recognize and fill them.

## Commands (typical forms)

| Purpose | Command |
|---|---|
| focused tests | `pytest tests/<area> -q` (prefixed by the package runner, such as `uv run`) |
| one test | `pytest tests/<area>/test_<name>.py::<test> -q` |
| suite | `pytest -q` |
| lint and format | `ruff check <paths>` · `ruff format --check <paths>` |
| type check (if the repo uses it) | `mypy <package>` or `pyright` |
| run a job in dev | the orchestrator's or repo's run command, with the dev target |
| consumers | no built-in lineage: search the repo for the table name, and read the orchestrator's asset or job graph |

Never run a job with a target or config that matches `production.command_patterns` in `.claude/orca/config.json`.

## Contract

A versioned schema file next to the job, from `.claude/orca/templates/schema.yml`. Its keys mirror dbt's model YAML so both profiles read the same way. The job uses it to validate its output before writing; a contract test fails if the output and the file differ.

```python
# tests/contracts/test_contract_bronze_shop_orders.py
def test_contract_bronze_shop_orders(spark, orders_fixture):
    expected = load_contract("src/<pkg>/schemas/bronze_shop_orders.yml")   # -> StructType
    output = build_bronze_orders(spark, orders_fixture)
    assert output.schema == expected
```

- `load_contract` is a small shared utility that turns the YAML into a `StructType` (with nullability). It is built once, in the first item that needs it.
- Readable docs are generated from the schema files, never written by hand.
- Never `mergeSchema` or automatic schema evolution on writes to contracted tables.

## Tests

| Level | PySpark form |
|---|---|
| pure transformation | pytest on small DataFrames built from hand-written rows |
| component | the job against a local SparkSession and a temp warehouse dir |
| contract | the schema test above |
| integration | the job in dev on a small window (`/orca-qa`, phase 2) |
| runtime checks | the framework the project chose (`docs/testing.md`) |

Local session fixture:

```python
# tests/conftest.py
@pytest.fixture(scope="session")
def spark(tmp_path_factory):
    return (
        SparkSession.builder.master("local[2]")
        .config("spark.sql.session.timeZone", "UTC")
        .config("spark.sql.shuffle.partitions", "2")
        .config("spark.sql.warehouse.dir", str(tmp_path_factory.mktemp("warehouse")))
        .getOrCreate()
    )
```

Test shape:

```python
def test_r2_same_event_two_batches_one_row(spark):
    batch_1 = spark.createDataFrame([("e1", "2026-09-01T10:00:00Z", "v1")], EVENT_SCHEMA)
    batch_2 = spark.createDataFrame([("e1", "2026-09-03T08:00:00Z", "v2")], EVENT_SCHEMA)
    table = upsert_events(upsert_events(empty_events(spark), batch_1), batch_2)
    rows = table.collect()
    assert len(rows) == 1
    assert rows[0]["payload"] == "v2"
```

- Build DataFrames with an explicit schema, never inferred.
- Compare whole outputs with `pyspark.testing.assertDataFrameEqual` (Spark 3.5+) or by collecting sorted rows.
- Inject the clock and run date as parameters; no `current_timestamp()` in logic under test.
- No network: API clients are tested against synthetic responses.

## Fixtures

Synthetic records in `tests/fixtures/<source>/` (JSON or small CSV), one per case, built from `docs/sources/<source>.md`. Never a recorded real response.

## Floor guard patterns (PySpark)

Flag in a diff, unless the spec names the change:
- a new `dropDuplicates()` or `.distinct()` after a join or before a write;
- a new `F.coalesce`, `try_cast`, or a cast to `string` on a column that was failing;
- a new `.filter(...)` or `.na.drop()` that removes the rows a test or check was failing on;
- `mergeSchema` set to `true`, or schema auto-merge enabled;
- `pytest.mark.skip` / `xfail` added; an `assert` removed; `rtol` / `atol` or a threshold loosened;
- `except Exception: pass`, or a broad `except` that swallows a failure;
- a fixture changed in the same PR as the code it tests, so that a wrong output passes.
