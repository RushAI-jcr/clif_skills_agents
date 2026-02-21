---
name: jupyter-notebook
language: python
description: Organizes, cleans, and optimizes Jupyter notebooks - removes empty cells, adds structure, extracts functions, generates documentation. Use when user asks to "clean notebook", "organize jupyter", "refactor notebook", or "jupyter best practices".
allowed-tools: [Read, Write, Bash]
---

# Jupyter Notebook Assistant

Cleans, organizes, and optimizes Jupyter notebooks for better readability and maintainability.

## When to Use

- "Clean up this Jupyter notebook"
- "Organize my notebook"
- "Refactor Jupyter code"
- "Extract functions from notebook"
- "Add structure to notebook"

## Instructions

### 1. Analyze Notebook

Read and parse .ipynb file:

```python
import json

with open('notebook.ipynb') as f:
    nb = json.load(f)

# Count cells
total_cells = len(nb['cells'])
code_cells = sum(1 for c in nb['cells'] if c['cell_type'] == 'code')
markdown_cells = sum(1 for c in nb['cells'] if c['cell_type'] == 'markdown')
empty_cells = sum(1 for c in nb['cells'] if not c['source'])

print(f"Total cells: {total_cells}")
print(f"Code cells: {code_cells}")
print(f"Markdown cells: {markdown_cells}")
print(f"Empty cells: {empty_cells}")
```

### 2. Common Cleanup Tasks

**Remove empty cells:**
```python
nb['cells'] = [c for c in nb['cells'] if c['source']]
```

**Clear outputs:**
```python
for cell in nb['cells']:
    if cell['cell_type'] == 'code':
        cell['outputs'] = []
        cell['execution_count'] = None
```

**Remove trailing whitespace:**
```python
for cell in nb['cells']:
    cell['source'] = [line.rstrip() + '\n' for line in cell['source']]
```

### 3. Add Structure

**Add section headers:**
```python
# Detect major sections and add markdown headers
sections = [
    "# Setup and Imports",
    "# Data Loading",
    "# Data Exploration",
    "# Data Preprocessing",
    "# Model Training",
    "# Evaluation",
    "# Visualization",
    "# Conclusion"
]

# Insert markdown cells at appropriate positions
```

**Add table of contents:**
```markdown
# Table of Contents

1. [Setup and Imports](#setup)
2. [Data Loading](#data)
3. [Data Exploration](#explore)
4. [Model Training](#train)
5. [Evaluation](#eval)
6. [Conclusions](#conclusion)
```

### 4. Extract Reusable Functions

**Identify repeated code patterns:**
```python
# Before: Repeated in multiple cells
df = pd.read_csv('data.csv')
df = df.dropna()
df = df[df['value'] > 0]

# After: Extract to function
def load_and_clean_data(filename):
    """Load CSV and apply standard cleaning."""
    df = pd.read_csv(filename)
    df = df.dropna()
    df = df[df['value'] > 0]
    return df

df = load_and_clean_data('data.csv')
```

**Create utils.py:**
```python
# utils.py - extracted helper functions
def plot_distribution(data, column, title=None):
    """Plot distribution of a column."""
    plt.figure(figsize=(10, 6))
    plt.hist(data[column], bins=50)
    plt.title(title or f'Distribution of {column}')
    plt.show()

def calculate_metrics(y_true, y_pred):
    """Calculate common ML metrics."""
    return {
        'accuracy': accuracy_score(y_true, y_pred),
        'precision': precision_score(y_true, y_pred),
        'recall': recall_score(y_true, y_pred),
        'f1': f1_score(y_true, y_pred)
    }
```

### 5. Generate requirements.txt

**Extract imports:**
```python
import re

imports = set()
for cell in nb['cells']:
    if cell['cell_type'] == 'code':
        for line in cell['source']:
            if line.startswith('import ') or line.startswith('from '):
                # Extract module name
                match = re.match(r'(?:from|import)\s+(\w+)', line)
                if match:
                    imports.add(match.group(1))

# Map to package names
package_mapping = {
    'sklearn': 'scikit-learn',
    'cv2': 'opencv-python',
    'PIL': 'Pillow'
}

with open('requirements.txt', 'w') as f:
    for imp in sorted(imports):
        pkg = package_mapping.get(imp, imp)
        f.write(f"{pkg}\n")
```

### 6. Add Documentation

**Add docstrings:**
```python
# Add markdown cell before major code sections
"""
## Data Preprocessing

This section handles:
- Missing value imputation
- Feature scaling
- Categorical encoding

Input: Raw DataFrame
Output: Preprocessed DataFrame ready for modeling
"""
```

**Document parameters:**
```python
# Parameter documentation cell
"""
### Hyperparameters

- `learning_rate`: 0.001 (tested 0.0001, 0.001, 0.01)
- `batch_size`: 32 (optimal for our dataset size)
- `epochs`: 100 (with early stopping)
- `dropout`: 0.5 (prevents overfitting)
"""
```

### 7. Best Practices

**Recommended structure:**
```
1. Title and Summary
2. Table of Contents
3. Setup (imports, config)
4. Data Loading
5. EDA (Exploratory Data Analysis)
6. Preprocessing
7. Model Definition
8. Training
9. Evaluation
10. Visualization
11. Conclusions
12. Next Steps
```

