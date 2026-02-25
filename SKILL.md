---
name: clif-icu
language: both
description: >
  CLIF (Common Longitudinal ICU data Format) assistant skill for critical care data science.
  Use this skill whenever a prompt mentions CLIF-related keywords such as: clif, clifpy, pyclif,
  hospitalization_id, vital_category, lab_category, med_category, device_category, mode_category,
  patient_assessments, medication_admin_continuous, medication_admin_intermittent, respiratory_support,
  crrt_therapy, mCIDE, SOFA score computation, ventilation episodes, prone positioning, dialysis,
  ECMO/MCS, microbiology_culture, sensitivity, ADT location_category, discharge_category,
  or any task involving ICU EHR data harmonization, federated critical care research,
  or CLIF schema/table questions. Also trigger when the user asks about CLIF table structures,
  permissible values, outlier thresholds, or ETL mapping for ICU data. Even if the user doesn't
  say "CLIF" explicitly, trigger if they're working with ICU data tables that match CLIF's
  entity-relationship model (e.g., joining vitals to respiratory_support via hospitalization_id).
---

# CLIF-ICU Skill

## Purpose

Provide accurate, schema-adherent guidance for working with the Common Longitudinal ICU data Format (CLIF). This includes table structures, permissible category values (mCIDE), outlier thresholds, ETL patterns, and analytic code generation.

## Core Principles

1. **Schema adherence**: Only reference variables and tables defined in CLIF. Default to `*_category` fields for analysis; preserve `*_name` fields for QC.
2. **Permissible values enforcement**: Use only validated mCIDE values from the reference files. Flag unlisted terms.
3. **Clinical accuracy**: Respect the clinical semantics behind CLIF tables (e.g., `device_category` hierarchy in respiratory_support, continuous vs intermittent medication separation).
4. **Federated model awareness**: No patient-level data sharing. Code should operate on local CLIF databases and return aggregate results.

## Quick Reference: CLIF Tables

### Beta Tables (stable, used in federated projects)
- **patient** – Demographics (race_category, ethnicity_category, sex_category)
- **hospitalization** – Encounter-level (admission/discharge times, discharge_category, geo codes)
- **adt** – Admission-Discharge-Transfer movements (location_category: ed, ward, icu, etc.)
- **vitals** – Longitudinal vital signs (vital_category: temp_c, heart_rate, sbp, dbp, spo2, etc.)
- **labs** – Longitudinal lab results (lab_category: 52 standardized labs with reference units)
- **respiratory_support** – Ventilator settings + observations (device_category, mode_category, tracheostomy flag)
- **medication_admin_continuous** – Continuous infusions (med_category, med_group: vasoactives, sedation, etc.)
- **position** – Prone/not_prone positioning
- **patient_assessments** – GCS, RASS, CAM-ICU, Braden, etc. (assessment_category, assessment_group)

### Concept Tables (in development)
- **medication_admin_intermittent** – Same schema as continuous, separate mCIDE list
- **medication_orders** – Order-level medication data
- **microbiology_culture** – Culture results (fluid_category, organism_category)
- **sensitivity** – Antibiotic susceptibility
- **dialysis / crrt_therapy** – Renal replacement therapy
- **ecmo_mcs** – Mechanical circulatory support
- **code_status** – DNR/DNI longitudinal tracking
- **procedures** – Bedside ICU procedures
- **transfusion** – Blood product administration
- **intake_output**, **invasive_hemodynamics**, **hospital_diagnosis**, **provider**

## Key Relationships

All clinical tables link to `hospitalization` via `hospitalization_id`. The `patient` table links via `patient_id`. The ADT table defines physical location over time.

## Reference Files

For detailed information, read the appropriate reference file:

| Need | Reference file |
|------|----------------|
| Full CLIF 2.1 schema with types and allowed values | `references/TABLES.md` |
| Common pitfalls with wrong/right code examples | `references/PITFALLS.md` |
| clifpy Python API (return schemas, error handling) | `references/PYTHON-API.md` |
| R API patterns (Arrow, DuckDB, tidyverse) | `references/R-API.md` |
| Lab categories + units + outlier thresholds | [mCIDE labs CSV](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF/tree/main/mCIDE/labs) |
| Continuous med categories | [mCIDE continuous CSV](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF/tree/main/mCIDE/medication_admin_continuous) |
| Intermittent med categories | [mCIDE intermittent CSV](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF/tree/main/mCIDE/medication_admin_intermittent) |
| Vital categories | [mCIDE vitals CSV](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF/tree/main/mCIDE/vitals) |
| Assessment categories | [mCIDE assessments CSV](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF/tree/main/mCIDE/patient_assessments) |
| Respiratory device + mode categories | [mCIDE respiratory CSV](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF/tree/main/mCIDE/respiratory_support) |
| ADT location types | [mCIDE ADT CSV](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF/tree/main/mCIDE/adt) |
| Microbiology organism + fluid categories | [mCIDE microbiology CSV](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF/tree/main/mCIDE/microbiology_culture) |
| Outlier thresholds (vitals, labs, resp, CRRT) | [mCIDE outlier-handling CSVs](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF/tree/main/mCIDE/outlier-handling) |

## Common Patterns

### Identifying ICU patients
```sql
SELECT h.hospitalization_id, h.patient_id, h.admission_dttm, h.discharge_dttm
FROM hospitalization h
JOIN adt a ON h.hospitalization_id = a.hospitalization_id
WHERE a.location_category = 'icu'
  AND a.in_dttm <= h.admission_dttm + INTERVAL '48 hours'
```

### Extracting first-24h lab values
```sql
SELECT l.hospitalization_id, l.lab_category,
       MIN(l.lab_value_numeric) AS min_val,
       MAX(l.lab_value_numeric) AS max_val,
       AVG(l.lab_value_numeric) AS mean_val
FROM labs l
JOIN adt a ON l.hospitalization_id = a.hospitalization_id
WHERE a.location_category = 'icu'
  AND l.lab_result_dttm BETWEEN a.in_dttm AND a.in_dttm + INTERVAL '24 hours'
  AND l.lab_category IN ('albumin', 'creatinine', 'lactate', 'wbc', 'hemoglobin')
GROUP BY l.hospitalization_id, l.lab_category
```

### Identifying mechanical ventilation episodes
Use `respiratory_support` table where `device_category = 'IMV'`. The `tracheostomy` flag distinguishes ETT-based vs trach-based ventilation. The `mode_category` field captures ventilation mode.

### Continuous medication dose extraction
```sql
SELECT hospitalization_id, admin_dttm, med_category, med_dose, med_dose_unit
FROM medication_admin_continuous
WHERE med_group = 'vasoactives'
  AND mar_action_group NOT IN ('stopped', 'missed')
ORDER BY hospitalization_id, admin_dttm
```

## Coding Conventions

- **Python**: Use Polars for large-scale clinical data processing. Pandas is acceptable.
- **SQL**: CLIF is language-agnostic; works as SQL DB or parquet flat files.
- **Datetime**: All datetimes must be timezone-aware UTC (`YYYY-MM-DD HH:MM:SS+00:00`).
- **FiO2**: Always in decimal form (0.21–1.0), never percentage.
- **Outlier handling**: Apply thresholds from the outlier reference files before analysis.

## Authoritative Sources

- CLIF GitHub: https://github.com/Common-Longitudinal-ICU-data-Format/CLIF
- CLIF Website: https://clif-icu.com
- Data Dictionary: https://clif-icu.com/data-dictionary
- mCIDE Explorer: https://clif-icu.com/mcide-explorer
- ETL Guide: https://clif-icu.com/etl-guide
