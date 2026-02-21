---
name: r-data-tooling
language: r
description: R data tooling for CLIF projects — Arrow lazy evaluation, DuckDB integration, renv reproducibility, tableone/gtsummary Table 1s, encounter stitching, SOFA scoring, ventilator episodes, vasopressor dosing, federated analysis, and Rscript batch pipelines. Use when writing R code that loads, transforms, or summarizes CLIF data.
---

# R Data Tooling

Data engineering and analysis patterns for R-based CLIF projects. Covers Arrow, DuckDB, renv, summary tables, and canonical implementations of algorithms that Python gets from clifpy but R must implement manually.

## Core Principles

1. **Lazy by default** — use `open_dataset()`, filter/select before `collect()`
2. **renv for every project** — no exceptions for multi-site reproducibility
3. **Never `collect()` too early** — the #1 R-specific performance mistake
4. **tableone or gtsummary for Table 1** — publication-ready, JAMA-compatible output
5. **Cohort ID filter table** — maintain a small running ID table, join-then-filter everywhere
6. **Normalize units before computing** — FiO2 0-100 vs 0-1, weight-based dosing conversions

## Arrow Lazy Evaluation

### open_dataset() over read_parquet()

`read_parquet()` loads everything into memory. `open_dataset()` builds a query plan and only materializes after filtering.

```r
library(arrow)
library(dplyr)

# GOOD: lazy — only creatinine rows enter memory
labs <- open_dataset(
  file.path(config$clif_data_path, paste0(config$site, "_clif_labs.", config$file_type))
) |>
  filter(lab_category == "creatinine") |>
  select(encounter_id, lab_value_numeric, lab_collect_dttm) |>
  collect()

# BAD: eager — entire labs table loaded first
labs <- read_parquet(path) |>
  filter(lab_category == "creatinine")
```

### Predicate and Projection Pushdown

Arrow pushes `filter()` and `select()` into the Parquet reader, reading only the rows and columns needed:

```r
# Only reads 3 columns and matching rows from disk
vitals_sbp <- open_dataset(vitals_path) |>
  filter(vital_category == "sbp") |>
  select(encounter_id, vital_value, vital_dttm) |>
  collect()
```

### large_utf8 Casting Workaround

Arrow sometimes reads strings as `large_utf8`, breaking joins. Cast immediately after `open_dataset()`:

```r
cast_large_utf8_to_utf8 <- function(x) {
  sch <- arrow::schema(x)
  large_cols <- vapply(
    sch$fields,
    function(f) f$type$ToString() %in% c("large_string", "large_utf8"),
    logical(1)
  )
  if (any(large_cols)) {
    x |> mutate(across(all_of(sch$names[large_cols]), ~ arrow::cast(.x, arrow::utf8())))
  } else {
    x
  }
}

# Apply immediately after open_dataset(), before any filter/join
labs <- open_dataset(labs_path) |>
  cast_large_utf8_to_utf8() |>
  filter(lab_category == "creatinine") |>
  collect()
```

### Arrow Anti-Patterns

```r
# BAD: collect then filter (loads everything)
df <- read_parquet(path) |> collect() |> filter(lab_category == "creatinine")

# BAD: multiple collects in a pipeline
step1 <- open_dataset(path) |> filter(x > 1) |> collect()
step2 <- step1 |> mutate(y = x * 2)  # already in memory, fine
# Should have done both operations before collect()

# GOOD: chain all operations, collect once at the end
result <- open_dataset(path) |>
  filter(x > 1) |>
  mutate(y = x * 2) |>
  collect()
```

## DuckDB Integration

Use DuckDB when you need complex SQL (window functions, CTEs) or data exceeds memory even with Arrow lazy eval.

```r
library(duckdb)
library(DBI)

con <- dbConnect(duckdb::duckdb())

# Register Arrow dataset (zero-copy)
labs_arrow <- arrow::open_dataset(labs_path)
duckdb::duckdb_register_arrow(con, "labs", labs_arrow)

# Complex query with SQL
result <- dbGetQuery(con, "
  WITH ranked AS (
    SELECT *,
      ROW_NUMBER() OVER (
        PARTITION BY encounter_id
        ORDER BY lab_collect_dttm
      ) AS rn
    FROM labs
    WHERE lab_category = 'lactate'
  )
  SELECT * FROM ranked WHERE rn = 1
")

dbDisconnect(con)
```

