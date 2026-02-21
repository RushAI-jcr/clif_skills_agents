# Top-Level Imports in Marimo

Marimo allows importing top-level functions and classes from notebooks into other Python scripts using standard import syntax.

## Requirements for Exportable Definitions

1. **Single definition per cell**: The cell must define just a single function or class.
2. **Symbol scope**: The defined function or class can only refer to symbols defined in the setup cell, or to other top-level symbols.

## Implementation

Use `@app.function` decorator for exportable code. Define setup dependencies using `app.setup` as a context manager.

```python
from my_notebook import calculate_statistics
```

This makes notebook code maintainable in text editors while preserving interactivity within marimo.
