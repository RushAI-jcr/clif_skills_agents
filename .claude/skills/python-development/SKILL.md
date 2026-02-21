---
name: python-development
language: python
description: Modern Python development practices emphasizing Polars expressions, DuckDB integration, uv project management, type hints, and performance patterns. Use when Claude needs to write Python code for CLIF data analysis or general Python guidance.
---

# Python Development

This skill provides comprehensive guidance for modern Python development in CLIF projects, emphasizing Polars idioms, DuckDB integration, and uv-based project management.

## Core Principles

1. **Use Polars expressions, not pandas habits** - Write expression-based code; never use `.apply()` or row-wise iteration
2. **Lazy by default** - Use `scan_parquet` / `scan_csv` and `.collect()` only when needed
3. **uv for everything** - Package management, script running, virtual environments
4. **Type hints on all function signatures** - Use modern Python 3.10+ syntax (`X | None`, not `Optional[X]`)

## Polars Essentials

### Lazy Frames (Always Prefer)

```python
import polars as pl

# Lazy: query optimizer pushes filters and projections down
lf = (
    pl.scan_parquet("labs.parquet")
    .filter(pl.col("lab_category") == "lactate")
    .select("encounter_id", "lab_value_numeric", "lab_collect_dttm")
)
result = lf.collect()

# AVOID: eager read loads everything into memory first
df = pl.read_parquet("labs.parquet")  # only when dataset is small
```

### Expressions over Apply (CRITICAL)

Polars expressions run in parallel on Rust. `.apply()` / `.map_elements()` drop to single-threaded Python — never use them.

```python
# GOOD: native expressions, parallel execution
df = df.with_columns(
    (pl.col("lab_value_numeric") * 10).alias("scaled"),
    pl.col("lab_collect_dttm").dt.hour().alias("hour"),
    pl.when(pl.col("vital_category") == "sbp")
    .then(pl.lit("systolic"))
    .otherwise(pl.lit("other"))
    .alias("bp_type"),
)

# BAD: Python UDF, single-threaded, no optimization
df = df.with_columns(
    pl.col("lab_value_numeric").map_elements(lambda x: x * 10).alias("scaled")
)
```

### Contexts: select, with_columns, filter, group_by

```python
# select: choose/compute columns (drops others)
df.select("encounter_id", pl.col("lab_value_numeric").mean().alias("mean_lab"))

# with_columns: add/overwrite columns (keeps all others)
df.with_columns(pl.col("vital_value").cast(pl.Float64))

# filter: row-level predicates
df.filter(
    (pl.col("lab_category") == "creatinine")
    & (pl.col("lab_value_numeric") > 1.2)
)

# group_by + agg: aggregations
df.group_by("encounter_id").agg(
    pl.col("lab_value_numeric").mean().alias("mean_val"),
    pl.col("lab_value_numeric").count().alias("n_labs"),
    pl.col("lab_collect_dttm").min().alias("first_lab"),
)
```

### Joins

```python
# Standard join
labs.join(encounters, on="encounter_id", how="inner")

# Multi-key join
meds.join(encounters, on=["encounter_id", "hospitalization_id"], how="left")

# Asof join (nearest timestamp match) — great for time-series alignment
vitals.sort("vital_dttm").join_asof(
    labs.sort("lab_collect_dttm"),
    left_on="vital_dttm",
    right_on="lab_collect_dttm",
    by="encounter_id",
    strategy="backward",  # most recent lab before vital
)
```

### Window Functions

```python
# Rank within groups
df.with_columns(
    pl.col("lab_value_numeric")
    .rank(method="ordinal")
    .over("encounter_id")
    .alias("lab_rank")
)

# Running calculations
df.sort("lab_collect_dttm").with_columns(
    pl.col("lab_value_numeric")
    .rolling_mean(window_size=3)
    .over("encounter_id")
    .alias("rolling_avg")
)
```

### Common Anti-Patterns

```python
# BAD: iterating rows
for row in df.iter_rows(named=True):
    process(row)

# GOOD: express the logic as columns
df.with_columns(...)

# BAD: chaining separate with_columns (kills parallelism)
df = df.with_columns(a=...)
df = df.with_columns(b=...)

# GOOD: single with_columns block
df = df.with_columns(
    a=...,
    b=...,
)

# BAD: converting to pandas for a quick operation
result = df.to_pandas().groupby("x").mean()

# GOOD: stay in Polars
result = df.group_by("x").agg(pl.all().mean())
```

## DuckDB Integration

Use DuckDB for complex SQL queries on Parquet files or when SQL is clearer than expression chains.

