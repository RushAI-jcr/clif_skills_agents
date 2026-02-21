# CLIF Skills & Agents

> Set up guide for any AI coding agent to work with CLIF clinical data.

Python or R. Marimo or Jupyter or Quarto. It auto-detects your stack.

> **PHI Disclaimer:** If no BAA is in place for your LLM use, use synthetic CLIF data for code generation. Develop and test your code with synthetic data, then run the finished code on your real CLIF data locally.

---

## Setup

### Claude Code

```bash
cd your-clif-project/
cp -r /path/to/clif_skills_agents/.claude .claude
cp /path/to/clif_skills_agents/CLAUDE.md .
cp /path/to/clif_skills_agents/SKILL.md .
```

### Kilo Code (VS Code)

```bash
cp /path/to/clif_skills_agents/AGENTS.md .
cp -r /path/to/clif_skills_agents/.kilocode .kilocode
```

### Codex CLI

```bash
cp /path/to/clif_skills_agents/AGENTS.md .
```

---

## Usage

```
/start-session
```

Done. It detects your stack automatically:

| Your files | What it loads |
|------------|---------------|
| `.py` + marimo | Python + marimo skills |
| `.ipynb` | Python + Jupyter skills |
| `.R` `.Rmd` `renv.lock` | R + RMarkdown skills |
| `.qmd` | R + Quarto skills |

Then just ask it to write CLIF code. It knows the rules.

---

## Data config

**Python** — `clif_config.json`:
```json
{"data_directory": "/path/to/parquet", "timezone": "US/Eastern", "site_name": "YOUR_SITE"}
```
```bash
pip install clifpy polars duckdb marimo
```

**R** — `config/config_template.yaml`:
```yaml
site: "YOUR_SITE"
clif_data_path: "/path/to/parquet"
site_time_zone: "US/Eastern"
```
```r
install.packages(c("arrow", "dplyr", "tidyverse", "duckdb", "lubridate", "quarto"))
```

---

## What it knows

**8 enforced rules** — filter on `_category` not `_name`, validate before analysis, use ADT for ICU times, log attrition at every step, stitch encounters, separate continuous vs intermittent meds, UTC timestamps, JAMA figure style.

**Full CLIF 2.1 schema** — 28 tables, every column, every data type, every allowed category value.

**16 pitfalls** with wrong/right code in both Python and R.

**clifpy API** — return schemas, error handling, memory patterns.

**mCIDE categories** — labs, vitals, meds, assessments, respiratory, microbiology.

---

## Links

| | |
|---|---|
| [clif-consortium.org](https://clif-consortium.org/) | Main site |
| [Data Dictionary](https://clif-consortium.org/data-dictionary/) | Schema reference |
| [clifpy](https://pypi.org/project/clifpy/) | Python package |
| [CLIF-TableOne](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF-TableOne) | Table 1 + CONSORT |
| [CLIF-101](https://common-longitudinal-icu-data-format.github.io/CLIF-101/) | Onboarding docs |

---

MIT License
