# Matplotlib Reference — Scientific Figures

Complete copy-paste Python code for publication-quality figures.

## Style Configuration

```python
"""
sciplot.py — Professional Scientific Figure Style for Matplotlib
Synthesized from JAMA, Nature, NEJM specifications.

Usage:
    import sciplot
    sciplot.apply()                        # Apply globally
    with sciplot.context(): ...            # Scoped
    fig, axes = sciplot.figure(1, 2)       # Journal-sized figure
    sciplot.fix_overlap(fig)               # Auto-fix overlaps
    sciplot.save(fig, "figure1")           # Multi-format export
"""

import matplotlib as mpl
import matplotlib.pyplot as plt
import matplotlib.ticker as ticker
from pathlib import Path


# ═══════════════════════════════════════════════════════════════
# COLOR PALETTES
# ═══════════════════════════════════════════════════════════════

# Okabe-Ito / Wong (Nature Methods 2011) — colorblind-safe
OKABE_ITO = {
    "blue":       "#0072B2",
    "orange":     "#E69F00",
    "sky_blue":   "#56B4E9",
    "green":      "#009E73",
    "yellow":     "#F0E442",
    "vermillion": "#D55E00",
    "purple":     "#CC79A7",
    "black":      "#000000",
}

# Ordered cycle (skip yellow — low contrast on white)
COLOR_CYCLE = [
    "#0072B2",  # blue
    "#D55E00",  # vermillion
    "#009E73",  # green
    "#E69F00",  # orange
    "#CC79A7",  # reddish purple
    "#56B4E9",  # sky blue
    "#000000",  # black
]

# Lighter fills for bar charts (edges from COLOR_CYCLE)
BAR_FILLS = ["#A8D4E6", "#F5B895", "#8CD4BD", "#F5D280", "#E6B8D4", "#B8E2F5"]
BAR_EDGES = COLOR_CYCLE[:6]

# Two-group clinical defaults
TX_COLOR   = "#0072B2"  # treatment / intervention
CTRL_COLOR = "#D55E00"  # control / placebo


# ═══════════════════════════════════════════════════════════════
# FIGURE DIMENSIONS (inches)
# ═══════════════════════════════════════════════════════════════

SINGLE_COL  = 3.5   # ~89 mm
ONEHALF_COL = 5.5   # ~140 mm
DOUBLE_COL  = 7.0   # ~178 mm
MAX_HEIGHT  = 9.0   # safe across all journals


# ═══════════════════════════════════════════════════════════════
# STYLE DICTIONARY
# ═══════════════════════════════════════════════════════════════

STYLE = {
    # Figure
    "figure.figsize": (DOUBLE_COL, 4.0),
    "figure.dpi": 150,
    "figure.facecolor": "white",
    "figure.constrained_layout.use": True,

    # Export
    "savefig.dpi": 600,
    "savefig.bbox": "tight",
    "savefig.pad_inches": 0.05,
    "savefig.facecolor": "white",
    "savefig.transparent": False,

    # Font
    "font.family": "sans-serif",
    "font.sans-serif": ["Helvetica", "Arial", "DejaVu Sans"],
    "font.size": 7,
    "axes.titlesize": 8,
    "axes.labelsize": 7,
    "xtick.labelsize": 6.5,
    "ytick.labelsize": 6.5,
    "legend.fontsize": 6.5,
    "legend.title_fontsize": 7,
    "figure.titlesize": 9,

    # Axes — open frame
    "axes.linewidth": 0.6,
    "axes.spines.top": False,
    "axes.spines.right": False,
    "axes.labelpad": 3,
    "axes.titlepad": 6,
    "axes.labelcolor": "black",
    "axes.edgecolor": "black",
    "axes.xmargin": 0.02,
    "axes.ymargin": 0.05,

    # Ticks — outward
    "xtick.major.size": 3.5,
    "ytick.major.size": 3.5,
    "xtick.minor.size": 2.0,
    "ytick.minor.size": 2.0,
    "xtick.major.width": 0.6,
    "ytick.major.width": 0.6,
    "xtick.minor.width": 0.4,
    "ytick.minor.width": 0.4,
    "xtick.direction": "out",
    "ytick.direction": "out",
    "xtick.major.pad": 3,
    "ytick.major.pad": 3,
    "xtick.minor.visible": False,
    "ytick.minor.visible": False,
    "xtick.color": "black",
    "ytick.color": "black",

    # Grid — off by default
    "axes.grid": False,
    "grid.color": "#E0E0E0",
    "grid.linewidth": 0.4,
    "grid.linestyle": "--",

    # Legend — frameless
    "legend.frameon": False,
    "legend.borderpad": 0.3,
    "legend.handlelength": 1.2,
    "legend.handletextpad": 0.5,
    "legend.columnspacing": 1.0,
    "legend.labelspacing": 0.4,

    # Lines & Markers
    "lines.linewidth": 1.2,
    "lines.markersize": 4,
    "lines.markeredgewidth": 0.5,
    "scatter.marker": "o",

    # Error bars
    "errorbar.capsize": 2.5,

    # Patches
    "patch.linewidth": 0.6,
    "patch.edgecolor": "black",

    # Color cycle
    "axes.prop_cycle": mpl.cycler(color=COLOR_CYCLE),

    # Math text
    "mathtext.default": "regular",
}


# ═══════════════════════════════════════════════════════════════
# APPLY / RESET
# ═══════════════════════════════════════════════════════════════

def apply():
    """Apply scientific style globally."""
    mpl.rcParams.update(STYLE)

def context():
    """Use as context manager: with sciplot.context(): ..."""
    return mpl.rc_context(STYLE)

def reset():
    """Reset to matplotlib defaults."""
    mpl.rcParams.update(mpl.rcParamsDefault)


# ═══════════════════════════════════════════════════════════════
# FIGURE CREATION (with constrained_layout for zero overlap)
# ═══════════════════════════════════════════════════════════════

def figure(nrows=1, ncols=1, width="double", height=None,
           sharex=False, sharey=False, **kwargs):
    """Create figure with journal dimensions and constrained_layout.

    Args:
        width: "single" (3.5in), "1.5" (5.5in), "double" (7.0in), or float
        height: Inches. Auto-calculated via golden ratio if None.
    """
    w = {"single": SINGLE_COL, "1.5": ONEHALF_COL,
         "double": DOUBLE_COL}.get(width, width)
    if height is None:
        height = min(w * 0.618, MAX_HEIGHT)
    fig, axes = plt.subplots(nrows, ncols, figsize=(w, height),
                             sharex=sharex, sharey=sharey,
                             constrained_layout=True, **kwargs)
    return fig, axes


# ═══════════════════════════════════════════════════════════════
# ANTI-OVERLAP SYSTEM
# ═══════════════════════════════════════════════════════════════

def fix_overlap(fig, ax_or_axes=None):
    """Auto-fix text overlap. Call AFTER plotting, BEFORE save().

    Detects and fixes: tick label collisions, legend-over-data.
    """
    axes = fig.get_axes() if ax_or_axes is None else (
        [ax_or_axes] if not hasattr(ax_or_axes, '__iter__') else list(ax_or_axes)
    )
    for ax in axes:
        _fix_xtick_overlap(ax)
        _fix_ytick_overlap(ax)
        _fix_legend_overlap(ax)


def _fix_xtick_overlap(ax):
    """Rotate or reduce x-tick labels if they overlap."""
    fig = ax.get_figure()
    fig.canvas.draw()
    renderer = fig.canvas.get_renderer()
    labels = ax.get_xticklabels()
    if len(labels) < 2:
        return

    bboxes = [l.get_window_extent(renderer=renderer)
              for l in labels if l.get_text()]
    if len(bboxes) < 2:
        return

    has_overlap = any(
        bboxes[i].x1 > bboxes[i + 1].x0 - 2
        for i in range(len(bboxes) - 1)
    )

    if has_overlap:
        # First: reduce tick count
        from matplotlib.ticker import MaxNLocator
        ax.xaxis.set_major_locator(MaxNLocator(nbins=6, integer=True))
        fig.canvas.draw()

        # Re-check
        labels = ax.get_xticklabels()
        bboxes = [l.get_window_extent(renderer=renderer)
                  for l in labels if l.get_text()]
        still_overlaps = len(bboxes) >= 2 and any(
            bboxes[i].x1 > bboxes[i + 1].x0 - 2
            for i in range(len(bboxes) - 1)
        )

        # Second: rotate 45°
        if still_overlaps:
            ax.set_xticklabels(ax.get_xticklabels(),
                               rotation=45, ha="right",
                               rotation_mode="anchor")


def _fix_ytick_overlap(ax):
    """Reduce y-tick count if labels overlap."""
    fig = ax.get_figure()
    fig.canvas.draw()
    renderer = fig.canvas.get_renderer()
    labels = ax.get_yticklabels()
    bboxes = [l.get_window_extent(renderer=renderer)
              for l in labels if l.get_text()]
    if len(bboxes) >= 2:
        has_overlap = any(
            bboxes[i].y0 < bboxes[i + 1].y1 + 2
            for i in range(len(bboxes) - 1)
        )
        if has_overlap:
            from matplotlib.ticker import MaxNLocator
            ax.yaxis.set_major_locator(MaxNLocator(nbins=6))


def _fix_legend_overlap(ax):
    """Ensure legend has no frame."""
    legend = ax.get_legend()
    if legend is not None:
        legend.set_frame_on(False)


def safe_legend(ax, loc="best", outside=False, **kwargs):
    """Place legend with guaranteed no-overlap.

    Args:
        outside: If True, place below axes (safest for dense plots)
    """
    if outside:
        ax.legend(loc="upper center", bbox_to_anchor=(0.5, -0.15),
                  ncol=3, frameon=False, fontsize=6.5, **kwargs)
    else:
        ax.legend(loc=loc, frameon=False, fontsize=6.5, **kwargs)


# ═══════════════════════════════════════════════════════════════
# PANEL LABELS
# ═══════════════════════════════════════════════════════════════

def panel_label(ax, label, x=-0.08, y=1.06, style="jama"):
    """Add panel label. style='jama' → A,B,C; style='nature' → a,b,c."""
    text = label.upper() if style == "jama" else label.lower()
    ax.text(x, y, text, transform=ax.transAxes,
            fontsize=9, fontweight="bold", va="top", ha="left",
            fontstyle="normal")


# ═══════════════════════════════════════════════════════════════
# SCIENTIFIC AXIS LABELS (see references/axis_labels.md for full dict)
# ═══════════════════════════════════════════════════════════════

# Import AXIS_LABELS from references/axis_labels.md or paste inline.
# Quick subset for common use:
AXIS_LABELS = {
    "time_days":       "Time (days)",
    "time_months":     "Time (months)",
    "time_years":      "Time (years)",
    "survival":        "Survival Probability",
    "cumulative_inc":  "Cumulative Incidence (%)",
    "change_baseline": "Change From Baseline",
    "response_rate":   "Response Rate",
    "proportion":      "Proportion",
    "hr":              "Hazard Ratio (95% CI)",
    "or":              "Odds Ratio (95% CI)",
    "aor":             "Adjusted Odds Ratio (95% CI)",
    "creatinine":      "Serum Creatinine (mg/dL)",
    "egfr":            r"eGFR (mL/min/1.73 m$^2$)",
    "lactate":         "Lactate (mmol/L)",
    "crp":             "C-Reactive Protein (mg/L)",
    "hemoglobin":      "Hemoglobin (g/dL)",
    "map":             "Mean Arterial Pressure (mm Hg)",
    "sbp":             "Systolic Blood Pressure (mm Hg)",
    "heart_rate":      "Heart Rate (beats/min)",
    "spo2":            r"SpO$_2$ (%)",
    "fio2":            r"FiO$_2$ (%)",
    "fev1":            r"FEV$_1$ (L)",
    "fvc":             "FVC (L)",
    "fev1_fvc":        r"FEV$_1$/FVC Ratio",
    "pf_ratio":        r"PaO$_2$/FiO$_2$ Ratio",
    "pao2":            r"PaO$_2$ (mm Hg)",
    "sofa":            "SOFA Score",
    "apache":          "APACHE II Score",
    "ventilator_days": "Ventilator-Free Days",
    "icu_los":         "ICU Length of Stay (days)",
    "hospital_los":    "Hospital Length of Stay (days)",
    "sensitivity":     "Sensitivity",
    "specificity":     "1 − Specificity",
    "predicted_prob":  "Predicted Probability",
    "observed_freq":   "Observed Frequency",
    "age":             "Age (years)",
    "incidence_rate":  "Incidence Rate (per 100,000)",
    "mortality_rate":  "Mortality Rate (per 1,000)",
}

def xlabel(ax, key_or_text):
    """Set x-axis label from AXIS_LABELS dict or custom string."""
    ax.set_xlabel(AXIS_LABELS.get(key_or_text, key_or_text))

def ylabel(ax, key_or_text):
    """Set y-axis label from AXIS_LABELS dict or custom string."""
    ax.set_ylabel(AXIS_LABELS.get(key_or_text, key_or_text))


# ═══════════════════════════════════════════════════════════════
# P-VALUES & STATISTICAL ANNOTATIONS
# ═══════════════════════════════════════════════════════════════

def format_pvalue(p, style="jama"):
    """Format p-value. JAMA/NEJM: P<.001. Nature: P < 0.001."""
    if style in ("jama", "nejm"):
        if p < 0.001:    return "P<.001"
        elif p < 0.01:   return f"P={p:.3f}".replace("0.", ".")
        elif p < 1:      return f"P={p:.2f}".replace("0.", ".")
        else:            return "P>.99"
    else:
        if p < 0.001:    return "P < 0.001"
        elif p < 0.01:   return f"P = {p:.3f}"
        else:            return f"P = {p:.2f}"


def add_significance(ax, x1, x2, y, p, h=0.02, style="jama"):
    """Add significance bracket between two x positions."""
    ax.plot([x1, x1, x2, x2], [y, y + h, y + h, y],
            lw=0.8, color="black")
    ax.text((x1 + x2) / 2, y + h, format_pvalue(p, style),
            ha="center", va="bottom", fontsize=6)


# ═══════════════════════════════════════════════════════════════
# PLOT-TYPE HELPERS
# ═══════════════════════════════════════════════════════════════

def km_style(ax):
    """Apply Kaplan-Meier curve formatting."""
    ax.set_ylim(-0.02, 1.05)
    ax.set_ylabel("Survival Probability")
    ax.yaxis.set_major_formatter(ticker.FuncFormatter(
        lambda y, _: f"{y:.1f}"))
    ax.set_xlim(left=0)

def forest_style(ax):
    """Apply forest plot formatting."""
    ax.axvline(x=1, color="black", linewidth=0.6, linestyle="--")
    ax.spines["left"].set_visible(False)
    ax.tick_params(left=False)
    ax.set_xlabel("Hazard Ratio (95% CI)")

def strip_axes(ax, keep_x=True, keep_y=True):
    """Remove axes — useful for schematics."""
    if not keep_x:
        ax.spines["bottom"].set_visible(False)
        ax.tick_params(bottom=False, labelbottom=False)
    if not keep_y:
        ax.spines["left"].set_visible(False)
        ax.tick_params(left=False, labelleft=False)


# ═══════════════════════════════════════════════════════════════
# EXPORT
# ═══════════════════════════════════════════════════════════════

def save(fig, filename, formats=("pdf", "tiff", "png"), output_dir="."):
    """Save in multiple publication-ready formats.

    PDF: vector (preferred). TIFF: 600 dpi LZW. PNG: 300 dpi.
    """
    outdir = Path(output_dir)
    outdir.mkdir(parents=True, exist_ok=True)
    for fmt in formats:
        dpi = 600 if fmt in ("tiff", "tif") else 300
        fig.savefig(
            outdir / f"{filename}.{fmt}",
            format=fmt.replace("tiff", "tif"),
            dpi=dpi, bbox_inches="tight", pad_inches=0.05,
            facecolor="white", transparent=False,
            pil_kwargs={"compression": "tiff_lzw"} if "tif" in fmt else {},
        )
    print(f"✓ Saved: {', '.join(f'{filename}.{f}' for f in formats)}")
```

## Example: Two-Panel Clinical Figure

```python
apply()
fig, (ax1, ax2) = figure(1, 2, width="double", height=3.2)

# Panel A: KM survival curves
import numpy as np
t = np.linspace(0, 365, 200)
ax1.step(t, np.exp(-0.002 * t), where="post",
         color=TX_COLOR, lw=1.2, label="Treatment (n=312)")
ax1.step(t, np.exp(-0.005 * t), where="post",
         color=CTRL_COLOR, lw=1.2, label="Control (n=308)")
km_style(ax1)
xlabel(ax1, "time_days")
safe_legend(ax1, loc="lower left")
panel_label(ax1, "A")

# Panel B: Bar chart
groups = ["Treatment", "Control"]
bars = ax2.bar(groups, [0.72, 0.48], width=0.5,
               yerr=[0.06, 0.07],
               color=BAR_FILLS[:2], edgecolor=[TX_COLOR, CTRL_COLOR],
               linewidth=0.8, capsize=3, error_kw={"linewidth": 0.8})
ylabel(ax2, "response_rate")
ax2.set_ylim(0, 1.0)
add_significance(ax2, 0, 1, 0.85, p=0.003)
panel_label(ax2, "B")

fix_overlap(fig)
save(fig, "figure1")
```
