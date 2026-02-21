---
name: marimo-batch
language: python
description: Prepare marimo notebooks for scheduled batch execution with Pydantic-based CLI/UI hybrid configuration.
---

# marimo Batch Execution

Prepare marimo notebooks for scheduled batch execution using Pydantic-based configuration.

## UI + CLI Hybrid Pattern

Define parameters with Pydantic models, then enable configuration through either an interactive marimo UI or command-line arguments. Use the UI to test and iterate, then use the CLI to run the batch job.

## Implementation

1. Declare a Pydantic `BaseModel` with typed fields and descriptions
2. Conditionally load values based on execution mode (script vs interactive)
3. Detect mode with `mo.app_meta().mode == "script"`

## Guidelines

- **Environment config**: Use `python-dotenv` and an `EnvConfig` class for API keys and credentials
- **W&B integration**: Optionally log `ModelParams` to Weights & Biases; include `wandb_project` and `wandb_run_name` if applicable
- **Column preservation**: When adding batch functionality, maintain existing column assignments
- **CLI convention**: Hyphens in CLI arguments convert to underscores for Python variable names

## Workflow

1. Identify which parameters should be configurable
2. Verify changes before implementation
3. Confirm W&B integration needs
4. Add environment variable handling where needed

## Complete CLIF Batch Example

### Pydantic Model

```python
from pydantic import BaseModel, Field
from typing import Optional
from datetime import date

class ClifBatchConfig(BaseModel):
    """Configuration for a CLIF batch analysis run."""
    site: str = Field("SITE_A", description="Site identifier")
    data_directory: str = Field("/data/clif", description="Path to CLIF parquet files")
    start_date: date = Field(date(2024, 1, 1), description="Cohort start date")
    end_date: date = Field(date(2024, 12, 31), description="Cohort end date")
    min_age: int = Field(18, ge=0, le=120, description="Minimum age at admission")
    max_age: Optional[int] = Field(None, ge=0, le=120, description="Maximum age (None = no limit)")
    lab_categories: list[str] = Field(
        default=["creatinine", "lactate", "bilirubin_total"],
        description="Lab categories to extract"
    )
    output_path: str = Field("output/", description="Directory for results")
    timezone: str = Field("US/Eastern", description="Site timezone for UTC conversion")
```

### Mode Detection and CLI Invocation

```python
import marimo as mo

if mo.app_meta().mode == "script":
    # CLI mode: python notebook.py -- --site SITE_A --start-date 2023-01-01
    import argparse
    parser = argparse.ArgumentParser(description="CLIF batch analysis")
    parser.add_argument("--site", default="SITE_A")
    parser.add_argument("--data-directory", default="/data/clif")
    parser.add_argument("--start-date", default="2024-01-01")
    parser.add_argument("--end-date", default="2024-12-31")
    parser.add_argument("--min-age", type=int, default=18)
    parser.add_argument("--max-age", type=int, default=None)
    parser.add_argument("--output-path", default="output/")
    parser.add_argument("--timezone", default="US/Eastern")
    args = parser.parse_args()
    config = ClifBatchConfig(
        site=args.site,
        data_directory=args.data_directory,
        start_date=args.start_date,
        end_date=args.end_date,
        min_age=args.min_age,
        max_age=args.max_age,
        output_path=args.output_path,
        timezone=args.timezone,
    )
else:
    # Interactive mode: use UI widgets with defaults
    config = ClifBatchConfig()
```

### CLI Usage

```bash
# Interactive development
marimo edit clif_analysis.py

# Read-only app deployment
marimo run clif_analysis.py

# Batch execution with arguments (note the -- separator)
python clif_analysis.py -- --site SITE_B --start-date 2023-06-01 --min-age 65

# Export to HTML report
marimo export html clif_analysis.py -o report.html -- --site SITE_A
```

**Note:** CLI argument names use hyphens (`--start-date`) which Pydantic/argparse auto-converts to underscores (`start_date`).
