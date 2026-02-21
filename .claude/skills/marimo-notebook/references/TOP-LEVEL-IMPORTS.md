# Top-Level Imports in Marimo

Marimo notebooks can export functions and classes for use in other Python scripts via standard `import` syntax.

## Requirements

1. **Single definition per cell**: The cell must define exactly one function or class
2. **Symbol scope**: The function/class can only reference symbols from the `app.setup` block or other `@app.function` definitions
3. **Python format only**: Markdown-format notebooks (`.md`) do not support this feature

## app.setup — Shared State

The `with app.setup:` block defines imports and constants available to all `@app.function` cells. It runs before all other cells.

```python
import marimo

__generated_with = "0.10.0"
app = marimo.App()

with app.setup:
    import polars as pl
    import numpy as np
    from datetime import date, datetime
    from pathlib import Path

    OUTLIER_THRESHOLDS = {
        "heart_rate": (0, 300),
        "sbp": (0, 300),
        "temp_c": (32, 44),
        "spo2": (50, 100),
    }
```

## @app.function — Exportable Definitions

Each `@app.function` cell contains exactly one function or class. These are importable from other scripts.

```python
@app.function
def load_clif_table(data_dir: str, table_name: str) -> pl.DataFrame:
    """Load a CLIF parquet table with basic validation."""
    path = Path(data_dir) / f"clif_{table_name}.parquet"
    if not path.exists():
        raise FileNotFoundError(f"Missing: {path}")
    return pl.read_parquet(path)
```

```python
@app.function
def apply_outlier_filter(
    df: pl.DataFrame,
    category_col: str,
    value_col: str,
    category: str,
) -> pl.DataFrame:
    """Filter rows outside CLIF outlier thresholds."""
    if category not in OUTLIER_THRESHOLDS:
        return df.filter(pl.col(category_col) == category)
    low, high = OUTLIER_THRESHOLDS[category]
    return df.filter(
        (pl.col(category_col) == category)
        & pl.col(value_col).is_between(low, high)
    )
```

## Importing in Another Script

```python
# In analysis.py — standard Python import syntax
from my_clif_notebook import load_clif_table, apply_outlier_filter

labs = load_clif_table("/data/clif", "labs")
creatinine = apply_outlier_filter(labs, "lab_category", "lab_value_numeric", "creatinine")
print(f"Creatinine measurements: {creatinine.shape[0]:,}")
```

## Import Ordering Conventions for CLIF Notebooks

```python
with app.setup:
    # 1. Standard library
    import json
    from datetime import date, datetime
    from pathlib import Path

    # 2. Third-party
    import polars as pl
    import duckdb
    import marimo as mo

    # 3. CLIF-specific
    from clifpy import ClifOrchestrator

    # 4. Project constants
    DATA_DIR = "/path/to/clif/data"
    SITE = "SITE_A"
```

## Constraints

- Cyclic dependencies between `@app.function` definitions are prohibited
- Local variables (underscore-prefixed `_var`) cannot be exported
- Regular cell-local variables cannot be referenced by `@app.function` cells
- Only symbols in `app.setup` and other `@app.function` definitions are in scope
