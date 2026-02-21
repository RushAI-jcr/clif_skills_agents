---
name: marimo-notebook
language: python
description: marimo notebook development — script, interactive, and run modes with reactive DAG-based cells. Use when creating or editing marimo notebooks.
---

# marimo Notebook Development

## Modes

- **Script mode**: `uv run <notebook.py>` — non-interactive execution
- **Interactive mode**: `uv run marimo edit <notebook.py>` — browser-based editing
- **Run mode**: `uv run marimo run <notebook.py>` — interactive execution

## Design Philosophy

Show all UI elements always. Only change the data source in script mode. Detect mode with `mo.app_meta().mode == "script"`.

## Critical Rules

- **Don't guard cells with `if` statements.** Marimo's reactivity manages cell execution automatically. Wrapping code in conditionals prevents output rendering.
- **Don't use try/except for flow control.** Only for anticipated, specific error types with meaningful recovery.
- **Don't indent expressions expecting them to render.** Marimo renders only the final expression in a cell.

## Technical Practices

- Use `pathlib.Path` instead of `os.path` for file operations
- Prefix loop variables with underscores (`_name`) to prevent cell-scope conflicts
- Include PEP 723 dependency declarations in notebook headers
- Validate notebooks with `uvx marimo check <notebook.py>` before delivery
- For API help: `uv --with marimo run python -c 'import marimo as mo; help(mo.ui.form)'`
