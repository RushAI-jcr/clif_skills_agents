# Start CLIF Session

Get up to speed on this CLIF project and prepare for focused work.

## Steps

### 1. Detect Project Language

Scan the project root for language indicators:
- **R project**: `.R`, `.Rmd`, `.qmd` files, `renv.lock`, `config/config_template.yaml`
- **Python project**: `.py` files, `pyproject.toml`, `uv.lock`, `clif_config.json`
- If both are present, ask which language to use for this session

### 2. Check Environment

**Python**: Verify `clifpy`, `polars`, and `duckdb` are installed. Check for marimo or Jupyter as appropriate.

**R**: Verify `arrow`, `dplyr`, `data.table`, `duckdb`, `lubridate`, and `yaml` are installed. Check `renv.lock` if present.

### 3. Read Project Context

- Review project-level instructions or README for goals, architecture, and CLIF tables in use
- Review config file (Python: `clif_config.json` / R: `config/config_template.yaml`)
- Check for prior session notes, progress logs, or to-do lists

### 4. Present Current State

Summarize:
- What was accomplished in the last session
- Open items, blockers, or unresolved decisions
- Current recommended next steps
- CLIF tables in use and site configuration
- Data quality issues previously noted

### 5. Confirm Focus

Ask: "What would you like to work on today?" — pick from existing tasks or describe something new.

### 6. Plan Before Coding

- Explore the relevant codebase
- Clarify assumptions and constraints (eligibility criteria, CLIF table versions, `_category` values)
- Propose an implementation or analysis plan
- Get confirmation before writing code
