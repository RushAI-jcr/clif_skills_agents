# marimo UI Components

## Interactive CLIF Filter Panel

Build a complete filter panel for exploring CLIF data with dropdowns, date ranges, and interactive tables:

```python
import marimo as mo
import polars as pl

lab_picker = mo.ui.dropdown(
    options=["creatinine", "lactate", "hemoglobin", "platelet_count",
             "bilirubin_total", "sodium", "potassium", "wbc", "albumin"],
    value="creatinine",
    label="Lab category"
)
date_range = mo.ui.date_range(
    start="2023-01-01",
    stop="2025-12-31",
    value=("2024-01-01", "2024-06-30"),
    label="Collection window"
)
site_picker = mo.ui.dropdown(
    options=["All", "SITE_A", "SITE_B", "SITE_C"],
    value="All",
    label="Site"
)
mo.hstack([lab_picker, date_range, site_picker], gap=1.0)
```

```python
# Reactive cell — re-runs when any filter changes
start, end = date_range.value
filtered = labs.filter(
    (pl.col("lab_category") == lab_picker.value)
    & (pl.col("lab_collect_dttm").cast(pl.Date) >= start)
    & (pl.col("lab_collect_dttm").cast(pl.Date) <= end)
)
mo.ui.table(filtered, selection="multi", page_size=25, label="Filtered results")
```

## Input Controls

```python
# Text and number
name = mo.ui.text(placeholder="Patient ID...", label="Search")
threshold = mo.ui.number(start=0, stop=100, value=50, step=1, label="Threshold")

# Slider
cutoff = mo.ui.slider(start=0, stop=300, value=100, step=10, label="Heart rate cutoff")

# Checkbox and radio
include_outliers = mo.ui.checkbox(label="Include outliers")
method = mo.ui.radio(options=["mean", "median", "max"], value="mean", label="Aggregation")

# Dropdown (dict form — display key differs from value)
device = mo.ui.dropdown(
    options={"Invasive Vent": "IMV", "BiPAP": "NIPPV", "High Flow": "High Flow NC"},
    value="IMV",
    label="Device"
)
# device.value -> "IMV"
# device.selected_key -> "Invasive Vent"

# Searchable dropdown (useful for large category lists)
organism = mo.ui.dropdown(
    options=["escherichia_coli", "klebsiella_pneumoniae", "staphylococcus_aureus", "pseudomonas_aeruginosa"],
    searchable=True,
    allow_select_none=True,
    label="Organism"
)

# Date and datetime
admit_date = mo.ui.date(start="2020-01-01", stop="2025-12-31", label="Admission date")
event_time = mo.ui.datetime(precision="minute", label="Event timestamp")
```

## Form Composition

Wrap multiple UI elements in a form that submits all values at once:

```python
query_form = (
    mo.md("""
    **Cohort Query**

    Lab: {lab}
    Min value: {min_val}
    Max value: {max_val}
    Date range: {start} to {end}
    """)
    .batch(
        lab=mo.ui.dropdown(["creatinine", "lactate", "hemoglobin"], label="Lab"),
        min_val=mo.ui.number(start=0, stop=100, value=0, label="Min"),
        max_val=mo.ui.number(start=0, stop=100, value=20, label="Max"),
        start=mo.ui.date(label="Start"),
        end=mo.ui.date(label="End"),
    )
    .form(
        submit_button_label="Run Query",
        show_clear_button=True,
        clear_on_submit=False,
    )
)
query_form
```

```python
# Access values after submission (None until first submit)
mo.stop(query_form.value is None, mo.md("*Submit the form above to see results.*"))
params = query_form.value
# params = {"lab": "creatinine", "min_val": 0, "max_val": 20, "start": date(...), "end": date(...)}
```

Single-element form:
```python
sample_size = mo.ui.slider(1, 1000, value=100, label="Sample size").form(
    submit_button_label="Apply",
    validate=lambda val: None if val >= 10 else "Must be >= 10",
)
```

## Data Display

```python
# Interactive table with formatting and selection
table = mo.ui.table(
    data=df,
    selection="multi",
    pagination=True,
    page_size=20,
    show_column_summaries=True,
    show_data_types=True,
    show_download=True,
    format_mapping={"lab_value_numeric": "{:.2f}".format},
    label="Lab results"
)

# Data explorer for ad-hoc exploration
mo.ui.data_explorer(df)

# Altair chart integration
import altair as alt
chart = alt.Chart(df.to_pandas()).mark_point().encode(x="x:Q", y="y:Q")
mo.ui.altair_chart(chart)
```

## Dashboard Layout

```python
# Horizontal stack
controls = mo.hstack(
    [lab_picker, date_range, site_picker],
    justify="start",    # start, center, end, space-between, space-around
    align="center",     # start, end, center, stretch
    gap=1.0,            # rem units
    widths="equal",     # "equal" | [0.3, 0.5, 0.2] relative weights
)

# Vertical stack
sidebar = mo.vstack(
    [mo.md("## Filters"), lab_picker, site_picker, date_range],
    align="stretch",
    gap=0.5,
)

# Nested grid layout
dashboard = mo.vstack([
    mo.hstack([mo.md("# CLIF Dashboard"), mo.ui.switch(label="Dark mode")], justify="space-between"),
    mo.hstack([lab_picker, date_range, site_picker, mo.ui.button("Run", kind="success")], gap=1.0),
    mo.hstack([summary_card_1, summary_card_2, summary_card_3], widths="equal"),
    mo.hstack([chart_panel, table_panel], widths=[0.4, 0.6]),
])

# Tabs
tabs = mo.ui.tabs({
    "Overview": overview_content,
    "Labs": labs_table,
    "Vitals": vitals_chart,
    "Export": export_panel,
})
```

## Custom Components

Implement `_display_` for custom rendering:

```python
class CohortSummary:
    def __init__(self, df: pl.DataFrame):
        self.n = df.shape[0]
        self.n_patients = df["patient_id"].n_unique()

    def _display_(self):
        return mo.md(f"**Cohort:** {self.n:,} rows, {self.n_patients:,} patients")
```

## External Widget Libraries

- `drawdata.ScatterWidget` — draw points, get coordinates as DataFrame
- `anywidget` subclasses — custom JS/CSS components (wrap with `mo.ui.anywidget()`)
