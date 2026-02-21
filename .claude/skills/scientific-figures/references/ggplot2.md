# ggplot2 Reference — Scientific Figures

Complete copy-paste R code for publication-quality figures.

## Theme, Palette & Helpers

```r
# ═══════════════════════════════════════════════════════════════
# sciplot.R — Professional Scientific Figure Style for ggplot2
# Synthesized from JAMA, Nature, NEJM specifications.
# ═══════════════════════════════════════════════════════════════

library(ggplot2)

# ── Color Palettes ────────────────────────────────────────────
# Okabe-Ito / Wong — colorblind-safe (Nature Methods 2011)
okabe_ito <- c(
  blue       = "#0072B2",
  vermillion = "#D55E00",
  green      = "#009E73",
  orange     = "#E69F00",
  purple     = "#CC79A7",
  sky_blue   = "#56B4E9",
  black      = "#000000"
)

bar_fills <- c("#A8D4E6", "#F5B895", "#8CD4BD",
               "#F5D280", "#E6B8D4", "#B8E2F5")

tx_color   <- "#0072B2"
ctrl_color <- "#D55E00"

scale_color_sci <- function(...) {
  scale_color_manual(values = unname(okabe_ito), ...)
}
scale_fill_sci <- function(...) {
  scale_fill_manual(values = bar_fills, ...)
}
scale_color_tx_ctrl <- function(...) {
  scale_color_manual(values = c("Treatment" = tx_color,
                                "Control"   = ctrl_color), ...)
}

# ── Dimensions ────────────────────────────────────────────────
SINGLE_COL  <- 3.5
ONEHALF_COL <- 5.5
DOUBLE_COL  <- 7.0
MAX_HEIGHT  <- 9.0


# ── Main Theme ────────────────────────────────────────────────
theme_sci <- function(base_size = 7, base_family = "Helvetica") {
  theme_bw(base_size = base_size, base_family = base_family) %+replace%
    theme(
      # Panel — clean white, no grid
      panel.grid.major    = element_blank(),
      panel.grid.minor    = element_blank(),
      panel.border        = element_blank(),
      panel.background    = element_rect(fill = "white", color = NA),
      plot.background     = element_rect(fill = "white", color = NA),

      # Axes — open frame (left + bottom only)
      axis.line           = element_line(color = "black", linewidth = 0.3),
      axis.ticks          = element_line(color = "black", linewidth = 0.3),
      axis.ticks.length   = unit(1.5, "pt"),
      axis.title          = element_text(size = 7, face = "plain",
                                          color = "black"),
      axis.title.x        = element_text(margin = margin(t = 3)),
      axis.title.y        = element_text(margin = margin(r = 3)),
      axis.text           = element_text(size = 6.5, color = "black"),

      # Legend — frameless
      legend.background   = element_blank(),
      legend.key          = element_blank(),
      legend.key.size     = unit(0.7, "lines"),
      legend.key.spacing  = unit(0.3, "lines"),
      legend.title        = element_text(size = 7, face = "bold"),
      legend.text         = element_text(size = 6.5),
      legend.position     = "bottom",
      legend.margin       = margin(t = -2),

      # Facets
      strip.background    = element_blank(),
      strip.text          = element_text(size = 7, face = "bold", hjust = 0),

      # Plot
      plot.title          = element_text(size = 8, face = "bold", hjust = 0),
      plot.subtitle       = element_text(size = 7, hjust = 0,
                                          margin = margin(b = 4)),
      plot.caption        = element_text(size = 6, hjust = 1, color = "gray40"),
      plot.margin         = margin(4, 4, 4, 4),
      plot.tag            = element_text(size = 9, face = "bold")
    )
}


# ═══════════════════════════════════════════════════════════════
# ANTI-OVERLAP HELPERS
# ═══════════════════════════════════════════════════════════════

# Rotate x-axis labels to prevent collision
rotate_x <- function(angle = 45, hjust = 1, vjust = 1) {
  theme(axis.text.x = element_text(
    angle = angle, hjust = hjust, vjust = vjust, size = 6.5))
}

# Safe legend positions — never on top of data
legend_bottom <- function(ncol = 3) {
  theme(legend.position = "bottom",
        legend.direction = "horizontal",
        legend.box = "horizontal",
        legend.margin = margin(t = -2, b = 0),
        legend.text = element_text(size = 6.5))
}

legend_right <- function() {
  theme(legend.position = "right",
        legend.direction = "vertical",
        legend.margin = margin(l = 2))
}

legend_inside <- function(x = 0.8, y = 0.9) {
  theme(legend.position = "inside",
        legend.position.inside = c(x, y),
        legend.background = element_blank(),
        legend.key = element_blank(),
        legend.key.size = unit(0.6, "lines"))
}

# Reduce tick density to prevent label collisions
scale_x_clean <- function(n = 6, ...) {
  scale_x_continuous(breaks = scales::pretty_breaks(n = n), ...)
}
scale_y_clean <- function(n = 6, ...) {
  scale_y_continuous(breaks = scales::pretty_breaks(n = n), ...)
}


# ═══════════════════════════════════════════════════════════════
# SCIENTIFIC AXIS LABELS
# ═══════════════════════════════════════════════════════════════

sci_labels <- list(
  # Survival / time-to-event
  time_days       = "Time (days)",
  time_months     = "Time (months)",
  time_years      = "Time (years)",
  survival        = "Survival Probability",
  cumulative_inc  = "Cumulative Incidence (%)",
  event_free      = "Event-Free Survival",

  # Clinical endpoints
  change_baseline = "Change From Baseline",
  pct_change      = "Percent Change From Baseline (%)",
  response_rate   = "Response Rate",
  proportion      = "Proportion",

  # Effect sizes
  hr              = "Hazard Ratio (95% CI)",
  or              = "Odds Ratio (95% CI)",
  rr              = "Relative Risk (95% CI)",
  aor             = "Adjusted Odds Ratio (95% CI)",
  mean_diff       = "Mean Difference (95% CI)",

  # Labs / biomarkers
  creatinine      = "Serum Creatinine (mg/dL)",
  egfr            = expression(eGFR~(mL/min/1.73~m^2)),
  lactate         = "Lactate (mmol/L)",
  crp             = "C-Reactive Protein (mg/L)",
  hemoglobin      = "Hemoglobin (g/dL)",
  platelets       = expression(Platelet~Count~(x10^3/mu*L)),
  troponin        = "Troponin I (ng/mL)",

  # Vitals / physiology
  map             = "Mean Arterial Pressure (mm Hg)",
  sbp             = "Systolic Blood Pressure (mm Hg)",
  heart_rate      = "Heart Rate (beats/min)",
  spo2            = expression(SpO[2]~("%")),
  fio2            = expression(FiO[2]~("%")),
  temperature     = "Temperature (°C)",

  # Pulmonary
  fev1            = expression(FEV[1]~(L)),
  fvc             = "FVC (L)",
  fev1_fvc        = expression(FEV[1]/FVC~Ratio),
  pf_ratio        = expression(PaO[2]/FiO[2]~Ratio),
  pao2            = expression(PaO[2]~(mm~Hg)),

  # ICU
  sofa            = "SOFA Score",
  apache          = "APACHE II Score",
  ventilator_days = "Ventilator-Free Days",
  icu_los         = "ICU Length of Stay (days)",
  hospital_los    = "Hospital Length of Stay (days)",

  # Diagnostics
  sensitivity     = "Sensitivity",
  specificity     = "1 \u2212 Specificity",
  predicted_prob  = "Predicted Probability",
  observed_freq   = "Observed Frequency",

  # Epidemiology
  age             = "Age (years)",
  incidence_rate  = "Incidence Rate (per 100,000)",
  mortality_rate  = "Mortality Rate (per 1,000)"
)

sci_lab <- function(key) {
  if (key %in% names(sci_labels)) sci_labels[[key]] else key
}


# ═══════════════════════════════════════════════════════════════
# P-VALUE FORMATTING
# ═══════════════════════════════════════════════════════════════

format_pvalue <- function(p, style = "jama") {
  if (style %in% c("jama", "nejm")) {
    if (p < 0.001) return("P<.001")
    if (p < 0.01)  return(sub("^0", "", sprintf("P=%.3f", p)))
    return(sub("^P=0", "P=", sprintf("P=%.2f", p)))
  } else {
    if (p < 0.001) return("P < 0.001")
    return(sprintf("P = %.2f", p))
  }
}


# ═══════════════════════════════════════════════════════════════
# EXPORT
# ═══════════════════════════════════════════════════════════════

save_sci <- function(plot, filename, width = DOUBLE_COL, height = 4,
                     units = "in", formats = c("pdf", "tiff", "png")) {
  for (fmt in formats) {
    dpi <- ifelse(fmt == "tiff", 600, 300)
    fname <- paste0(filename, ".", fmt)
    if (fmt == "pdf") {
      ggsave(fname, plot, width = width, height = height,
             units = units, device = cairo_pdf)
    } else if (fmt == "tiff") {
      ggsave(fname, plot, width = width, height = height,
             units = units, dpi = dpi, compression = "lzw")
    } else {
      ggsave(fname, plot, width = width, height = height,
             units = units, dpi = dpi)
    }
  }
  message("\u2713 Saved: ",
          paste(paste0(filename, ".", formats), collapse = ", "))
}
```

