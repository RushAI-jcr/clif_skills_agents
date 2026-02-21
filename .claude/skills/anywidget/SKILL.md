---
name: anywidget
language: python
description: Create custom anywidget components in marimo notebooks using vanilla JavaScript and CSS.
---

# anywidget in Marimo

Create custom anywidget components in marimo notebooks using vanilla JavaScript and CSS.

## JavaScript

Define a `render` function that takes `{ model, el }` as parameters and export it as default. Use:
- `model.get()` / `model.set()` / `model.save_changes()` for state management
- `model.on("change:traitname", callback)` for reactive updates

## CSS

Include `_css` styling that remains minimal. Accommodate light and dark modes:
```css
@media (prefers-color-scheme: dark) { ... }
```

## Display in Marimo

Wrap widgets with marimo's anywidget wrapper:
```python
widget = mo.ui.anywidget(OriginalAnywidget())
```

Access state values through `widget.value` (returns a dictionary).

## Philosophy

Dumber is better. Prefer obvious, direct code over clever abstractions. For complex widgets with substantial JS/CSS, reference external files via `pathlib` rather than embedding inline.

Keep examples minimal and focused on core functionality.
