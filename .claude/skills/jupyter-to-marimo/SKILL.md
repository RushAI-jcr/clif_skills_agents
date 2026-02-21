---
name: jupyter-to-marimo
language: python
description: Convert Jupyter notebooks (.ipynb) to marimo (.py) format using uvx marimo convert. Use when migrating notebooks.
---

# Converting Jupyter Notebooks to Marimo

When asked to translate a notebook, ALWAYS run the conversion command first before examining files — converted Python files are much smaller than raw notebook files.

## Steps

1. **Convert**: `uvx marimo convert <notebook.ipynb> -o <notebook.py>`
2. **Validate**: `uvx marimo check <notebook.py>` — fix any reported errors
3. **Refine**:
   - Verify metadata block includes all dependencies
   - Remove Jupyter-specific remnants (`display()` calls, magic commands)
   - Ensure each cell's final expression produces renderable output
   - Consider `EnvConfig` widgets from wigglystuff for environment variables
   - Replace ipywidgets with marimo's `mo.ui` equivalents
4. **Revalidate**: Run `marimo check` again to confirm no issues