### Arrow ↔ DuckDB Handoff

Use `to_duckdb()` / `to_arrow()` to move between engines mid-pipeline:

```r
# Start in Arrow, switch to DuckDB for window functions, back to Arrow
result <- open_dataset(vitals_path) |>
  filter(vital_category == "sbp") |>
  to_duckdb() |>
  group_by(encounter_id) |>
  filter(vital_value == min(vital_value, na.rm = TRUE)) |>
  to_arrow() |>
  collect()
```

### When to Use DuckDB vs Arrow

| Use DuckDB | Use Arrow |
|---|---|
| Window functions, CTEs | Simple filter/select/mutate |
| Complex multi-table joins | Single-table operations |
| SQL-native aggregations | dplyr-style pipelines |
| Data too large even for lazy Arrow | Data fits with lazy evaluation |

## renv Reproducibility

### Full Lifecycle

```r
# 1. Initialize in a new project (creates renv.lock, .Rprofile, renv/)
renv::init()

# 2. Install packages (goes through renv, not install.packages)
renv::install("arrow")
renv::install("dplyr")
renv::install("duckdb")
renv::install("gtsummary")
renv::install("lubridate")

# 3. Snapshot current state to lockfile
renv::snapshot()

# 4. Restore on another machine or site
renv::restore()

# 5. Check status — are lockfile and library in sync?
renv::status()
```

### Key Rules

- **Always commit `renv.lock`** to version control — this is the reproducibility contract
- **Use `renv::install()`, not `install.packages()`** — renv tracks what you install
- **Run `renv::snapshot()` after adding/updating packages** — keeps lockfile current
- **Run `renv::restore()` at new sites before any analysis** — ensures identical environments
- **Pin versions for CLIF consortium work** — avoid surprises across 20+ sites

### renv with Quarto

Quarto projects work with renv automatically. Ensure `renv::restore()` runs before `quarto render`:

```bash
# At a new site
cd my-clif-project
Rscript -e 'renv::restore()'
quarto render
```

### Updating a Single Package

```r
renv::update("dplyr")     # update one package
renv::snapshot()           # record the change
```

## Cohort ID Filter Table Pattern

Maintain a small running table of eligible IDs. Update it after each exclusion step. Join it into every CLIF table to filter — never filter large Arrow tables with complex predicates directly.

```r
# Initialize after first inclusion criteria
cohort_ids <- adt |>
  filter(location_category == "icu") |>
  distinct(patient_id, hospitalization_id) |>
  mutate(in_cohort = 1L)

# After each exclusion, update
cohort_ids <- cohort_ids |>
  left_join(exclusion_flags, by = "hospitalization_id") |>
  filter(is.na(excluded)) |>
  select(patient_id, hospitalization_id, in_cohort)

# Log CONSORT count at every step
message(sprintf("After excluding <reason>: %s patients, %s encounters",
  n_distinct(cohort_ids$patient_id),
  nrow(cohort_ids)
))

# Use everywhere: join first, then work with the filtered data
labs_cohort <- labs |>
  inner_join(cohort_ids, by = "hospitalization_id")
```

## Table 1 Generation

### tableone (CLIF Consortium Standard)

The `tableone` package is the de facto standard in CLIF consortium projects. The export idiom produces consistently shaped CSVs that can be `rbind()`-ed across sites:

```r
library(tableone)

vars <- c("age", "sex", "race_category", "bmi", "sofa_score", "los_days")
cat_vars <- c("sex", "race_category")

tab <- CreateTableOne(
  vars = vars,
  factorVars = cat_vars,
  strata = "mortality_outcome",
  data = cohort,
  addOverall = TRUE
)

# Export as clean CSV for federated pooling
tab_df <- as.data.frame(
  print(tab, printToggle = FALSE, missing = TRUE, smd = TRUE)
) |>
  tibble::rownames_to_column("characteristic") |>
  mutate(site = config$site) |>
  relocate(site) |>
  rename(percent_missing = Missing) |>
  select(-test)

write.csv(tab_df, file.path("project_output", paste0(config$site, "_table1.csv")),
          row.names = FALSE)
```

