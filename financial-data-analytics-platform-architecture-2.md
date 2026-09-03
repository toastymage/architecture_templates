# Financial Data Analytics Platform — Architecture & Framework Spec

## 1. Goals & Constraints

- Persist pandas DataFrames (wide, columnar, keyed by financial instrument/component identifiers such as CUSIP, SEDOL, ISIN, or account IDs) at every stage of a calculation pipeline: **interim → results → published**.
- Use **Parquet** as the on-disk format and **DuckDB** as the single query/compute abstraction for both writing and reading.
- Build a **governed dataset layer** on top of raw files: versioned, schema-checked, discoverable, queryable in SQL.
- **Storage-neutral**: local disk, NFS mount, or S3 (and by extension any S3-compatible object store) — swappable without changing application code.
- Same physical datasets must be consumable by:
  - Dynamic UI applications (low-latency, filtered/paginated SQL queries)
  - Snowflake (via external tables / stages, without duplicating data if possible)
  - Ad hoc analyst SQL (DuckDB CLI / notebooks)

The core design tension is that **Parquet + object storage has no native transactions**, so governance (atomic publishes, versioning, schema evolution, concurrent readers/writers) has to be engineered on top. The architecture below borrows the "table format" idea from Iceberg/Delta/Hudi but keeps it deliberately lightweight so DuckDB remains the only engine you need — no JVM catalog service required. If your governance needs grow, this design has a clean upgrade path to real Iceberg/Delta.

---

## 2. Layered Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│  Consumers                                                           │
│  UI Apps (SQL over DuckDB/HTTP)   Snowflake (External Tables/Stage)  │
│  Analyst SQL (DuckDB CLI/Notebook) Batch jobs / downstream pipelines │
└───────────────────────────┬────────────────────────────────────────┘
                             │  SQL (DuckDB dialect) or Snowflake SQL
┌───────────────────────────┴────────────────────────────────────────┐
│  Access Layer                                                        │
│  - DuckDB views/macros over the catalog                              │
│  - Read API (Python): DatasetReader                                  │
│  - Optional lightweight query service (FastAPI + DuckDB) for UI      │
└───────────────────────────┬────────────────────────────────────────┘
┌───────────────────────────┴────────────────────────────────────────┐
│  Governance / Table-Format Layer                                     │
│  - Dataset Catalog (metadata store: datasets, versions, schemas)     │
│  - Manifest files (atomic pointer to the current valid version)      │
│  - Schema Registry & compatibility checks                            │
│  - Data-quality / validation gate before publish                     │
│  - Lineage (run_id, source_id, code_version, inputs → outputs)       │
└───────────────────────────┬────────────────────────────────────────┘
┌───────────────────────────┴────────────────────────────────────────┐
│  Write/Compute Layer                                                 │
│  - DatasetWriter (Python): DataFrame → validated → Parquet           │
│  - DuckDB as the write engine (COPY TO / INSERT INTO parquet)        │
└───────────────────────────┬────────────────────────────────────────┘
┌───────────────────────────┴────────────────────────────────────────┐
│  Storage Abstraction Layer                                           │
│  - StorageBackend interface: local fs / NFS / S3 (httpfs)            │
│  - URI scheme resolution: file://, s3://, nfs mount paths             │
└────────────────────────────────────────────────────────────────────┘
```

---

## 3. Storage Layout

Partition physically for pruning, and separate governance metadata from data files.

```
<root>/                                   # local dir, NFS mount, or s3://bucket/prefix
  datasets/
    <dataset_name>/                       # e.g. "sector_exposure"
      _catalog/
        schema/                           # versioned schema definitions (JSON)
          v1.json
          v2.json
        manifests/                        # one manifest per committed version
          manifest_0001.json
          manifest_0002.json
        CURRENT                           # pointer file -> latest manifest (atomic swap)
      data/
        stage=interim/
          run_date=2026-08-31/
            run_id=8f3a.../
              part-0000.parquet
              part-0001.parquet
        stage=results/
          run_date=2026-08-31/
            portfolio=PORT123/
              run_id=8f3a.../
                part-0000.parquet
        stage=published/
          as_of_date=2026-08-31/
            portfolio=PORT123/
              version=v0007/
                part-0000.parquet
