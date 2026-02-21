# CLIF Common Pitfalls

## 1. Filtering on _name instead of _category

**Python — Wrong:** `labs.filter(pl.col("lab_name") == "Hemoglobin")`
**Python — Right:** `labs.filter(pl.col("lab_category") == "hemoglobin")`

**R — Wrong:** `labs %>% filter(lab_name == "Hemoglobin")`
**R — Right:** `labs %>% filter(lab_category == "hemoglobin")`

Names vary across sites and EHR systems. Categories are the CLIF controlled vocabulary.

## 2. Using hospitalization times as ICU times

**Wrong:** `hospitalization.admission_dttm` as ICU start
**Right:** Filter `adt` where `location_category == 'icu'` and use `in_dttm` as ICU start

The hospitalization table tracks hospital admission, not ICU admission.

## 3. Not handling encounter stitching

A single "ICU stay" may appear as multiple hospitalizations if a patient is briefly transferred to step-down.

**Python:** `clif.encounter_stitching(gap_hours=24)`
**R:** Custom logic required — group sequential ICU ADT records with gaps < 24 hours into a single stay. No clifpy equivalent exists in R.

## 4. Ignoring timezone conversion

Raw data from some sites stores local time.

**Python:** clifpy handles this via the `timezone` parameter.
**R:** Use `lubridate::with_tz(dt_column, tzone = config$site_time_zone)`. Always convert before computing durations.

## 5. Treating respiratory_support as continuous

A patient may have no respiratory_support record for several hours while still on the ventilator. Handle gaps in device records explicitly when computing ventilation duration.

## 6. Confusing continuous vs intermittent medications

- `medication_admin_continuous` = drips, infusions (time-series with dose rates)
- `medication_admin_intermittent` = boluses, scheduled doses (single events)

Filter `mar_action_group == 'administered'` for actual administration, not just orders.

## 7. CLIF version mismatch

Column names and controlled vocabularies changed between CLIF 1.0, 2.0, and 2.1. Always check which version your data conforms to. clifpy currently targets 2.0; CLIF-TableOne targets 2.1.

## 8. Not logging cohort attrition

Every inclusion/exclusion step should log a row count. This is required for CONSORT-style reporting and catches data issues early.

**Python:**
```python
print(f"After age filter: {df.shape[0]:,} rows")
```

**R:**
```r
message(sprintf("After age filter: %s rows", format(nrow(df), big.mark = ",")))
```

## 9. Not validating data first

**Python:** Always call `clif.validate_all()` immediately after loading data.
**R:** Verify column names and types against TABLES.md, or run CLIF-TableOne validation.

## 10. Arrow large_utf8 type casting (R-specific)

Arrow may read string columns as `large_utf8` instead of `utf8`, causing type mismatches in joins and filters.

**Fix:**
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

## 11. Collecting too early in Arrow pipelines (R-specific)

Calling `collect()` before filtering brings the entire dataset into memory.

**Wrong:**
```r
labs <- read_parquet(path) %>% collect() %>% filter(lab_category == "creatinine")
```

**Right:**
```r
labs <- open_dataset(path) %>% filter(lab_category == "creatinine") %>% collect()
```

Use `arrow::open_dataset()` for lazy evaluation — filter and select before `collect()`.

## 12. renv reproducibility (R-specific)

Always commit `renv.lock` to version control. When setting up at a new site, run `renv::restore()` before any analysis. Pin package versions to avoid surprises.

## 13. FiO2 unit confusion

FiO2 in CLIF is always stored as a **decimal** (0.21–1.0), never as a percentage (21–100). Some source EHR systems store it as a percentage. Failing to normalize causes PaO2/FiO2 ratios to be off by 100x.

**Python — Wrong:**
```python
# Assumes FiO2 is already decimal — but source data might be 21–100
pf_ratio = labs_pao2 / resp["fio2_set"]
```

**Python — Right:**
```python
# Normalize FiO2 to decimal if values > 1.0 exist
resp = resp.with_columns(
    pl.when(pl.col("fio2_set") > 1.0)
    .then(pl.col("fio2_set") / 100)
    .otherwise(pl.col("fio2_set"))
    .alias("fio2_set")
)
# Then apply outlier thresholds
resp = resp.filter(pl.col("fio2_set").is_between(0.21, 1.0))
```

