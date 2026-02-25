---
name: jupyter-notebook
language: python
description: Jupyter notebook conventions for CLIF projects — structure, cleanup, and migration to marimo. Use when working with .ipynb files.
---

# Jupyter Notebook Conventions

## Structure

Follow the CLIF numbered convention:
```
notebooks/
├── 00_cohort.ipynb
├── 01_data_quality.ipynb
├── 02_analysis.ipynb
└── 03_figures.ipynb
```

Each notebook: imports first, one task per cell, clear outputs before committing.

## Cleanup Checklist

- Remove empty cells
- Clear all outputs (`Cell → All Output → Clear`)
- Add markdown section headers between logical blocks
- Extract repeated code into `utils.py` or shared modules
- Set random seeds at the top for reproducibility

## Key Commands

```bash
# Format
black notebook.ipynb

# Lint
nbqa ruff notebook.ipynb

# Convert to script
jupyter nbconvert --to python notebook.ipynb

# Execute and save
jupyter nbconvert --to notebook --execute notebook.ipynb
```

## Migration to marimo

If the project is moving to marimo, use the jupyter-to-marimo skill:
1. `uvx marimo convert notebook.ipynb -o notebook.py`
2. `uvx marimo check notebook.py`
3. Remove Jupyter-specific remnants (`display()`, magics, ipywidgets)
4. Replace ipywidgets with `mo.ui` equivalents