```

**Data model: wide-in-memory, long-on-disk**

The in-memory pandas frame is typically wide — one column per component (e.g. an instrument identifier such as CUSIP, SEDOL, or ISIN, or an account/entity id) holding a value for a given date/period. On disk this gets **normalized to long/tidy format**: one row per `(component_id, component_type)` per period, with the relevant measures as columns. This is what makes the store SQL-governable — a wide, identifier-per-column frame isn't a stable schema (the column set changes every rebalance or refresh), whereas a long table has a fixed schema regardless of how many components a dataset covers.

Canonical long-format schema for a `results`/`published` dataset:

| column | type | notes |
|---|---|---|
| `as_of_date` | DATE | calculation/period date |
| `portfolio_id` | VARCHAR | fund or basket identifier |
| `benchmark_id` | VARCHAR | nullable — set when the row is benchmark-relative |
| `component_id` | VARCHAR | CUSIP (or SEDOL/ISIN/internal id) |
| `component_type` | VARCHAR | e.g. `security`, `sector`, `benchmark_constituent`, `composite` |
| `weight` | DOUBLE | |
| `benchmark_weight` | DOUBLE | nullable |
| `return` | DOUBLE | |
| `active_weight` | DOUBLE | derived, nullable |
| `allocation_effect` | DOUBLE | nullable — derived/decomposition stage only (example measure; rename or extend per use case) |
| `selection_effect` | DOUBLE | nullable |
| `interaction_effect` | DOUBLE | nullable |
| `contribution` | DOUBLE | nullable |
| `run_id` | VARCHAR | ties back to lineage |

`component_type` is a governed, small-cardinality **reference dimension** (register allowed values in the catalog, e.g. `security`, `sector`, `country`, `benchmark_constituent`, `composite_sleeve`) rather than a free-text field, so the DQ gate can reject unrecognized types and the UI can offer a reliable filter/dropdown.

**Partitioning guidance**
- Do **not** partition by `component_id`/CUSIP directly (high cardinality → too many tiny files). Partition by low-cardinality, query-frequent dimensions: `stage`, `as_of_date`, `portfolio_id`, and — since it's low-cardinality and commonly filtered on in the UI (e.g. "show me sector-level effects only") — `component_type` is a good partition candidate too.
- Sort each file `ORDER BY component_type, component_id` so DuckDB/Snowflake row-group pruning and predicate pushdown are effective on component filters even within a partition.
- Target file size ~128–512 MB per Parquet file for scan efficiency; split into `part-000N.parquet` files for large portfolios/baskets rather than one giant file per partition.
- Use dictionary encoding (Parquet default) for `component_id`, `component_type`, `portfolio_id` — effective for the repetitive, identifier-heavy nature of this data.
- Enforce uniqueness on `(as_of_date, portfolio_id, benchmark_id, component_id, component_type)` per version at the DQ gate — this is your primary key and the thing that guards against duplicate/double-counted holdings on rerun.

**Hierarchical membership (security → sector → asset class, N levels)**

Multi-level rollups need two separate structures — a **membership dimension** (governs what belongs to what, and when) and a **fact table extended to carry rows at every level**, not just the leaf. Keeping these separate matters because membership changes over time independent of any given calculation run (a stock's GICS sector gets reclassified; that shouldn't rewrite history for runs computed before the reclassification).

*1. `component_hierarchy_membership` — effective-dated dimension table*

| column | type | notes |
|---|---|---|
| `hierarchy_scheme_id` | VARCHAR | e.g. `GICS`, `CUSTOM_SLEEVE_V2` — you may run more than one scheme in parallel |
| `component_id` | VARCHAR | leaf-level id, e.g. CUSIP |
| `component_type` | VARCHAR | `security` |
| `level_1_id` / `level_1_name` | VARCHAR | e.g. asset class |
| `level_2_id` / `level_2_name` | VARCHAR | e.g. sector |
| `level_3_id` / `level_3_name` | VARCHAR | e.g. industry — nullable if the scheme only has 2 levels |
| ... | | extend flat columns up to the scheme's max depth |
| `effective_date` | DATE | membership valid from |
| `expiry_date` | DATE | nullable — null = currently active |

This is a **flattened, fixed-depth** model (levels as columns, not a generic parent/child adjacency list) — confirmed as the right fit here since depth is known and bounded (at least 4 levels: e.g. asset class → sector → industry → security). Register each scheme's depth and level names as metadata in the catalog (a `hierarchy_schemes` table: `scheme_id`, `level_number`, `level_name`) so depth is data-driven per scheme rather than hardcoded — provision the flat columns generously (e.g. `level_1_id` through `level_6_id`, left null past a given scheme's actual depth) so a deeper scheme later doesn't require a schema migration. The recursive adjacency-list fallback (`parent_component_id` + `WITH RECURSIVE`) isn't needed as the primary model given this constraint — worth keeping only as a documented escape hatch, not something to build now.

*2. Fact table (`financial_results`) extended for level-tagged rows*

Rather than storing only leaf-level values and rolling up at query time, hierarchical financial calculations typically **compute and persist derived measures at each level separately** — the decomposition math often differs level to level (this is the pattern behind attribution effects, risk contribution breakdowns, or exposure decomposition alike). Add to the canonical schema from above:

| column | type | notes |
|---|---|---|
| `hierarchy_scheme_id` | VARCHAR | which scheme this row's grouping belongs to |
| `level_number` | INTEGER | 0 = total portfolio, 1 = asset class, ... N = security |
| `level_name` | VARCHAR | denormalized from `hierarchy_schemes` for convenient filtering |
| `parent_component_id` | VARCHAR | nullable — id of the level above, for drill-down joins |

`component_id`/`component_type` at a given row now mean "the id/type **at this row's level**" — e.g. a `level_name='sector'` row's `component_id` is the sector id, not a security. This is what lets a single fact table hold the full hierarchy for a run: filter `level_name='security'` for leaf detail, `level_name='sector'` for a rolled-up sector view, both from the same table.

*Partitioning* — add `level_name` alongside `stage`/`as_of_date`/`portfolio_id` as a partition: it's low-cardinality and exactly what a UI drill-down view filters on first (top-level asset-class view vs. security-level detail are different partitions, not different scans of the same one). Sort each file `ORDER BY level_number, parent_component_id, component_id` so drill-down queries (parent → children) prune well.

*Governance check* — a natural DQ gate addition: for each parent row, the sum of its direct children's `contribution` (and any other additive derived measures, per the specific calculation's decomposition math) should tie out to the parent row's own value within tolerance. This catches rollup bugs at publish time rather than in the UI:

```sql
SELECT parent.component_id, parent.contribution AS parent_contrib,
       SUM(child.contribution) AS children_sum,
       parent.contribution - SUM(child.contribution) AS diff
