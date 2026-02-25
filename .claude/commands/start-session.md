# Start CLIF Session

Get up to speed on this CLIF project and prepare for focused work.

## 1. Detect Project Language

Scan the project root for language indicators:
- **R project**: `.R`, `.Rmd`, `.qmd` files, `renv.lock`, `config/config_template.yaml`
- **Python project**: `.py` files, `pyproject.toml`, `uv.lock`, `clif_config.json`
- If both are present, ask the user which language to use for this session

## 2. Load Language-Specific Skills

Based on the detected language, only reference skills matching that stack:

- **Python + marimo** (`.py` marimo files, `marimo` in dependencies): clif-data, python-development, marimo-notebook, marimo-pro, scientific-figures
- **Python + Jupyter** (`.ipynb` files, no marimo indicators): clif-data, python-development, jupyter-notebook, scientific-figures
- **Python migrating to marimo** (both `.ipynb` and marimo present, or user requests migration): clif-data, python-development, jupyter-to-marimo, marimo-notebook, marimo-pro, scientific-figures
- **R projects**: clif-data, r-development, r-data-tooling, quarto-notebook, scientific-figures
- **Both languages detected**: use clif-data (shared), ask the user which language, then apply the rules above

**Detection heuristics for Python notebook type:**
- marimo: `.py` files with `import marimo as mo` or `app = mo.App()`, `marimo` in `pyproject.toml` dependencies
- Jupyter: `.ipynb` files present
- If ambiguous, ask the user: "This project has both .ipynb and marimo files — are you working in marimo or Jupyter?"

**Do NOT cross-reference stacks.** Never mention marimo in a Jupyter or R session, never mention Jupyter in a marimo session, never mention Quarto/RMarkdown in a Python session.

## 3. Environment Check

### If Python project:
- Check that `clifpy` is installed: `pip show clifpy`. If missing, remind to run:
  ```
  pip install clifpy
  ```
- Verify DuckDB and Polars are available: `python -c "import duckdb; import polars"`
- **If marimo stack**: check `pip show marimo`. If missing, remind to install.
- **If Jupyter stack**: check `pip show jupyter`. If missing, remind to install.

### If R project:
- Check R is available: `Rscript --version`
- Check key packages are installed:
  ```r
  Rscript -e 'for (pkg in c("arrow", "dplyr", "data.table", "duckdb", "lubridate", "yaml")) cat(pkg, ": ", requireNamespace(pkg, quietly=TRUE), "\n")'
  ```
- Check `renv.lock` exists — if so, suggest `renv::restore()` if packages are missing
- Verify data loading works: `Rscript -e 'library(arrow); cat("arrow OK\n")'`

## 4. Read Project Context
- Review `./CLAUDE.md` to understand:
  - Project goals and architecture
  - Key commands and workflows
  - Implementation notes (timezone handling, eligibility criteria, CLIF tables)
- Review config file:
  - Python: `./clif_config.json` or `./config.json`
  - R: `./config/config_template.yaml` or `./config/site_config.yaml`
- Review `./.claude/claude-progress.md` (if it exists) to understand:
  - What has already been completed
  - Prior decisions and rationale
  - Open questions or unfinished work
- Review `./.claude/claude-todo.md` (if it exists) to see:
  - Pending tasks
  - Prioritized next steps
- Review `./.claude/lessons.md` for patterns and mistakes to avoid

## 5. Present Current State
Summarize for the user:
- What was accomplished in the **last session**
- Any **open items**, blockers, or unresolved decisions
- The **current recommended next steps**
- Which CLIF tables are in use and current site configuration
- Any data quality issues previously noted

## 6. Confirm Focus for This Session
Ask the user:
- "What would you like to work on today?"
  - Choose from existing to-do items, **or**
  - Describe a new task or change in direction

## 7. Plan Before Coding
Once a task is selected:
- Explore the relevant codebase or documents
- Clarify assumptions and constraints (eligibility criteria, CLIF table versions, _category values)
- Propose an implementation or analysis plan
- For interactive analysis or exploration:
  - **Python (marimo)**: prefer **marimo notebooks** (`marimo edit`) — use for data exploration, cohort building, and visualization
  - **Python (Jupyter)**: use **Jupyter notebooks** (`.ipynb`) for interactive analysis
  - **R**: prefer **Quarto notebooks** (`.qmd`) or **RMarkdown** (`.Rmd`) for interactive analysis
- Get confirmation before writing code or making changes
