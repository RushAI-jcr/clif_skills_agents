# SQL in Marimo

`mo.sql()` executes SQL queries via DuckDB and returns results as Polars DataFrames. DuckDB can query in-scope Python DataFrames by variable name — no import or registration needed.

## Basic Usage

```python
result = mo.sql("SELECT * FROM my_table LIMIT 10")
```

- Uses DuckDB in-memory by default
- Returns a Polars DataFrame (configurable to Pandas in app settings)
- Use `output=False` to suppress cell rendering and return results as variables only
- Prefix output variable with `_` to make it private (not referenceable downstream)

## CLIF-Specific DuckDB Example

Load parquet files, filter by lab_category, and join to hospitalization:

```python
import marimo as mo
import polars as pl

# Load CLIF parquet files into Polars DataFrames
labs = pl.read_parquet("clif_labs.parquet")
hospitalization = pl.read_parquet("clif_hospitalization.parquet")
```

```python
# DuckDB queries the Polars DataFrames directly by variable name
first_24h_labs = mo.sql(
    f"""
    SELECT
        l.hospitalization_id,
        l.lab_category,
        MIN(l.lab_value_numeric) AS min_val,
        MAX(l.lab_value_numeric) AS max_val,
        AVG(l.lab_value_numeric) AS mean_val,
        COUNT(*) AS n_measurements
    FROM labs l
    JOIN hospitalization h ON l.hospitalization_id = h.hospitalization_id
    WHERE l.lab_category IN ('creatinine', 'lactate', 'bilirubin_total')
      AND l.lab_result_dttm BETWEEN h.admission_dttm
          AND h.admission_dttm + INTERVAL '24 hours'
      AND l.lab_value_numeric IS NOT NULL
    GROUP BY l.hospitalization_id, l.lab_category
    ORDER BY l.hospitalization_id
    """
)
```

## Parameterized SQL with UI Elements

UI values flow into SQL via f-string interpolation. The SQL cell re-runs reactively when the UI value changes.

```python
import marimo as mo

lab_filter = mo.ui.dropdown(
    options=["creatinine", "lactate", "hemoglobin", "platelet_count", "bilirubin_total"],
    value="creatinine",
    label="Lab category"
)
date_range = mo.ui.date_range(
    start="2023-01-01",
    stop="2024-12-31",
    value=("2024-01-01", "2024-06-30"),
    label="Date range"
)
mo.hstack([lab_filter, date_range], gap=1.0)
```

```python
# This cell re-executes whenever lab_filter or date_range changes
start_date, end_date = date_range.value
filtered_labs = mo.sql(
    f"""
    SELECT
        hospitalization_id,
        lab_collect_dttm,
        lab_value_numeric,
        reference_unit
    FROM labs
    WHERE lab_category = '{lab_filter.value}'
      AND lab_collect_dttm BETWEEN '{start_date}' AND '{end_date}'
      AND lab_value_numeric IS NOT NULL
    ORDER BY lab_collect_dttm
    """
)
```

```python
# Display results in an interactive table
mo.ui.table(filtered_labs, selection="multi", page_size=20, label="Filtered labs")
```

## DataFrame-to-SQL Bridge

Any Polars or Pandas DataFrame in scope is queryable by its Python variable name — DuckDB auto-discovers it.

```python
import polars as pl

# Create a DataFrame in Python
cohort = pl.DataFrame({
    "hospitalization_id": ["H001", "H002", "H003"],
    "site": ["SITE_A", "SITE_A", "SITE_B"],
    "age": [65, 42, 78],
})
```

```python
# Query it with SQL as if it were a table
elderly = mo.sql(
    f"""
    SELECT * FROM cohort
    WHERE age >= 65
    ORDER BY age DESC
    """
)
```

```python
# Join a SQL result back to a Python DataFrame
combined = mo.sql(
    f"""
    SELECT c.hospitalization_id, c.site, c.age,
           l.lab_category, l.lab_value_numeric
    FROM cohort c
    JOIN labs l ON c.hospitalization_id = l.hospitalization_id
    WHERE l.lab_category = 'creatinine'
    """
)
```

## Custom DuckDB Connection

```python
import duckdb

# File-based DuckDB for persistent data
conn = duckdb.connect("clif_warehouse.duckdb")

# marimo auto-discovers the connection in the SQL cell dropdown
results = mo.sql("SELECT * FROM persistent_table LIMIT 100")
```

## Supported Engines

- **DuckDB** (default): In-memory or file-based
- **SQLAlchemy**: `sqlalchemy.create_engine("sqlite:///:memory:")`
- **PyIceberg**: REST-based enterprise data catalogs
