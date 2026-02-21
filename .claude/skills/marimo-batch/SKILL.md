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