FROM financial_results parent
JOIN financial_results child
  ON child.parent_component_id = parent.component_id
 AND child.as_of_date = parent.as_of_date
 AND child.run_id = parent.run_id
WHERE parent.level_name = 'sector' AND child.level_name = 'security'
GROUP BY parent.component_id, parent.contribution
HAVING ABS(parent.contribution - SUM(child.contribution)) > 1e-6;
```

---

## 4. Table-Format / Governance Layer

This is the piece that turns "a folder of Parquet files" into a governed dataset.

### 4.1 Manifest pattern (atomic publish without a database)

Each **commit** (write of a new version of a dataset+partition) produces:
1. New Parquet file(s) written to a **new, uniquely-named location** (never overwrite in place).
2. A **manifest** JSON listing exactly which files constitute that version, plus metadata (schema id, row count, checksum, created_by, run_id, created_at).
3. An **atomic pointer update**: write `CURRENT` (or `CURRENT.tmp` then rename/copy) to reference the new manifest.

This gives you:
- **Atomicity**: readers either see the old, fully-consistent version or the new one — never a half-written state.
- **Time travel**: keep old manifests around; point a version parameter at any historical manifest.
- **Cheap rollback**: repoint `CURRENT` to a prior manifest.

On S3, "atomic rename" doesn't exist, so use **conditional writes** (S3 now supports `If-None-Match` for put, or use a DynamoDB/Postgres row as the pointer if you need strict compare-and-swap) or accept last-writer-wins on `CURRENT` with manifest immutability guaranteeing readers never see corrupt data — only a slightly stale pointer in a race, which is acceptable for most batch calculation pipelines. On local/NFS, `os.rename` is atomic within the same filesystem and is sufficient.

> If your concurrency/versioning needs grow (many concurrent writers, need for MERGE/upsert semantics, time-travel SQL), swap this hand-rolled manifest for **Apache Iceberg** (DuckDB has an Iceberg extension, and Snowflake reads Iceberg tables natively via Iceberg REST/Glue catalog). The storage layout above is intentionally Iceberg-shaped so that migration is a lift, not a rewrite.

### 4.2 Dataset Catalog

A small, queryable metadata store — itself just a DuckDB database file (or Postgres if multiple writers need concurrent metadata transactions) holding:

```sql
CREATE TABLE job_executions (
    job_run_id           VARCHAR PRIMARY KEY,  -- same value used as `run_id` everywhere below
    job_type              VARCHAR,               -- e.g. 'sector_exposure_calc', 'risk_decomposition'
    parameters              JSON,                  -- full parameter payload: portfolio_id, period_start,
                                                     -- period_end, benchmark_id, hierarchy_scheme_id,
                                                     -- currency, methodology flags, other job-specific config
    parameter_hash           VARCHAR,               -- hash of normalized parameters — idempotency/dedup checks
    status                    VARCHAR,               -- queued | running | succeeded | failed
    triggered_by               VARCHAR,               -- user id or scheduler name
    code_version                 VARCHAR,               -- git sha of the calc engine
    started_at                    TIMESTAMP,
    ended_at                       TIMESTAMP,
    error_message                   VARCHAR              -- nullable
);

