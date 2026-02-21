# CLIF Data Rules

## Mandatory Rules

1. **Always filter on `_category` columns, never `_name` columns.** Names are site-specific EHR strings. Categories are the CLIF controlled vocabulary. This is the #1 source of cross-site bugs.
2. **Always validate data before analysis.** Python: `clif.validate_all()` / R: verify schemas against TABLES.md or use CLIF-TableOne validation.
3. **Use ADT table for ICU times, not hospitalization table.** `hospitalization.admission_dttm` is hospital admission, not ICU admission. Filter `adt` where `location_category == 'icu'`.
4. **Log row counts at every cohort filter step.** CONSORT-style attrition tracking is required.
5. **Use encounter stitching for ICU stay identification.** A single ICU stay may span multiple hospitalization records.
6. **Distinguish continuous vs intermittent medications.** `medication_admin_continuous` = drips/infusions. `medication_admin_intermittent` = boluses/scheduled.
7. **All timestamps are UTC.** Timezone conversion is mandatory — Python: clifpy's `timezone` parameter / R: `lubridate::with_tz()`.
8. **When creating figures, follow JAMA style:** Arial font, minimum 8pt text, no overlap, legends outside plot area, sentence case axis labels.

## Coding Conventions

- **Python**: Use Polars for large-scale clinical data processing. Pandas is acceptable.
- **R**: Use tidyverse + arrow for data manipulation.
- **Datetime**: All datetimes must be timezone-aware UTC.
- **FiO2**: Always in decimal form (0.21-1.0), never percentage.
- **Outlier handling**: Apply thresholds from outlier reference files before analysis.
- **Naming**: Use `*_category` columns for analysis, `*_name` for QC only.