## Example: Multi-Panel with patchwork

```r
library(patchwork)

# JAMA style (uppercase A, B)
combined <- (p1 | p2) / (p3 | p4) +
  plot_annotation(tag_levels = "A") &
  theme_sci() &
  theme(plot.tag = element_text(size = 9, face = "bold"))

# Nature style (lowercase a, b)
combined_nature <- (p1 | p2) +
  plot_annotation(tag_levels = "a") &
  theme_sci() &
  theme(plot.tag = element_text(size = 9, face = "bold"))

save_sci(combined, "figure1", width = DOUBLE_COL, height = 5.5)
```

## Example: KM Curve

```r
library(survival)
library(survminer)

fit <- survfit(Surv(time, status) ~ sex, data = lung)

p_km <- ggsurvplot(
  fit,
  data         = lung,
  palette      = c(tx_color, ctrl_color),
  risk.table   = TRUE,
  risk.table.height = 0.25,
  pval         = TRUE,
  pval.method  = TRUE,
  conf.int     = FALSE,
  xlab         = sci_lab("time_days"),
  ylab         = sci_lab("survival"),
  legend.title = "",
  legend.labs  = c("Male (n=138)", "Female (n=90)"),
  ggtheme      = theme_sci(base_size = 7),
  font.x = 7, font.y = 7, font.tickslab = 6.5, font.legend = 6.5,
  tables.theme = theme_sci(base_size = 6.5)
)
```