CREATE TABLE datasets (
    dataset_name        VARCHAR PRIMARY KEY,
    description          VARCHAR,
    owner                VARCHAR,
    stage_type           VARCHAR,      -- interim | results | published
    schema_version        INTEGER,
    created_at            TIMESTAMP
);

CREATE TABLE dataset_versions (
    dataset_name         VARCHAR,
    version_id            VARCHAR,      -- e.g. 'v0007'
    manifest_path         VARCHAR,
    run_id                 VARCHAR,     -- FK -> job_executions.job_run_id
    as_of_date             DATE,
    row_count              BIGINT,
    schema_version         INTEGER,
    checksum               VARCHAR,
    created_by              VARCHAR,
    created_at              TIMESTAMP,
    status                  VARCHAR      -- draft | validated | published | deprecated
);

CREATE TABLE dataset_lineage (
    dataset_name          VARCHAR,
    version_id             VARCHAR,
    input_dataset          VARCHAR,
    input_version           VARCHAR,
    code_version             VARCHAR,     -- git sha of calc engine
    run_id                    VARCHAR      -- FK -> job_executions.job_run_id
);

CREATE TABLE schema_registry (
    dataset_name           VARCHAR,
    schema_version           INTEGER,
    schema_json               VARCHAR,   -- column name/type/nullability
    compatible_with_prior      BOOLEAN,
    created_at                  TIMESTAMP
);
```

This catalog is small enough to query directly with DuckDB and can be replicated/exported for the UI backend or mirrored into Snowflake as a governance dashboard.

### 4.3 Schema governance

- Every write validates the DataFrame's dtypes against the registered schema for that dataset before it's allowed to land in `results` or `published`.
- Additive changes (new nullable column) → auto-approved, bump minor schema version.
- Breaking changes (type change, column removal, rename) → require explicit schema version bump + a migration/compat note; old readers pinned to old schema_version keep working against old manifests.
- Enforce with a `pyarrow.Schema` comparison, not just pandas dtypes, since Parquet's real type system is Arrow's.

### 4.4 Data quality gate

Before a version moves `interim → results → published`, run a validation suite (row count deltas vs prior run, null-rate thresholds on key columns, identifier format/checksum validation, sum-to-total tie-out checks for derived measures). Only a passing run gets a manifest promoted to `published` status in the catalog — this is your "governed" guarantee.

### 4.5 Linking datasets to job execution context

Every dataset version is produced by running a job with parameters — portfolio id, period (start/end date), benchmark, hierarchy scheme, and so on. The question of *where those parameters live* is best answered by splitting it, not picking one side:

**The job execution is its own governed entity, not metadata bolted onto the dataset.** `job_executions` (above) is the single source of truth for a run's full parameter payload, status, timing, and code version — recorded **once per run**, regardless of how many dataset versions that run produces. A single job execution commonly produces *several* dataset versions (e.g. `interim` intermediate output, `results` at multiple hierarchy levels, then a `published` version) — that's a one-to-many relationship, which is exactly why the parameters shouldn't live redundantly on every dataset row: they'd be duplicated across every output of the same run for no benefit.

**Only the parameters that define what a row *is about* get denormalized onto the dataset rows/partitions — everything else stays in `job_executions.parameters` only.** The dividing line: does a consumer (UI filter, analyst SQL, Snowflake query) need this value to filter or interpret a row *without joining back to the job table*? If yes, it's a first-class column/partition key on the dataset (this is already true of `portfolio_id`, `as_of_date`, `benchmark_id`, `hierarchy_scheme_id` in the schema above — they're not job bookkeeping, they're the row's identity). If the parameter is job-behavior configuration that doesn't change how a row should be read (retry policy, source-system snapshot id, a methodology toggle that's already reflected in the computed values themselves, batch sizing) — it stays only in `job_executions.parameters`, retrievable by joining on `run_id` when someone needs full reproducibility context.

**`run_id` is the required link, present as a column on every dataset row and in every manifest/version record — but it's a foreign key, not a copy of the payload.** This is already in the canonical schema (Section 3) and the manifest (Section 4.1); it's what lets you answer "what job, with what full parameters, produced this exact row" via one join, without bloating every row with the entire parameter set:

```sql
SELECT f.*, j.parameters, j.code_version, j.started_at
FROM financial_results f
JOIN job_executions j ON f.run_id = j.job_run_id
WHERE f.portfolio_id = 'PORT123' AND f.as_of_date = '2026-08-31';
```

**Practical implications for the writer:**
- `DatasetWriter.write()` should require (or look up) a `job_run_id` and, if the `job_executions` row doesn't already exist for it, refuse to write — a dataset version should never exist without a corresponding job record.
- `parameter_hash` on `job_executions` supports idempotency: before launching a calc, check whether a `succeeded` run already exists for the same `(job_type, parameter_hash, code_version)` and skip/reuse rather than recompute.
- Partition values (`portfolio_id`, `as_of_date`, etc.) passed to `write()` should be derived from — and validated against — the same job's `parameters`, so the partition path and the job record never disagree about what was actually run.

---

## 5. Storage Abstraction Layer

Keep exactly one seam where "where are the bytes" is decided, so everything above it is storage-agnostic.

```python
from dataclasses import dataclass
from urllib.parse import urlparse
import duckdb

