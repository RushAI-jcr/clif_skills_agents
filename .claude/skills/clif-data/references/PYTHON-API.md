# clifpy Python API Reference

## Installation
```bash
pip install clifpy
# or with uv:
uv pip install clifpy
```

## ClifOrchestrator

```python
from clifpy import ClifOrchestrator

clif = ClifOrchestrator(
    data_directory='/path/to/clif/data',   # directory containing clif_*.parquet files
    timezone='US/Eastern'                   # site-specific timezone for conversion
)
```

## Validation
```python
clif.validate_all()  # ALWAYS run first — checks schema compliance for all loaded tables
```

## Table Accessors
Each returns a lazy-loaded Polars DataFrame via `.df`:

| Accessor | Table |
|---|---|
| `clif.patient.df` | Demographics |
| `clif.hospitalization.df` | Encounter spine |
| `clif.adt.df` | Location timeline (ADT) |
| `clif.vitals.df` | Time-series vitals |
| `clif.labs.df` | Time-series labs |
| `clif.respiratory_support.df` | Ventilator data |
| `clif.medication_admin_continuous.df` | Drips/infusions |
| `clif.medication_admin_intermittent.df` | Boluses/scheduled |
| `clif.patient_assessments.df` | GCS, RASS, pain |
| `clif.hospital_diagnosis.df` | Billing diagnoses |
| `clif.microbiology_culture.df` | Culture results |
| `clif.position.df` | Prone positioning |
| `clif.crrt_therapy.df` | Dialysis |
| `clif.code_status.df` | Goals of care |
| `clif.patient_procedures.df` | Billing procedures |

## Clinical Transformations

### compute_sofa_scores()

```python
sofa_df = clif.compute_sofa_scores()
```

**Returns a Polars DataFrame with columns:**

| Column | Type | Description |
|---|---|---|
| hospitalization_id | VARCHAR | Encounter identifier |
| hour | INT | Hour offset from ICU admission |
| sofa_respiration | INT | PaO2/FiO2 component (0–4) |
| sofa_coagulation | INT | Platelet component (0–4) |
| sofa_liver | INT | Bilirubin component (0–4) |
| sofa_cardiovascular | INT | MAP + vasopressor component (0–4) |
| sofa_cns | INT | GCS component (0–4) |
| sofa_renal | INT | Creatinine component (0–4) |
| sofa_total | INT | Sum of all components (0–24) |

### create_wide_dataset()

```python
wide_df = clif.create_wide_dataset()
```

**Returns a Polars DataFrame with columns:**

| Column | Type | Description |
|---|---|---|
| hospitalization_id | VARCHAR | Encounter identifier |
| hour | INT | Hour offset from ICU admission |
| temp_c | FLOAT | Most recent temperature |
| heart_rate | FLOAT | Most recent heart rate |
| sbp | FLOAT | Most recent systolic BP |
| dbp | FLOAT | Most recent diastolic BP |
| spo2 | FLOAT | Most recent SpO2 |
| respiratory_rate | FLOAT | Most recent resp rate |
| map | FLOAT | Most recent MAP |
| (one column per lab_category) | FLOAT | Most recent lab value |
| fio2_set | FLOAT | Most recent FiO2 |
| peep_set | FLOAT | Most recent PEEP |
| device_category | VARCHAR | Current respiratory device |
| gcs_total | FLOAT | Most recent GCS |
| rass | FLOAT | Most recent RASS |
| (vasopressor dose columns) | FLOAT | Current infusion rates |

### encounter_stitching()

```python
stitched = clif.encounter_stitching(gap_hours=24)
```

Merges sequential hospitalization records with ICU gaps < `gap_hours` into a single continuous ICU stay. Returns a DataFrame with a `stitched_id` column.

## Outlier Handling

clifpy includes built-in outlier detection for vitals, labs, and respiratory support based on clinically plausible ranges. Check `clif.validate_all()` output for flagged values.

## Error Handling

clifpy raises standard Python exceptions:

| Exception | When |
|---|---|
| `FileNotFoundError` | `data_directory` doesn't exist or expected parquet files are missing |
| `ValueError` | Invalid `timezone` string, or data fails schema validation |
| `KeyError` | Accessing a table that doesn't exist in the data directory |
| `polars.exceptions.SchemaError` | Column types don't match expected CLIF schema |

**Pattern for safe initialization:**
```python
from clifpy import ClifOrchestrator

try:
    clif = ClifOrchestrator(
        data_directory=config["data_directory"],
        timezone=config["timezone"]
    )
    clif.validate_all()
except FileNotFoundError as e:
    print(f"Data not found: {e}")
except ValueError as e:
    print(f"Validation failed: {e}")
```

## Memory Considerations

- **Lazy loading**: Table accessors (`.df`) use Polars lazy frames — data isn't loaded until you call `.collect()` or run an operation
- **Large datasets** (>10M rows): Use `scan_parquet` with predicate pushdown. Filter on `hospitalization_id` or `lab_category` before collecting
- **Wide dataset**: `create_wide_dataset()` can be memory-intensive for large cohorts. Filter to your cohort first, then call it
- **SOFA scores**: `compute_sofa_scores()` joins multiple tables internally. For very large sites (>100k hospitalizations), consider computing in batches

```python
# Memory-efficient pattern for large sites
cohort_ids = clif.hospitalization.df.filter(
    pl.col("age_at_admission") >= 18
).select("hospitalization_id").collect()

# Filter tables to cohort before computing
labs_cohort = clif.labs.df.filter(
    pl.col("hospitalization_id").is_in(cohort_ids["hospitalization_id"])
)
```

## Configuration: `clif_config.json`
```json
{
  "data_directory": "/path/to/clif/data",
  "timezone": "US/Eastern",
  "site_name": "SITE_A"
}
```

Load config in code:
```python
import json
with open("clif_config.json") as f:
    config = json.load(f)
clif = ClifOrchestrator(
    data_directory=config["data_directory"],
    timezone=config["timezone"]
)
```

## Package Manager
- **uv** (recommended): `uv pip install clifpy polars duckdb marimo`
- **pip**: `pip install clifpy polars duckdb marimo`

## Key Dependencies
- `polars` — DataFrame library (used internally by clifpy)
- `duckdb` — SQL engine for large datasets
- `marimo` — reactive notebook environment