**Code style:**
- Use meaningful variable names
- Add comments for complex operations
- Keep cells focused (one task per cell)
- Separate imports from main code
- Clear outputs before committing

### 8. Optimization

**Memory management:**
```python
# Delete large intermediate dataframes
del large_df
import gc
gc.collect()

# Use chunking for large files
for chunk in pd.read_csv('large.csv', chunksize=10000):
    process(chunk)
```

**Reproducibility:**
```python
# Set random seeds at the top
import numpy as np
import random
import torch

SEED = 42
random.seed(SEED)
np.random.seed(SEED)
torch.manual_seed(SEED)
```

### 9. Convert to Script

**Extract to .py file:**
```python
# notebook_to_script.py
import nbformat
from nbconvert import PythonExporter

with open('notebook.ipynb') as f:
    nb = nbformat.read(f, as_version=4)

exporter = PythonExporter()
body, _ = exporter.from_notebook_node(nb)

with open('script.py', 'w') as f:
    f.write(body)
```

### 10. Automated Tools

**nbconvert:**
```bash
# Convert to HTML
jupyter nbconvert --to html notebook.ipynb

# Convert to Python script
jupyter nbconvert --to python notebook.ipynb

# Execute and save
jupyter nbconvert --to notebook --execute notebook.ipynb
```

**black (code formatter):**
```bash
pip install black[jupyter]

black notebook.ipynb
```

**nbqa (linting):**
```bash
pip install nbqa

nbqa flake8 notebook.ipynb
nbqa mypy notebook.ipynb
```

**jupytext (version control):**
```bash
pip install jupytext

# Convert to .py (for better git diffs)
jupytext --to py notebook.ipynb

# Sync .ipynb and .py
jupytext --set-formats ipynb,py notebook.ipynb
```

## Quick Checklist

- [ ] Remove empty cells
- [ ] Clear outputs
- [ ] Add section headers
- [ ] Add table of contents
- [ ] Extract reusable functions
- [ ] Generate requirements.txt
- [ ] Add documentation
- [ ] Set random seeds
- [ ] Format code
- [ ] Test notebook runs top-to-bottom

---

## CLIF Notebook Structure

For CLIF clinical data projects, follow this recommended cell order:

### 1. Config + Imports
```python
# Cell 1: Configuration
import json
import polars as pl
from clifpy import ClifOrchestrator

with open("clif_config.json") as f:
    config = json.load(f)

clif = ClifOrchestrator(
    data_directory=config["data_directory"],
    timezone=config["timezone"]
)
```

### 2. Data Loading + Validation
```python
# Cell 2: Validate and load
clif.validate_all()

hospitalization = clif.hospitalization.df
labs = clif.labs.df
vitals = clif.vitals.df
adt = clif.adt.df

print(f"Hospitalizations: {hospitalization.shape[0]:,}")
print(f"Labs: {labs.shape[0]:,}")
print(f"Vitals: {vitals.shape[0]:,}")
```

### 3. Cohort Definition (with attrition logging)

Every filter step must log the row count. This is mandatory for CONSORT-style reporting.

```python
# Cell 3: Cohort definition — log every step
cohort = hospitalization
print(f"All hospitalizations: {cohort.shape[0]:,}")

# Age filter
cohort = cohort.filter(pl.col("age_at_admission") >= 18)
print(f"After age >= 18: {cohort.shape[0]:,}")

# ICU patients only
icu_ids = adt.filter(pl.col("location_category") == "icu").select("hospitalization_id").unique()
cohort = cohort.join(icu_ids, on="hospitalization_id", how="semi")
print(f"After ICU filter: {cohort.shape[0]:,}")

# Date range
cohort = cohort.filter(pl.col("admission_dttm").is_between("2024-01-01", "2024-12-31"))
print(f"After date filter: {cohort.shape[0]:,}")
```

Add a markdown cell summarizing the attrition table:

```markdown
## Cohort Attrition

| Step | N |
|------|---|
| All hospitalizations | X,XXX |
| Age >= 18 | X,XXX |
| ICU admission | X,XXX |
| Date range 2024 | X,XXX |
```

### 4. Analysis
```python
# Cell 4+: Analysis using only _category columns (never _name)
labs_cohort = labs.join(cohort.select("hospitalization_id"), on="hospitalization_id", how="semi")
creatinine = labs_cohort.filter(pl.col("lab_category") == "creatinine")
```

### 5. Figures
```python
# Cell N: JAMA-style figures
# Arial font, 8pt minimum, legends outside plot area
import matplotlib.pyplot as plt
plt.rcParams.update({"font.family": "Arial", "font.size": 9})
```

### CLIF-Specific Best Practices

- **Always validate before analysis**: `clif.validate_all()` must be the first operation after loading
- **Log attrition in markdown cells**: Every inclusion/exclusion step gets a markdown cell summarizing the count
- **Filter on `_category`, never `_name`**: Names are site-specific EHR strings
- **Use ADT for ICU times**: `hospitalization.admission_dttm` is hospital admission, not ICU admission
- **Apply outlier thresholds**: Use CLIF reference ranges before any aggregation
- **Clear outputs before committing**: Never commit cell outputs containing patient data
