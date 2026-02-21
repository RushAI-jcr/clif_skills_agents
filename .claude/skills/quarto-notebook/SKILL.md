---
name: quarto-notebook
language: r
description: Quarto (.qmd) notebook development for R-based CLIF projects — YAML headers, code chunks, rendering, and Quarto extensions.
---

# Quarto Notebook Skill

Quarto (`.qmd`) is the preferred notebook format for R-based CLIF projects. It supports R, Python, and Observable JS in a single document.

## Modes

- **Interactive preview**: `quarto preview notebook.qmd` — live reload in browser
- **Batch render**: `quarto render notebook.qmd` — produce output file

## YAML Header

```yaml
---
title: "Cohort Derivation"
author: "CLIF Consortium"
format:
  html:
    toc: true
    code-fold: true
params:
  config_path: "config/config_template.yaml"
---
```

Access parameters in R chunks: `params$config_path`

## Chunk Options

Use `#|` comment syntax for chunk options:

````markdown
```{r}
#| label: load-data
#| message: false
#| warning: false

library(arrow)
library(dplyr)
library(yaml)

config <- read_yaml(params$config_path)
```
````

Common chunk options:
- `#| label:` — unique chunk identifier
- `#| echo: false` — hide code in output
- `#| eval: false` — show code but don't run
- `#| message: false` — suppress messages
- `#| warning: false` — suppress warnings
- `#| fig-width: 8` / `#| fig-height: 6` — figure dimensions
- `#| cache: true` — cache chunk results

## Numbered Script Convention

Follow the CLIF convention for multi-step analyses:

```
scripts/
├── 00_cohort.qmd
├── 01_data_quality.qmd
├── 02_descriptive.qmd
├── 03_analysis.qmd
└── 04_figures.qmd
```

## Output Formats

- `format: html` — interactive, default
- `format: pdf` — for manuscripts (requires LaTeX)
- `format: docx` — for collaborator review

## Parameterized Execution

Run with different configs:
```bash
quarto render 00_cohort.qmd -P config_path:config/site_b_config.yaml
```

## CLIF-Specific Patterns

### Config-driven data loading
```r
config <- read_yaml(params$config_path)
load_clif_table <- function(table_name) {
  file.path(config$clif_data_path,
    paste0(config$site, "_clif_", table_name, ".", config$file_type)) |>
    arrow::read_parquet()
}
```

### CONSORT attrition logging
```r
log_attrition <- function(step, df) {
  message(sprintf("%s: n = %s", step, format(nrow(df), big.mark = ",")))
}
```