```python
import duckdb

con = duckdb.connect()

# Query Parquet directly (no loading step)
result = con.sql("""
    SELECT encounter_id, COUNT(*) AS n_labs
    FROM read_parquet('data/clif/labs.parquet')
    WHERE lab_category = 'lactate'
    GROUP BY encounter_id
""").pl()  # .pl() returns Polars DataFrame

# Register an existing Polars DataFrame for SQL queries
con.register("vitals_df", vitals)
result = con.sql("""
    SELECT encounter_id, AVG(vital_value) AS mean_sbp
    FROM vitals_df
    WHERE vital_category = 'sbp'
    GROUP BY encounter_id
""").pl()

# Write query results to Parquet
con.sql("SELECT * FROM read_parquet('big_table.parquet') WHERE ...").to_parquet("filtered.parquet")
```

### When to Use DuckDB vs Polars

| Use DuckDB | Use Polars |
|---|---|
| Complex multi-table joins with CTEs | Simple transforms and aggregations |
| Ad-hoc SQL exploration | Programmatic pipelines |
| Files too large for memory | Data fits in memory |
| Team prefers SQL syntax | Expression-based workflows |

## uv Project Management

### Project Lifecycle

```bash
# Initialize a new project
uv init my-clif-project
cd my-clif-project

# Add dependencies
uv add polars duckdb clifpy marimo pyarrow

# Add dev dependencies
uv add --dev pytest ruff

# Lock dependencies (reproducible)
uv lock

# Sync environment from lockfile
uv sync

# Run a script/command in the project environment
uv run python my_analysis.py
uv run marimo edit notebook.py
uv run pytest
```

### Inline Script Dependencies (PEP 723)

For standalone scripts that declare their own dependencies:

```python
# /// script
# requires-python = ">=3.11"
# dependencies = [
#   "polars",
#   "duckdb",
#   "clifpy",
# ]
# ///

import polars as pl
from clifpy import ClifOrchestrator

clif = ClifOrchestrator("clif_config.json")
labs = clif.labs.df.collect()
```

```bash
# Run it — uv installs deps automatically
uv run my_script.py
```

### One-Off Tool Execution

```bash
# Run a CLI tool without installing it globally
uvx marimo edit notebook.py
uvx ruff check .
```

## Type Hints (Python 3.10+)

Use modern union syntax and built-in generics:

```python
# Modern (3.10+)
def process_labs(
    encounter_id: str,
    categories: list[str],
    threshold: float | None = None,
) -> pl.DataFrame:
    ...

# AVOID legacy typing imports
from typing import Optional, List  # don't use these
```

### Common patterns for CLIF code:

```python
from pathlib import Path
import polars as pl

def load_and_filter(
    path: Path,
    category: str,
    min_value: float | None = None,
) -> pl.LazyFrame:
    lf = pl.scan_parquet(path).filter(pl.col("lab_category") == category)
    if min_value is not None:
        lf = lf.filter(pl.col("lab_value_numeric") >= min_value)
    return lf
```

## Performance Patterns

### Profile Before Optimizing

```python
# Time operations
import time
start = time.perf_counter()
result = expensive_query.collect()
print(f"Elapsed: {time.perf_counter() - start:.2f}s")

# Inspect query plan
lazy_query.explain()       # optimized plan
lazy_query.explain(optimized=False)  # naive plan
```

### Key Strategies

1. **Stay lazy**: `scan_parquet` → chain transforms → `.collect()` once at the end
2. **Predicate pushdown**: filter early so Polars reads fewer rows from Parquet
3. **Projection pushdown**: select only needed columns so Polars reads fewer columns
4. **Streaming**: for datasets larger than RAM, use `.collect(streaming=True)`
5. **Sink to disk**: `lf.sink_parquet("output.parquet")` to avoid materializing in memory
6. **Avoid Python UDFs**: every `.map_elements()` is a performance cliff

### Memory Efficiency

```python
# Stream large files without loading all into memory
lf = pl.scan_parquet("huge_file.parquet")
lf.filter(...).select(...).sink_parquet("result.parquet")

# Use categorical dtype for repeated strings
df = df.with_columns(pl.col("lab_category").cast(pl.Categorical))
```

## Project Structure Conventions

```
my-clif-project/
  pyproject.toml          # uv project config + dependencies
  uv.lock                 # pinned dependency versions
  clif_config.json        # CLIF site configuration
  src/                    # shared modules (optional)
    utils.py
  notebooks/              # marimo or Jupyter notebooks
    01_cohort.py
    02_analysis.py
  tests/                  # pytest tests
  .claude/
    claude-todo.md
    claude-progress.md
    lessons.md
```