**R — Wrong:**
```r
pf_ratio <- labs_pao2$lab_value_numeric / resp$fio2_set
```

**R — Right:**
```r
resp <- resp |>
  mutate(fio2_set = if_else(fio2_set > 1.0, fio2_set / 100, fio2_set)) |>
  filter(fio2_set >= 0.21, fio2_set <= 1.0)
```

## 14. Duplicate rows from many-to-many joins

Joining labs to vitals (or any two time-series tables) on `hospitalization_id` alone creates a Cartesian product — one row per lab × vital combination.

**Python — Wrong:**
```python
# Produces N_labs × N_vitals rows per hospitalization
combined = labs.join(vitals, on="hospitalization_id")
```

**Python — Right:**
```python
# Aggregate to one row per hospitalization before joining
labs_agg = labs.group_by("hospitalization_id").agg(
    pl.col("lab_value_numeric").first().alias("first_lab_value")
)
vitals_agg = vitals.group_by("hospitalization_id").agg(
    pl.col("vital_value").first().alias("first_vital_value")
)
combined = labs_agg.join(vitals_agg, on="hospitalization_id")
```

**R — Wrong:**
```r
combined <- labs |> inner_join(vitals, by = "hospitalization_id")
```

**R — Right:**
```r
labs_agg <- labs |>
  summarise(first_lab_value = first(lab_value_numeric), .by = hospitalization_id)
vitals_agg <- vitals |>
  summarise(first_vital_value = first(vital_value), .by = hospitalization_id)
combined <- labs_agg |> inner_join(vitals_agg, by = "hospitalization_id")
```

Alternatively, use time-window joins (e.g., asof join in Polars, `rolling_join` in data.table) to match the closest-in-time measurement.

## 15. Missing data patterns across sites

Different CLIF sites represent missing data differently: `NaN`, `NULL`, empty string `""`, or the value simply absent from the table. Always handle all three before aggregation.

**Python — Wrong:**
```python
mean_val = labs.select(pl.col("lab_value_numeric").mean())  # NaN poisons the mean
```

**Python — Right:**
```python
# Drop all missing representations before aggregation
labs_clean = labs.filter(
    pl.col("lab_value_numeric").is_not_null()
    & pl.col("lab_value_numeric").is_not_nan()
)
mean_val = labs_clean.select(pl.col("lab_value_numeric").mean())
```

**R — Wrong:**
```r
mean(labs$lab_value_numeric)  # Returns NA if any NAs present
```

**R — Right:**
```r
labs_clean <- labs |>
  filter(!is.na(lab_value_numeric), lab_value_numeric != "")
mean(labs_clean$lab_value_numeric, na.rm = TRUE)
```

Also check for sentinel values (e.g., -999, 9999) that some sites use instead of NULL.

## 16. Outlier values not filtered before aggregation

CLIF provides clinically validated outlier thresholds for vitals, labs, and respiratory support parameters. Failure to apply them before aggregation causes extreme values to skew means, medians, and derived scores.

**Python — Wrong:**
```python
# Includes biologically implausible values (e.g., heart_rate = 999)
avg_hr = vitals.filter(
    pl.col("vital_category") == "heart_rate"
).select(pl.col("vital_value").mean())
```

**Python — Right:**
```python
# Apply CLIF outlier thresholds (heart_rate: 0–300 bpm)
avg_hr = vitals.filter(
    (pl.col("vital_category") == "heart_rate")
    & pl.col("vital_value").is_between(0, 300)
).select(pl.col("vital_value").mean())
```

**R — Wrong:**
```r
vitals |> filter(vital_category == "heart_rate") |> summarise(mean_hr = mean(vital_value))
```

**R — Right:**
```r
vitals |>
  filter(vital_category == "heart_rate", vital_value >= 0, vital_value <= 300) |>
  summarise(mean_hr = mean(vital_value, na.rm = TRUE))
```

Reference thresholds are documented in TABLES.md and available in the mCIDE outlier-handling CSVs at https://github.com/Common-Longitudinal-ICU-data-Format/CLIF/tree/main/mCIDE/outlier-handling.