### gtsummary (Alternative for Publication)

For more polished output (Word, LaTeX, HTML):

```r
library(gtsummary)

cohort |>
  select(age, sex, race_category, bmi, sofa_score, mortality_outcome) |>
  tbl_summary(
    by = mortality_outcome,
    label = list(age ~ "Age, y", sex ~ "Sex", race_category ~ "Race",
                 bmi ~ "BMI", sofa_score ~ "SOFA score"),
    statistic = list(
      all_continuous() ~ "{median} ({p25}, {p75})",
      all_categorical() ~ "{n} ({p}%)"
    ),
    missing = "no"
  ) |>
  add_overall() |>
  add_p() |>
  modify_header(label = "**Characteristic**") |>
  bold_labels()

# Export
tbl |> as_flex_table() |> flextable::save_as_docx(path = "table1.docx")
```

## Encounter Stitching (No clifpy Equivalent)

Python has `clif.encounter_stitching(gap_hours=24)`. R must implement manually:

```r
stitch_icu_encounters <- function(adt, gap_hours = 24) {
  adt |>
    filter(location_category == "icu") |>
    arrange(encounter_id, in_dttm) |>
    group_by(encounter_id) |>
    mutate(
      prev_out = lag(out_dttm),
      gap_hours_val = as.numeric(difftime(in_dttm, prev_out, units = "hours")),
      new_stay = is.na(gap_hours_val) | gap_hours_val > gap_hours,
      icu_stay_id = cumsum(new_stay)
    ) |>
    group_by(encounter_id, icu_stay_id) |>
    summarise(
      icu_in_dttm = min(in_dttm),
      icu_out_dttm = max(out_dttm),
      .groups = "drop"
    )
}
```

## FiO2 Unit Normalization

Sites store FiO2 as either 0-1 (fraction) or 0-100 (percentage). Normalize before computing PF ratios:

```r
normalize_fio2 <- function(fio2) {
  # If mean > 1, assume 0-100 scale; convert to fraction
  if (mean(fio2, na.rm = TRUE) > 1) fio2 / 100 else fio2
}
```

## SOFA Scoring (No clifpy Equivalent)

Python has `clif.compute_sofa_scores()`. R must score each component manually. Source data from CLIF tables within a 24-hour window before the enrollment timepoint.

```r
# --- Respiratory (PaO2/FiO2 ratio) ---
score_sofa_respiratory <- function(pf_ratio) {
  case_when(
    pf_ratio >= 400 ~ 0L, pf_ratio >= 300 ~ 1L,
    pf_ratio >= 200 ~ 2L, pf_ratio >= 100 ~ 3L,
    TRUE ~ 4L
  )
}

# --- Coagulation (platelets, x10^3/uL) ---
score_sofa_coagulation <- function(platelets) {
  case_when(
    platelets >= 150 ~ 0L, platelets >= 100 ~ 1L,
    platelets >= 50 ~ 2L, platelets >= 20 ~ 3L,
    TRUE ~ 4L
  )
}

# --- Liver (bilirubin, mg/dL) ---
score_sofa_liver <- function(bilirubin) {
  case_when(
    bilirubin < 1.2 ~ 0L, bilirubin < 2.0 ~ 1L,
    bilirubin < 6.0 ~ 2L, bilirubin < 12.0 ~ 3L,
    TRUE ~ 4L
  )
}

# --- Renal (creatinine, mg/dL) ---
score_sofa_renal <- function(creatinine) {
  case_when(
    creatinine < 1.2 ~ 0L, creatinine < 2.0 ~ 1L,
    creatinine < 3.5 ~ 2L, creatinine < 5.0 ~ 3L,
    TRUE ~ 4L
  )
}

# --- CNS (GCS total or RASS) ---
score_sofa_cns_gcs <- function(gcs) {
  case_when(
    gcs >= 15 ~ 0L, gcs >= 13 ~ 1L,
    gcs >= 10 ~ 2L, gcs >= 6 ~ 3L,
    TRUE ~ 4L
  )
}

score_sofa_cns_rass <- function(rass) {
  case_when(
    rass == 0 ~ 0L,
    rass %in% c(-1L, 1L) ~ 1L,
    rass %in% c(-2L, 2L) ~ 2L,
    rass %in% c(-3L, 3L) ~ 3L,
    TRUE ~ 4L
  )
}

# --- Cardiovascular (MAP + vasopressor dose) ---
# Uses norepinephrine equivalent dose (see below)
score_sofa_cardiovascular <- function(map, norepi_eq) {
  case_when(
    !is.na(norepi_eq) & norepi_eq > 0.1 ~ 4L,
    !is.na(norepi_eq) & norepi_eq > 0   ~ 3L,
    !is.na(map) & map < 70              ~ 2L,
    TRUE                                 ~ 0L
  )
}
```