@dataclass
class StorageConfig:
    root_uri: str            # "file:///data/attrib", "s3://my-bucket/attrib", "/mnt/nfs/attrib"
    s3_region: str | None = None
    s3_endpoint: str | None = None   # for S3-compatible stores (MinIO, etc.)
    s3_profile: str | None = None

class StorageBackend:
    """Thin wrapper: resolves paths and configures the DuckDB connection
    with the right extensions/secrets for the target backend."""

    def __init__(self, config: StorageConfig):
        self.config = config
        self._scheme = urlparse(config.root_uri).scheme or "file"

    def connect(self) -> duckdb.DuckDBPyConnection:
        con = duckdb.connect()
        con.execute("INSTALL httpfs; LOAD httpfs;")
        if self._scheme == "s3":
            if self.config.s3_endpoint:
                con.execute(f"SET s3_endpoint='{self.config.s3_endpoint}';")
            if self.config.s3_region:
                con.execute(f"SET s3_region='{self.config.s3_region}';")
            # Prefer DuckDB's native CREATE SECRET (supports credential chain,
            # STS, profile-based auth) over hardcoding keys.
            con.execute(f"""
                CREATE OR REPLACE SECRET s3_secret (
                    TYPE S3,
                    PROVIDER CREDENTIAL_CHAIN
                    {f", PROFILE '{self.config.s3_profile}'" if self.config.s3_profile else ''}
                );
            """)
        return con

    def resolve(self, *parts: str) -> str:
        base = self.config.root_uri.rstrip("/")
        return "/".join([base, *parts])
