---
name: scientific-figures
language: both
description: JAMA/Nature/NEJM publication-quality figures with colorblind-safe palettes, anti-overlap, and standardized axis labels. Covers matplotlib (Python) and ggplot2 (R).
---

# Scientific Figures

## Rules (Always Enforced)

- JAMA style: Arial/Helvetica, minimum 8pt text, no exceptions
- No text overlap — titles, legends, labels, annotations must not collide
- Legends outside plot area (below or beside) unless empty whitespace exists
- Axis labels: sentence case with units in parentheses
- Colorblind-safe palette (Okabe-Ito / Wong, Nature Methods 2011)
- Export: PDF (vector), TIFF (600 dpi LZW), PNG (300 dpi)

## Python (matplotlib)

```python
import sciplot

sciplot.apply()                        # Apply globally
fig, axes = sciplot.figure(1, 2)       # Journal-sized figure
sciplot.fix_overlap(fig)               # Auto-fix overlaps
sciplot.save(fig, "figure1")           # Multi-format export
```

- **Colors**: `TX_COLOR`, `CTRL_COLOR`, `COLOR_CYCLE`, `BAR_FILLS`, `BAR_EDGES`
- **Axis labels**: `xlabel(ax, "egfr")`, `ylabel(ax, "survival")` — keys from `AXIS_LABELS` dict or custom text
- **P-values**: `format_pvalue(p)`, `add_significance(ax, x1, x2, y, p)`
- **Plot helpers**: `km_style(ax)`, `forest_style(ax)`, `panel_label(ax, "A")`
- **Legend**: `safe_legend(ax, outside=True)` for guaranteed no-overlap
- Full reference: `references/matplotlib.md`

## R (ggplot2)

```r
source("sciplot.R")

ggplot(df, aes(x, y)) +
  geom_point() +
  theme_sci() +
  scale_color_sci() +
  labs(x = sci_lab("time_days"), y = sci_lab("survival"))

save_sci(p, "figure1")
```

- **Colors**: `tx_color`, `ctrl_color`, `okabe_ito`, `bar_fills`
- **Scales**: `scale_color_sci()`, `scale_fill_sci()`, `scale_color_tx_ctrl()`
- **Axis labels**: `labs(x = sci_lab("key"), y = sci_lab("key"))`
- **P-values**: `format_pvalue(p)`
- **Anti-overlap**: `rotate_x()`, `legend_bottom()`, `scale_x_clean()`, `scale_y_clean()`
- **Multi-panel**: patchwork with `plot_annotation(tag_levels = "A")` (JAMA) or `"a"` (Nature)
- Full reference: `references/ggplot2.md`

## Shared Axis Label Keys

Use standardized keys for consistent labeling across Python and R. Full dictionary in `references/axis_labels.md`.

| Key | Output |
|-----|--------|
| `time_days` | Time (days) |
| `survival` | Survival Probability |
| `hr` | Hazard Ratio (95% CI) |
| `or` | Odds Ratio (95% CI) |
| `creatinine` | Serum Creatinine (mg/dL) |
| `egfr` | eGFR (mL/min/1.73 m²) |
| `map` | Mean Arterial Pressure (mm Hg) |
| `sofa` | SOFA Score |
| `spo2` | SpO₂ (%) |
| `pf_ratio` | PaO₂/FiO₂ Ratio |
| `icu_los` | ICU Length of Stay (days) |
| `sensitivity` | Sensitivity |
| `specificity` | 1 − Specificity |
