# R Patterns for CLIF Data

## Key R Packages

| Package | Purpose |
|---|---|
| `arrow` | Read Parquet files, lazy evaluation with `open_dataset()` |
| `dplyr` | Data manipulation (filter, mutate, join, summarize) |
| `data.table` | High-performance alternative to dplyr for large datasets |
| `duckdb` | SQL engine for datasets too large for memory |
| `lubridate` | Timezone conversion and datetime manipulation |
| `yaml` | Read `config_template.yaml` |
| `tidyverse` | Meta-package (dplyr, tidyr, ggplot2, stringr, etc.) |
| `table1` | Table 1 demographics summaries |
| `metafor` | Meta-analysis for federated results |
| `brms` | Bayesian regression models |

## Configuration

R CLIF projects use a YAML config file:

```yaml
# config/config_template.yaml
site: "SITE_A"
file_type: "parquet"
clif_data_path: "/path/to/clif/data"
site_time_zone: "US/Eastern"
```

Load in R:
```r
library(yaml)
config <- read_yaml("config/config_template.yaml")
```

## File Naming Convention

CLIF data files follow the pattern: `{site}_clif_{tablename}.{filetype}`

Example: `SITE_A_clif_vitals.parquet`

## Generic Load Function

```r
load_clif_table <- function(table_name, config) {
  file_path <- file.path(
    config$clif_data_path,
    paste0(config$site, "_clif_", table_name, ".", config$file_type)
  )
  arrow::read_parquet(file_path)
}

# Usage
patient <- load_clif_table("patient", config)
hospitalization <- load_clif_table("hospitalization", config)
adt <- load_clif_table("adt", config)
vitals <- load_clif_table("vitals", config)
labs <- load_clif_table("labs", config)
respiratory_support <- load_clif_table("respiratory_support", config)
medication_admin_continuous <- load_clif_table("medication_admin_continuous", config)
medication_admin_intermittent <- load_clif_table("medication_admin_intermittent", config)
```

## Lazy Evaluation with Arrow

For large datasets, use `open_dataset()` to filter before loading into memory:

```r
labs <- arrow::open_dataset(
  file.path(config$clif_data_path, paste0(config$site, "_clif_labs.", config$file_type))
) %>%
  filter(lab_category == "creatinine") %>%
  collect()
```

## Filtering on _category Columns

```r
# Vitals
hr <- vitals %>% filter(vital_category == "heart_rate")

# Labs
creatinine <- labs %>% filter(lab_category == "creatinine")

# Respiratory support
imv <- respiratory_support %>% filter(device_category == "IMV")

# Medications
norepi <- medication_admin_continuous %>%
  filter(med_category == "norepinephrine", mar_action_group == "administered")
```

## Timezone Conversion

```r
library(lubridate)

# Convert UTC timestamps to local time
adt <- adt %>%
  mutate(
    in_dttm_local = with_tz(in_dttm, tzone = config$site_time_zone),
    out_dttm_local = with_tz(out_dttm, tzone = config$site_time_zone)
  )
```

## Type Casting Helper

Arrow may read string columns as `large_utf8`. Cast to `utf8` for compatibility:

```r
cast_large_utf8_to_utf8 <- function(df) {
  schema <- df$schema
  for (i in seq_along(schema)) {
    if (schema[[i]]$type$ToString() == "large_utf8") {
      df[[schema[[i]]$name]] <- df[[schema[[i]]$name]]$cast(arrow::utf8())
    }
  }
  df
}
```

## DuckDB for Large Data

When datasets exceed available memory:

```r
library(duckdb)

con <- dbConnect(duckdb::duckdb())

# Register Arrow table with DuckDB
labs_arrow <- arrow::open_dataset(labs_path)
duckdb::duckdb_register_arrow(con, "labs", labs_arrow)

# Query with SQL
result <- dbGetQuery(con, "
  SELECT hospitalization_id, lab_category, AVG(lab_value_numeric) as mean_value
  FROM labs
  WHERE lab_category IN ('creatinine', 'lactate', 'bilirubin_total')
  GROUP BY hospitalization_id, lab_category
")

dbDisconnect(con)
```

## No clifpy Equivalent in R

R does not have a clifpy package. The following require custom implementation or CLIF-TableOne:

- **Validation**: verify column names/types against TABLES.md manually or use CLIF-TableOne
- **SOFA scores**: implement scoring logic per SOFA component (see TABLES.md for source columns)
- **Encounter stitching**: group sequential ICU ADT records with gaps < threshold into single stays
- **Wide dataset**: pivot and join tables to create hourly wide format

CLIF-TableOne (`run_project.R`) provides Table 1 generation, CONSORT diagrams, and ECDF plots for R projects.