```

- **Local / NFS**: `root_uri` is just a filesystem path; DuckDB reads/writes Parquet natively, no extension needed. NFS behaves like local disk from DuckDB's point of view — just be mindful of NFS locking semantics for the `CURRENT` pointer file (prefer write-new-file + rename within the same directory, which NFSv4 handles atomically; avoid NFSv3 if you can).
- **S3**: use the `httpfs` extension; authenticate via DuckDB's `CREATE SECRET` (credential chain, so it picks up IAM role / env vars / profile — no hardcoded keys in app code).
- Adding a new backend (Azure ADLS, GCS) later is just another branch in `connect()` using DuckDB's `azure`/`httpfs` support — nothing above this layer changes.

---

## 6. Write & Read API

### 6.1 DatasetWriter

```python
import pandas as pd
import pyarrow as pa
import json, hashlib, uuid
from datetime import datetime, timezone

class DatasetWriter:
    def __init__(self, storage: StorageBackend, catalog):
        self.storage = storage
        self.catalog = catalog

    def write(
        self,
        dataset_name: str,
        df: pd.DataFrame,
        stage: str,               # interim | results | published
        run_id: str,              # must reference an existing job_executions.job_run_id
        partition_values: dict,   # e.g. {"portfolio": "PORT123", "as_of_date": "2026-08-31"}
        code_version: str,
        inputs: list[tuple[str, str]] | None = None,  # [(dataset_name, version_id), ...]
    ) -> str:
        # A dataset version must never exist without a corresponding job record —
        # this is what makes `run_id` a real foreign key rather than a free-form tag.
        job = self.catalog.get_job_execution(run_id)
        if job is None:
            raise ValueError(f"No job_executions record for run_id={run_id}; register the job before writing.")
        self.catalog.validate_partition_values_against_job(partition_values, job["parameters"])

        schema = self.catalog.get_current_schema(dataset_name)
        table = pa.Table.from_pandas(df, schema=schema, preserve_index=False)
        self.catalog.validate_schema(dataset_name, table.schema)

        version_id = f"v{self.catalog.next_version(dataset_name):04d}"
        part_dir = self.storage.resolve(
            "datasets", dataset_name, "data", f"stage={stage}",
            *[f"{k}={v}" for k, v in partition_values.items()],
            f"run_id={run_id}",
        )
        file_path = f"{part_dir}/part-0000.parquet"

        con = self.storage.connect()
        con.register("df_view", table)
        con.execute(f"""
            COPY (SELECT * FROM df_view ORDER BY cusip)
            TO '{file_path}' (FORMAT PARQUET, COMPRESSION ZSTD);
        """)

        checksum = hashlib.sha256(df.to_csv(index=False).encode()).hexdigest()[:16]
        manifest = {
            "dataset_name": dataset_name,
            "version_id": version_id,
            "files": [file_path],
            "row_count": len(df),
            "schema_version": self.catalog.current_schema_version(dataset_name),
            "checksum": checksum,
            "run_id": run_id,
            "created_at": datetime.now(timezone.utc).isoformat(),
            "partition_values": partition_values,
        }
        manifest_path = self.storage.resolve(
            "datasets", dataset_name, "_catalog", "manifests", f"manifest_{version_id}.json"
        )
        self._write_json_atomic(manifest_path, manifest)
        self._update_current_pointer(dataset_name, manifest_path)

        self.catalog.record_version(dataset_name, version_id, manifest_path, run_id,
                                     status="draft", partition_values=partition_values)
        if inputs:
            self.catalog.record_lineage(dataset_name, version_id, inputs, code_version, run_id)
        return version_id
