# SQL in Marimo

A SQL cell is just a function call to `marimo.sql`, which returns results as polars dataframes by default.

## Usage

```python
result = mo.sql("SELECT * FROM my_table")
```

The `mo.sql()` function accepts a query string and an optional database engine parameter. By default, marimo uses DuckDB in memory and can reference dataframe variables within scope.

## Supported Engines

**DuckDB** (default): In-memory or file-based connections:
```python
duckdb_conn = duckdb.connect("file.db", read_only=True)
```

**SQLAlchemy**: For various database backends:
```python
sqlite_engine = sqlalchemy.create_engine("sqlite:///:memory:")
```

**PyIceberg**: Supports data catalogs for REST-based enterprise data management.

## Output Configuration

- `output=False` to suppress cell rendering and return results as variables
- Configure pandas as an alternative to polars for dataframe output
