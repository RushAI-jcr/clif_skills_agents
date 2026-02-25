---
name: marimo-pro
language: python
description: Advanced marimo patterns — state management, caching, layouts, top-level exports, WASM deployment, batch execution, anywidgets, and performance optimization. Use for complex marimo notebooks beyond basic cell patterns.
---

# Advanced marimo Patterns

## State Management (`mo.state`)

`get_val, set_val = mo.state(initial_value)` — setter takes a function, never a raw value.

```python
get_count, set_count = mo.state(0)
increment = mo.ui.button(label="Add", on_change=lambda _: set_count(lambda v: v + 1))
mo.md(f"Count: {get_count()}")
```

Rules: never mutate state directly; never store `mo.ui` elements in state. Use to synchronize multiple UI elements.

## Conditional Execution (`mo.stop`)

Halts cell and all downstream dependents when condition is `True`.

```python
mo.stop(not file_upload.value, mo.md("Upload a file to continue"))
df = pl.read_csv(file_upload.value[0].contents)
```

## Caching

| Decorator | Scope | Survives restart |
|---|---|---|
| `@mo.cache` | In-memory, per session | No |
| `@mo.persistent_cache` | Disk-backed | Yes |

```python
@mo.cache
def compute_embeddings(text: str, model: str) -> np.ndarray:
    ...

@mo.persistent_cache
def load_large_dataset(path: str) -> pl.DataFrame:
    ...
```

Both work as decorators or context managers.

## Layouts

```python
mo.hstack([a, b], widths=[1, 3])        # horizontal, weighted widths
mo.vstack([a, b], align="stretch")      # vertical
mo.accordion({"Section": content})      # collapsible
mo.callout(content, kind="warn")        # info | warn | danger | success
mo.sidebar([nav, filters])              # persistent sidebar
mo.nav_menu({"Page": "/path"})          # navigation
mo.ui.tabs({"Tab A": a, "Tab B": b})    # tabbed content
```

## Forms and Batched Input

Prevents re-computation on every keystroke. Submit fires once on button click.

```python
config_form = mo.ui.form(
    mo.md('''
    **Analysis Config**
    - Category: {category}
    - Min value: {min_val}
    ''').batch(
        category=mo.ui.dropdown(["lactate", "creatinine"]),
        min_val=mo.ui.number(value=0, start=0, stop=100),
    )
)
# Access submitted values: config_form.value["category"]
```

## Top-Level Exports (`@app.function`)

Makes notebook functions importable by other notebooks or scripts.

```python
with app.setup:
    import polars as pl
    THRESHOLD = 1.2

@app.function
def filter_creatinine(df: pl.DataFrame) -> pl.DataFrame:
    return df.filter(pl.col("lab_value_numeric") > THRESHOLD)
```

Rules: must be pure; only reference symbols from `app.setup`; one definition per decorated cell.

## Batch Execution (Pydantic CLI/UI Hybrid)

Define parameters with Pydantic, configure via UI interactively or CLI in production.

1. Declare a Pydantic `BaseModel` with typed fields and descriptions
2. Detect mode with `mo.app_meta().mode == "script"`
3. In script mode, parse CLI args; in interactive mode, render UI controls

Guidelines:
- Use `python-dotenv` + `EnvConfig` for API keys/credentials
- CLI hyphens convert to Python underscores (`--my-param` → `my_param`)
- Optionally log params to W&B with `wandb_project` / `wandb_run_name` fields

## anywidget Components

Create custom JS/CSS widgets in marimo using anywidget.

```python
widget = mo.ui.anywidget(MyCustomWidget())
# Access state: widget.value (returns dict)
```

JavaScript: define a `render({ model, el })` function. Use `model.get()`/`model.set()`/`model.save_changes()` for state, `model.on("change:traitname", cb)` for reactivity.

CSS: include `_css` with `@media (prefers-color-scheme: dark)` support. For complex widgets, reference external files via `pathlib`.

## WASM Deployment

```bash
marimo export html-wasm notebook.py -o output_dir --mode run
```

- `--mode run` — app mode; `--mode edit` — editable
- Self-contained; deploy to GitHub Pages, S3, or any static host

## Performance Tips

- Gate expensive cells with `mo.stop()`
- `@mo.cache` for within-session; `@mo.persistent_cache` for cross-session
- Wrap expensive operations in `mo.ui.form()` to batch inputs
- Load heavy data in early cells; filter lazily downstream

## CLI Reference

| Command | Purpose |
|---|---|
| `marimo edit notebook.py` | Interactive editing |
| `marimo run notebook.py` | App mode (read-only) |
| `marimo check notebook.py` | Validate notebook |
| `marimo export html notebook.py` | Static HTML export |
| `marimo export html-wasm notebook.py -o dir` | WASM export |
| `marimo convert notebook.ipynb` | Convert from Jupyter |
| `marimo new` | Create new notebook |
