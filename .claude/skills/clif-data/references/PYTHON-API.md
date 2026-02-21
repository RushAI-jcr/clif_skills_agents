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

```python
# SOFA scores per hospitalization per hour
sofa_df = clif.compute_sofa_scores()

# Hourly wide format (one row per hospitalization per hour)
wide_df = clif.create_wide_dataset()

# Merge split ICU stays (gap_hours = max gap to consider same stay)
stitched = clif.encounter_stitching(gap_hours=24)
```

## Outlier Handling
clifpy includes built-in outlier detection for vitals and labs based on clinically plausible ranges. Check `clif.validate_all()` output for flagged values.

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