```

**Normalization step (wide → long)** — belongs in `DatasetWriter.write()` before schema validation, since upstream calc code will often still produce wide frames (one column per component):

```python
def normalize_wide_to_long(
    df_wide: pd.DataFrame,
    id_cols: list[str],          # e.g. ["as_of_date", "portfolio_id", "benchmark_id"]
    component_type: str,         # e.g. "security" — fixed for a given wide frame
    measure_name: str,           # e.g. "weight", "return", "allocation_effect"
) -> pd.DataFrame:
    long_df = df_wide.melt(
        id_vars=id_cols, var_name="component_id", value_name=measure_name
    )
    long_df["component_type"] = component_type
    return long_df
```

In practice a calc run produces several wide frames (weights, returns, effects) that share the same `id_cols` + `component_id` grain — melt each, then `merge`/`concat` them into the single long table matching the canonical schema above before calling `write()`. Keeping this as an explicit, testable function (rather than folding it into `write()` implicitly) makes it easy to validate the melt against known totals — e.g. sum of `weight` per `(as_of_date, portfolio_id)` should tie back to the wide frame's row-level total.

Key points:
- Sort by `component_type, component_id` at write time for pruning benefit on read.
- `ZSTD` compression — good ratio/speed tradeoff for wide financial data; DuckDB and Snowflake both read it natively.
- Files are **immutable**; every write is a new file + new manifest, never an in-place overwrite. This is what makes concurrent readers safe without locks.
- Promotion from `draft → validated → published` happens as a **catalog status update** after the DQ gate passes, not as a data rewrite.

### 6.2 DatasetReader

```python
class DatasetReader:
    def __init__(self, storage: StorageBackend, catalog):
        self.storage = storage
        self.catalog = catalog

    def read(
        self,
        dataset_name: str,
        version_id: str | None = None,   # None = latest published
        columns: list[str] | None = None,
        filters: str | None = None,      # raw SQL WHERE clause, e.g. "cusip IN (...)"
    ) -> pd.DataFrame:
        manifest = self.catalog.get_manifest(dataset_name, version_id)
        files = manifest["files"]
        con = self.storage.connect()
        col_expr = ", ".join(columns) if columns else "*"
        where_clause = f"WHERE {filters}" if filters else ""
        file_list = ", ".join(f"'{f}'" for f in files)
        query = f"""
            SELECT {col_expr}
            FROM read_parquet([{file_list}])
            {where_clause}
        """
        return con.execute(query).fetch_df()

    def sql(self, query: str) -> pd.DataFrame:
        """Arbitrary SQL over registered dataset views (see section 7)."""
        con = self.storage.connect()
        return con.execute(query).fetch_df()
```

For UI-facing querying, register **views per dataset** pointing at the current manifest's files (see below) so app developers write plain SQL against a stable table name instead of manifest plumbing.

---

## 7. Access Layer for UI & Analysts

Generate a DuckDB view layer on top of the catalog so consumers never touch file paths:

```python
def refresh_views(con, catalog, storage):
    for dataset_name in catalog.list_published_datasets():
        manifest = catalog.get_manifest(dataset_name, version_id=None)  # latest published
        files = ", ".join(f"'{f}'" for f in manifest["files"])
        con.execute(f"""
            CREATE OR REPLACE VIEW {dataset_name} AS
            SELECT * FROM read_parquet([{files}]);
        """)
