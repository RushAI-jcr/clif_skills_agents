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
