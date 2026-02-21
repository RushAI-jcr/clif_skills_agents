---
name: clif-data
language: both
description: CLIF data schema rules, _name/_category conventions, table structures, and validation patterns. Use for any CLIF data question regardless of language.
---

# CLIF Data Analysis

## What is CLIF?
The Common Longitudinal ICU Format — a standardized schema for critical care data enabling federated multi-site research. Current version: 2.1.0.

## The _name / _category Rule (CRITICAL)
Every clinical concept has two columns:
- `lab_name`, `vital_name`, `med_name` = raw EHR string (site-specific, **NEVER filter on these**)
- `lab_category`, `vital_category`, `med_category` = CLIF controlled vocabulary (**ALWAYS filter on these**)

This is the most common mistake in CLIF code. Cross-site portability depends on using `_category` columns exclusively.

## Loading CLIF Data

### Python (clifpy + Polars)
```python
from clifpy import ClifOrchestrator

clif = ClifOrchestrator(
    data_directory='/path/to/clif/data',
    timezone='US/Eastern'
)
clif.validate_all()

# Table access (lazy-loaded Polars DataFrames)
clif.patient.df
clif.vitals.df
clif.labs.df
```
See `references/PYTHON-API.md` for full clifpy API reference.

### R (arrow + dplyr)
```r
library(arrow)
library(dplyr)
library(yaml)

config <- read_yaml("config/config_template.yaml")

load_clif_table <- function(table_name, config) {
  file_path <- file.path(
    config$clif_data_path,
    paste0(config$site, "_clif_", table_name, ".", config$file_type)
  )
  arrow::read_parquet(file_path)
}

patient <- load_clif_table("patient", config)
vitals  <- load_clif_table("vitals", config)
labs    <- load_clif_table("labs", config)
```
See `references/R-API.md` for full R patterns reference.

## Configuration

### Python: `clif_config.json`
```json
{
  "data_directory": "/path/to/clif/data",
  "timezone": "US/Eastern",
  "site_name": "SITE_A"
}
```

### R: `config/config_template.yaml`
```yaml
site: "SITE_A"
file_type: "parquet"
clif_data_path: "/path/to/clif/data"
site_time_zone: "US/Eastern"
```

## Key Join Pattern
- `hospitalization_id` — primary join key across ALL clinical tables
- `patient_id` — links hospitalizations to demographics
- `hospitalization_joined_id` — links split ICU stays after encounter stitching

## ICU Stay Identification
Do NOT use `admission_dttm`/`discharge_dttm` from hospitalization as ICU time. Use ADT table:

**Python:**
```python
icu_stays = clif.adt.df.filter(pl.col("location_category") == "icu")
```

**R:**
```r
icu_stays <- adt %>% filter(location_category == "icu")
```

## Mechanical Ventilation Cohorts

**Python:**
```python
imv = clif.respiratory_support.df.filter(pl.col("device_category") == "IMV")
```

**R:**
```r
imv <- respiratory_support %>% filter(device_category == "IMV")
```

## Medication Analysis
- `medication_admin_continuous` = drips and infusions (time-series dose records)
- `medication_admin_intermittent` = boluses and scheduled doses (event records)
- Filter `mar_action_group == 'administered'` for actual infusion periods

## Timestamp Handling
All CLIF timestamps are DATETIME. Timezone conversion is mandatory:
- **Python**: clifpy handles this via the `timezone` parameter
- **R**: use `lubridate::with_tz(dt_column, tzone = config$site_time_zone)`

Never assume local time.

## Data Quality Workflow
1. **Python**: `clif.validate_all()` — schema compliance
2. **R**: verify schemas against TABLES.md or use CLIF-TableOne
3. CLIF-TableOne `run_project.py` (Python) or `run_project.R` — statistical plausibility + CONSORT diagram
4. CLIF-Lighthouse — interactive visual QC dashboard

## Cohort Building Pattern
1. Start from `hospitalization` table
2. Filter age: `age_at_admission >= 18`
3. Filter ICU: join to `adt` where `location_category == 'icu'`
4. Apply clinical criteria (diagnosis codes, device_category, lab_category, etc.)
5. Apply encounter stitching for linked stays
6. **Log row counts at every filter step** (CONSORT-style)

## Federated Analysis Structure
```
project/
├── config/
│   ├── clif_config.json         # Python config
│   └── config_template.yaml     # R config
├── scripts/                     # identical across all sites
├── output/
│   ├── intermediate/            # local only (patient-level)
│   └── final/                   # shared (aggregated only)
├── requirements.txt             # Python dependencies
└── renv.lock                    # R dependencies
```
No patient-level data leaves the site. Only aggregated outputs are shared.
