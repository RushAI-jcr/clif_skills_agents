---
name: rmarkdown-notebook
language: r
description: RMarkdown (.Rmd) notebook rendering and parameterized reports. For legacy CLIF projects; prefer Quarto for new work.
---

# RMarkdown Notebook Skill

RMarkdown (`.Rmd`) is still used in many CLIF consortium projects. For new projects, prefer Quarto (`.qmd`).

## Rendering

```r
rmarkdown::render("00_cohort.Rmd", params = list(config_path = "config/config_template.yaml"))
```

Or from the command line:
```bash
Rscript -e 'rmarkdown::render("00_cohort.Rmd")'
```

## YAML Header

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

## Chunk Options

Use knitr chunk options in the chunk header:

````markdown
```{r load-data, message=FALSE, warning=FALSE}
library(arrow)
library(dplyr)
config <- yaml::read_yaml(params$config_path)
```
````

Common options: `echo`, `eval`, `message`, `warning`, `fig.width`, `fig.height`, `cache`

## Numbered Script Convention

```
scripts/
├── 00_cohort.Rmd
├── 01_data_quality.Rmd
├── 02_descriptive.Rmd
├── 03_analysis.Rmd
└── 04_figures.Rmd
```

## Parameterized Reports

```r
# Render for a specific site
rmarkdown::render("00_cohort.Rmd",
  params = list(config_path = "config/site_b_config.yaml"),
  output_file = "00_cohort_site_b.html"
)
```

## When to Use RMarkdown vs Quarto

- **Use RMarkdown** if the existing project already uses `.Rmd` files
- **Use Quarto** for new projects — better multi-language support, modern features, active development
