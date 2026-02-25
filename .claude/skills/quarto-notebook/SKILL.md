---
name: quarto-notebook
language: r
description: Quarto (.qmd) and RMarkdown (.Rmd) notebook development for R-based CLIF projects — YAML headers, code chunks, rendering, parameterized reports, and CLIF-specific patterns.
---

# Quarto & RMarkdown Notebooks

Quarto (`.qmd`) is the preferred notebook format for R-based CLIF projects. RMarkdown (`.Rmd`) is supported for legacy projects.

## Quarto (.qmd) — Preferred

### Modes

- **Interactive preview**: `quarto preview notebook.qmd` — live reload in browser
- **Batch render**: `quarto render notebook.qmd` — produce output file

### YAML Header

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

### Chunk Options

Use `#|` comment syntax:

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

Common options: `label`, `echo: false`, `eval: false`, `message: false`, `warning: false`, `fig-width: 8`, `fig-height: 6`, `cache: true`

### Output Formats

- `format: html` — interactive, default
- `format: pdf` — for manuscripts (requires LaTeX)
- `format: docx` — for collaborator review

### Parameterized Execution

```bash
quarto render 00_cohort.qmd -P config_path:config/site_b_config.yaml
```

## RMarkdown (.Rmd) — Legacy

Use RMarkdown only if the existing project already uses `.Rmd` files. For new projects, use Quarto.

### YAML Header

```yaml
---
title: "Cohort Derivation"
author: "CLIF Consortium"
output:
  html_document:
    toc: true
    code_folding: hide
params:
  config_path: "config/config_template.yaml"
---
```

### Chunk Options

Use knitr chunk options in the chunk header:

````markdown
```{r load-data, message=FALSE, warning=FALSE}
library(arrow)
library(dplyr)
config <- yaml::read_yaml(params$config_path)
```
````

### Rendering

```r
rmarkdown::render("00_cohort.Rmd",
  params = list(config_path = "config/site_b_config.yaml"),
  output_file = "00_cohort_site_b.html"
)
```

## CLIF-Specific Patterns

### Numbered Script Convention

```
scripts/
├── 00_cohort.qmd      # (or .Rmd for legacy)
├── 01_data_quality.qmd
├── 02_descriptive.qmd
├── 03_analysis.qmd
└── 04_figures.qmd
```

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