## Example: Forest Plot

```r
df <- data.frame(
  subgroup = c("Overall", "Age \u226565", "Age <65", "Male", "Female"),
  hr       = c(0.72, 0.68, 0.78, 0.70, 0.75),
  lower    = c(0.58, 0.49, 0.60, 0.52, 0.57),
  upper    = c(0.89, 0.94, 1.01, 0.94, 0.99)
)
df$subgroup <- factor(df$subgroup, levels = rev(df$subgroup))

ggplot(df, aes(x = hr, y = subgroup)) +
  geom_vline(xintercept = 1, linetype = "dashed", linewidth = 0.4) +
  geom_errorbarh(aes(xmin = lower, xmax = upper),
                 height = 0.2, linewidth = 0.5) +
  geom_point(size = 2.5, shape = 15, color = tx_color) +
  scale_x_continuous(trans = "log",
                     breaks = c(0.5, 0.7, 1.0, 1.5),
                     limits = c(0.4, 1.6)) +
  labs(x = sci_lab("hr"), y = NULL) +
  theme_sci() +
  theme(axis.line.y = element_blank(), axis.ticks.y = element_blank())
```

## Anti-Overlap Quick Reference

```r
# Long categorical x-axis labels
+ rotate_x(angle = 45)

# Dense numeric axes
+ scale_x_clean(n = 5)
+ scale_y_clean(n = 5)

# Legend placement (pick one)
+ legend_bottom(ncol = 4)           # safest — below plot
+ legend_right()                    # right side
+ legend_inside(x = 0.8, y = 0.95) # inside empty region

# Prevent clipping of panel labels
+ coord_cartesian(clip = "off")
```
