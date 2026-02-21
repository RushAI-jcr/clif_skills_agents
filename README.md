# CLIF Skills & Agents

> **Pre-built AI assistant configuration for [CLIF](https://clif-consortium.org/) clinical data projects.**
> Works with Python or R. Supports Claude Code, Kilo Code (VS Code), and Codex CLI.

CLIF (Common Longitudinal ICU Format) standardizes ICU data across hospitals. This repo gives your AI coding assistant deep knowledge of CLIF conventions, data pitfalls, and best practices — so it writes correct clinical data code from the start.

---

## Quick Start

Pick your platform and language, then copy the config into your CLIF project.

### Claude Code (full features)

```bash
cd your-clif-project/

# Copy everything:
cp -r /path/to/clif_skills_agents/.claude .claude
cp /path/to/clif_skills_agents/CLAUDE.md ./CLAUDE.md
cp /path/to/clif_skills_agents/SKILL.md ./SKILL.md
```

Then start a session:
```
/start-session
```

The assistant auto-detects your language (Python or R), loads the right skills, checks your environment, and resumes where you left off.

### Kilo Code (VS Code)

```bash
cd your-clif-project/

cp /path/to/clif_skills_agents/AGENTS.md ./AGENTS.md
cp -r /path/to/clif_skills_agents/.kilocode .kilocode
```

### Codex CLI

```bash
cd your-clif-project/

cp /path/to/clif_skills_agents/AGENTS.md ./AGENTS.md
```

> Codex CLI only reads `AGENTS.md` — no slash commands, skills, or settings. It still gets all 8 CLIF data rules and language-specific conventions.

---

## Platform Comparison

| Feature | Claude Code | Kilo Code (VS Code) | Codex CLI |
|---------|:-----------:|:-------------------:|:---------:|
| CLIF data rules | CLAUDE.md | AGENTS.md + rules/ | AGENTS.md |
| Start/end session | Slash commands | Workflows | -- |
| Domain knowledge (12 skills) | Yes | -- | -- |
| Extended thinking + hooks | Yes | -- | -- |
| JAMA figure style | Yes | Rules file | AGENTS.md |

---

## Python Setup

Install dependencies:

```bash
pip install clifpy polars duckdb marimo
```

Create `clif_config.json` in your project root:

```json
{
  "data_directory": "/path/to/clif/parquet/files",
  "timezone": "US/Eastern",
  "site_name": "YOUR_SITE"
}
```

**What the assistant knows (Python):**
- Use **Polars** for data processing (Pandas accepted but Polars preferred)
- Validate with `clif.validate_all()` before any analysis
- Convert timezones with clifpy's `timezone` parameter
- Build notebooks in **marimo** (preferred) or **Jupyter**
- Generate JAMA-style figures with matplotlib/seaborn

---

## R Setup

Install dependencies:

```r
install.packages(c(
  "arrow", "dplyr", "tidyverse", "data.table",
  "duckdb", "lubridate", "yaml", "table1",
  "rmarkdown", "quarto"
))

# If renv.lock exists:
renv::restore()
```

Create `config/config_template.yaml`:

```yaml
site: "YOUR_SITE"
file_type: "parquet"
clif_data_path: "/path/to/clif/parquet/files"
site_time_zone: "US/Eastern"
```

**What the assistant knows (R):**
- Use **tidyverse + arrow** for data manipulation
- Verify schemas against TABLES.md or use CLIF-TableOne validation
- Convert timezones with `lubridate::with_tz()`
- Build notebooks in **Quarto** (`.qmd`, preferred) or **RMarkdown** (`.Rmd`)
- Generate JAMA-style figures with ggplot2

---

## The 8 CLIF Data Rules

These rules are enforced in every platform configuration. They prevent the most common cross-site bugs:

1. **Filter on `_category`, never `_name`.** Names are site-specific EHR strings. Categories are the CLIF controlled vocabulary.
2. **Validate before analysis.** Python: `clif.validate_all()` | R: verify schemas or use CLIF-TableOne.
3. **Use ADT for ICU times, not hospitalization.** `hospitalization.admission_dttm` is hospital admission. Filter `adt` where `location_category == 'icu'`.
4. **Log row counts at every filter step.** CONSORT-style attrition tracking is required.
5. **Use encounter stitching.** A single ICU stay may span multiple hospitalization records.
6. **Distinguish continuous vs intermittent meds.** `medication_admin_continuous` = drips. `medication_admin_intermittent` = boluses.
7. **All timestamps are UTC.** Convert with clifpy (Python) or `lubridate::with_tz()` (R).
8. **JAMA figure style.** Arial font, 8pt minimum, no text overlap, legends outside the plot area.

---

## Session Workflow

```
/start-session  -->  work on CLIF data  -->  /end-session
```

**Start session** auto-detects your stack and:
- Loads language-appropriate skills (Claude Code)
- Checks that required packages are installed
- Reads your config file (`clif_config.json` or `config_template.yaml`)
- Resumes context from previous sessions

**End session** persists state for next time:
- Logs what you accomplished and key decisions
- Updates your task list
- Records lessons learned from corrections

State files (`.claude/claude-progress.md`, `.claude/claude-todo.md`, `.claude/lessons.md`) are gitignored — they stay local.

---

## Skills Reference (Claude Code)

Claude Code loads domain-specific skills based on your detected language:

| Skill | Language | What it teaches the assistant |
|-------|----------|-------------------------------|
| `clif-data` | Both | CLIF table schemas, `_category` rules, cohort building, data pitfalls |
| `python-development` | Python | Polars patterns, packaging, testing |
| `r-development` | R | Tidyverse, rlang, performance, package development |
| `r-data-tooling` | R | Arrow, DuckDB, data.table integration |
| `marimo-notebook` | Python | Reactivity, SQL cells, UI components, import rules |
| `jupyter-notebook` | Python | Notebook organization and optimization |
| `jupyter-to-marimo` | Python | Migration from `.ipynb` to marimo |
| `marimo-batch` | Python | CLI execution with Pydantic parameters |
| `anywidget` | Python | Custom JS/CSS interactive widgets |
| `quarto-notebook` | R | Quarto rendering, parameters, CLIF patterns |
| `rmarkdown-notebook` | R | Knitr, parameterized reports |
| `scientific-figures` | Both | JAMA style, matplotlib/seaborn/ggplot2 best practices |

**Stack auto-detection:**

| If the project has... | Stack | Skills loaded |
|-----------------------|-------|---------------|
| `.py` with `import marimo`, `marimo` in pyproject.toml | Python + marimo | clif-data, python-development, marimo-notebook, marimo-batch, anywidget, scientific-figures |
| `.ipynb` files, no marimo | Python + Jupyter | clif-data, python-development, jupyter-notebook, scientific-figures |
| Both `.ipynb` and marimo | Migration | clif-data, python-development, jupyter-to-marimo, marimo-notebook, marimo-batch, anywidget, scientific-figures |
| `.R`, `.Rmd`, `.qmd`, `renv.lock` | R | clif-data, r-development, r-data-tooling, quarto-notebook, rmarkdown-notebook, scientific-figures |

> **Note:** The root `SKILL.md` references data files (CSV schemas, data dictionaries) that live in your target CLIF project, not in this repo. Copy `SKILL.md` into a project that has those files.

---

## CLIF Ecosystem

| Tool | Language | Purpose | Link |
|------|----------|---------|------|
| clifpy | Python | Load, validate, score CLIF data | `pip install clifpy` / [PyPI](https://pypi.org/project/clifpy/) |
| CLIF-TableOne | Both | Table 1, CONSORT diagrams, ECDF plots | [GitHub](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF-TableOne) |
| CLIF-Lighthouse | Python | Interactive QC dashboard | [GitHub](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF-Lighthouse) |
| CLIF-MIMIC | Python | MIMIC-IV to CLIF ETL pipeline | [GitHub](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF-MIMIC) |
| CLIF-101 | Docs | Onboarding documentation | [Docs](https://common-longitudinal-icu-data-format.github.io/CLIF-101/) |

**Learn more:** [clif-consortium.org](https://clif-consortium.org/) | [CLIF GitHub](https://github.com/Common-Longitudinal-ICU-data-Format) | [Data Dictionary](https://clif-consortium.org/data-dictionary/)

---

## Repository Structure

```
.
├── CLAUDE.md                          # Project rules (Claude Code)
├── AGENTS.md                          # Project rules (Kilo Code / Codex CLI)
├── SKILL.md                           # Root skill manifest (Claude Code)
├── .claude/
│   ├── commands/
│   │   ├── start-session.md           # /start-session command
│   │   └── end-session.md             # /end-session command
│   ├── settings.json                  # Extended thinking + hooks
│   └── skills/                        # 12 domain knowledge skills
│       ├── clif-data/                 #   includes references/TABLES.md, PITFALLS.md, etc.
│       ├── python-development/
│       ├── r-development/             #   includes references/rlang-patterns.md, etc.
│       ├── r-data-tooling/
│       ├── marimo-notebook/           #   includes references/SQL.md, UI.md, etc.
│       ├── jupyter-notebook/
│       ├── jupyter-to-marimo/
│       ├── marimo-batch/
│       ├── anywidget/
│       ├── quarto-notebook/
│       ├── rmarkdown-notebook/
│       └── scientific-figures/        #   includes references/matplotlib.md, ggplot2.md, etc.
├── .kilocode/
│   ├── rules/
│   │   ├── clif-data-rules.md         # CLIF data rules (Kilo Code)
│   │   └── figure-style.md            # JAMA figure rules (Kilo Code)
│   └── workflows/
│       ├── start-session.md           # Start session workflow
│       └── end-session.md             # End session workflow
└── .gitignore
```

---

## License

MIT