```

- UI backend (e.g. a small FastAPI service) holds a long-lived DuckDB connection, refreshes views on a schedule or on catalog-change notification, and exposes parameterized SQL/REST endpoints. DuckDB's low startup cost and in-process model make it a good fit for this — no separate query cluster needed for moderate concurrency.
- For heavier UI concurrency, consider DuckDB's read-only mode against the same files from multiple processes (safe, since files are immutable), or front it with a small connection pool.
- Ad hoc analysts get a `.duckdbrc`/attach script that runs the same `refresh_views` against the catalog — same SQL surface as the UI.

---

## 8. Snowflake Integration

Two options, in increasing order of integration depth:

1. **External Table over the same S3 Parquet** (fastest to stand up, zero duplication):
   ```sql
   CREATE OR REPLACE STAGE attrib_stage
     URL = 's3://my-bucket/analytics/datasets/sector_exposure/data/stage=published/'
     STORAGE_INTEGRATION = my_s3_integration;

   CREATE OR REPLACE EXTERNAL TABLE sector_exposure_ext
     WITH LOCATION = @attrib_stage
     AUTO_REFRESH = TRUE
     FILE_FORMAT = (TYPE = PARQUET);
   ```
   Snowflake reads the exact same governed Parquet files your DuckDB layer writes. Point Snowflake at the `published` stage specifically (never `interim`) so Snowflake only ever sees governed, validated data.

2. **Native ingestion (`COPY INTO`)** when you want Snowflake-managed tables (clustering, micro-partitions, faster repeated query performance) rather than external-table scan-per-query:
   ```sql
   COPY INTO sector_exposure_native
   FROM @attrib_stage
   FILE_FORMAT = (TYPE = PARQUET)
   MATCH_BY_COLUMN_NAME = CASE_INSENSITIVE;
   ```
   Trigger this from your pipeline right after a version is promoted to `published` in the catalog (webhook/Snowflake task polling the catalog's `dataset_versions` table, or just call `COPY INTO` as the last step of the publish workflow).

If you outgrow the hand-rolled manifest and move to **Iceberg** (Section 4.1 note), Snowflake can read your Iceberg tables directly as **Iceberg external tables/catalog integration**, and DuckDB reads the same Iceberg metadata — giving you one physical copy of data queried natively (not just via external-table scan) by both engines, with real time-travel and schema evolution. Worth planning the storage layout (already partition-friendly above) with this migration in mind even if you start with the simpler manifest.

---

## 9. Governance Summary — What "Governed" Means Concretely

| Concern | Mechanism |
|---|---|
| Immutability / audit trail | Files never overwritten; every version is a new file + manifest |
| Atomic publish | Manifest + `CURRENT` pointer swap; readers never see partial writes |
| Schema control | Schema registry with compatibility checks before write is accepted |
| Data quality | Validation gate between `results` and `published` status |
| Lineage | `dataset_lineage` table: run_id, code_version, input dataset+version |
| Job/parameter linkage | `job_executions` is the source of truth for full run parameters; dataset rows carry only the identity-defining subset (e.g. `portfolio_id`, `as_of_date`) plus `run_id` as the join key — no full-parameter duplication per row |
| Discoverability | Central `datasets` / `dataset_versions` catalog, queryable in SQL |
| Access control | Bucket/path-level IAM (S3) or filesystem ACLs (NFS); optionally row/column-level views for UI |
| Reproducibility | Any historical version re-readable via `version_id`; pin `code_version` in lineage |

---

## 10. Suggested Build Phases

1. **Phase 1 — Core plumbing**: StorageBackend, DatasetWriter/Reader without manifests (simple `latest/` symlink-style pointer), local + S3 support, ZSTD Parquet, CUSIP sort-on-write.
2. **Phase 2 — Governance**: manifest pattern, dataset catalog (DuckDB-backed), schema registry, DQ gate before `published`.
3. **Phase 3 — Access layer**: view-refresh service, FastAPI query endpoint for UI, lineage tracking.
4. **Phase 4 — Snowflake wiring**: external tables on `published` stage, optional `COPY INTO` automation triggered off catalog publish events.
5. **Phase 5 — Optional table-format upgrade**: migrate manifest layer to Apache Iceberg if concurrency/time-travel/merge needs grow beyond what the hand-rolled manifest comfortably handles.

---

## 11. Key Design Decisions to Confirm

- **Metadata store for the catalog**: DuckDB file (simple, single-writer-friendly) vs. Postgres (if multiple pipeline processes write concurrently and you need real row-level locking on version increments).
- **Conditional-write strategy on S3** for the `CURRENT` pointer if you have multiple concurrent publishers to the same dataset.
- **Retention policy** for `interim` stage files (these can grow fast — consider TTL/lifecycle rules on the object store).
- **Row-level/column-level security** if different UI users should see different portfolios — best enforced as parameterized views rather than raw file access.