### Combining Components

```r
# Source: labs (platelets, bilirubin, creatinine), vitals (MAP),
# patient_assessments (GCS, RASS), respiratory_support + labs (PF ratio),
# medication_admin_continuous (vasopressors)
sofa <- cohort |>
  mutate(
    sofa_resp = score_sofa_respiratory(pf_ratio),
    sofa_coag = score_sofa_coagulation(platelet_count),
    sofa_liver = score_sofa_liver(bilirubin_total),
    sofa_renal = score_sofa_renal(creatinine),
    sofa_cns = pmax(score_sofa_cns_gcs(gcs), score_sofa_cns_rass(rass), na.rm = TRUE),
    sofa_cardio = score_sofa_cardiovascular(map, norepi_eq),
    sofa_total = sofa_resp + sofa_coag + sofa_liver + sofa_renal + sofa_cns + sofa_cardio
  )
```

## Norepinephrine Equivalent Dose (Kotani Formula)

Convert all vasopressor infusions to a single norepinephrine-equivalent dose for SOFA scoring and severity comparisons:

```r
compute_norepi_equivalent <- function(meds, weight_kg = NULL) {
  # Normalize weight-based dosing if needed
  # mcg/min -> mcg/kg/min requires patient weight
  meds |>
    summarise(
      norepi_dose  = sum(dose[med_category == "norepinephrine"], na.rm = TRUE),
      epi_dose     = sum(dose[med_category == "epinephrine"], na.rm = TRUE),
      vaso_dose    = sum(dose[med_category == "vasopressin"], na.rm = TRUE),
      dopa_dose    = sum(dose[med_category == "dopamine"], na.rm = TRUE),
      phenyl_dose  = sum(dose[med_category == "phenylephrine"], na.rm = TRUE),
      angio_dose   = sum(dose[med_category == "angiotensin_ii"], na.rm = TRUE),
      .groups = "drop"
    ) |>
    mutate(
      norepi_eq = norepi_dose + epi_dose +
        vaso_dose * 2.5 +
        angio_dose * 0.0025 +
        dopa_dose / 100 +
        phenyl_dose * 0.06
    )
}
```

## Ventilator Episode Detection

Identify discrete ventilator episodes from `respiratory_support` with a liberation threshold (e.g., 24 hours off IMV = new episode):

```r
detect_vent_episodes <- function(resp_support, liberation_hours = 24) {
  resp_support |>
    filter(device_category == "IMV" | !is.na(device_category)) |>
    arrange(encounter_id, recorded_dttm) |>
    group_by(encounter_id) |>
    mutate(
      on_imv = device_category == "IMV",
      # Detect transitions off ventilator
      prev_on_imv = lag(on_imv, default = FALSE),
      gap_start = prev_on_imv & !on_imv,
      gap_end = !prev_on_imv & on_imv,
      # Compute time off vent
      time_off = if_else(
        !on_imv,
        as.numeric(difftime(lead(recorded_dttm), recorded_dttm, units = "hours")),
        NA_real_
      ),
      # Liberation = off vent for >= threshold
      liberated = !on_imv & time_off >= liberation_hours,
      # New episode after each liberation
      vent_episode = cumsum(on_imv & (row_number() == 1 | lag(liberated, default = FALSE)))
    ) |>
    filter(on_imv) |>
    group_by(encounter_id, vent_episode) |>
    summarise(
      vent_start = min(recorded_dttm),
      vent_end = max(recorded_dttm),
      .groups = "drop"
    )
}
```

## Rscript Batch Pipelines

### Numbered Script Convention

CLIF R projects typically use numbered scripts run sequentially:

```
scripts/
  00_cohort.R         # Define inclusion/exclusion
  01_data_pull.R      # Extract and join CLIF tables
  02_features.R       # Compute derived variables
  03_analysis.R       # Statistical models
  04_tables_figures.R # Output generation
```

### Running the Pipeline

```bash
# Run a single script
Rscript scripts/00_cohort.R

# Run entire pipeline sequentially
for script in scripts/0*.R; do
  echo "Running $script..."
  Rscript "$script" || exit 1
done
```

### Parameterized Scripts with commandArgs

```r
# At the top of the script
args <- commandArgs(trailingOnly = TRUE)
config_path <- if (length(args) >= 1) args[1] else "config/config_template.yaml"
config <- yaml::read_yaml(config_path)

message(sprintf("Running for site: %s", config$site))
```

```bash
# Run with a specific config
Rscript scripts/00_cohort.R config/site_config.yaml
```

### Multi-Site Loop

```bash
# Run the full pipeline for each site config
for cfg in config/site_*.yaml; do
  site=$(basename "$cfg" .yaml)
  echo "=== Processing $site ==="
  for script in scripts/0*.R; do
    Rscript "$script" "$cfg" || { echo "FAILED: $script for $site"; exit 1; }
  done
done
```

## Federated Analysis Patterns

### metafor for Standard Meta-Analysis

Pool site-level estimates when each site exports point estimates + standard errors:

```r
library(metafor)

# site_results: data frame with columns estimate, se, site
meta_fit <- rma(yi = estimate, sei = se, data = site_results, method = "REML")
summary(meta_fit)
forest(meta_fit, slab = site_results$site)
```

### Precision-Weighted Coefficient Pooling

When sites export full model coefficient vectors + covariance matrices (more powerful than standard meta-analysis, preserves multivariate correlation structure):

```r
# Each site exports: coefficients CSV + covariance matrix CSV
# Summary site loads them all:
est_list <- lapply(site_files, \(f) as.matrix(read.csv(f)$estimate))
cov_list <- lapply(cov_files, \(f) as.matrix(read.csv(f, row.names = 1)))

# Precision-weighted pooling
prec_list <- lapply(cov_list, solve)                    # precision = inverse covariance
global_cov <- solve(Reduce("+", prec_list))             # pooled covariance
global_est <- global_cov %*% Reduce("+", mapply(
  \(e, V) solve(V) %*% e, est_list, cov_list, SIMPLIFY = FALSE
))
```

Publish pooled coefficients to a shared location (e.g., GitHub raw URL) so each site can apply:

```r
# At each site: apply global propensity model
global_coefs <- read.csv(url("https://raw.githubusercontent.com/org/repo/main/global_coefficients.csv"))
X <- model.matrix(~ age + sex + sofa_score, data = cohort)
log_odds <- X %*% global_coefs$estimate
propensity <- plogis(log_odds)
```

## Site-Adaptive Modeling

CLIF sites vary: some have one hospital, others have many. Use `update()` to adapt model formulae dynamically:

```r
# Base formula includes hospital random effect
model_form <- outcome ~ age + sex + sofa_score + factor(hospital_id)

# Adapt for single-hospital sites
n_hospitals <- n_distinct(cohort$hospital_id[!is.na(cohort$hospital_id)])
if (n_hospitals <= 1) {
  model_form <- update(model_form, . ~ . - factor(hospital_id))
}

fit <- glm(model_form, data = cohort, family = binomial)
```

### Cluster-Robust Standard Errors

For multi-hospital sites, cluster SEs by hospital:

```r
library(sandwich)
library(lmtest)

fit <- glm(model_form, data = cohort, family = binomial)

if (n_hospitals > 1) {
  robust_vcov <- vcovCL(fit, cluster = cohort$hospital_id)
  coef_table <- coeftest(fit, vcov. = robust_vcov)
  ci <- coefci(fit, vcov. = robust_vcov)
} else {
  coef_table <- coeftest(fit)
  ci <- confint(fit)
}

# Extract OR + 95% CI
results <- data.frame(
  term = rownames(coef_table),
  or = exp(coef_table[, "Estimate"]),
  ci_lower = exp(ci[, 1]),
  ci_upper = exp(ci[, 2]),
  p_value = coef_table[, "Pr(>|z|)"]
)
```
