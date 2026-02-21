# Scientific Axis Labels — Complete Reference

Ready-to-use axis labels with proper units, subscripts, and superscripts for clinical and biomedical research figures.

## Usage

**Python (matplotlib):**
```python
xlabel(ax, "egfr")       # → "eGFR (mL/min/1.73 m²)"
ylabel(ax, "pf_ratio")   # → "PaO₂/FiO₂ Ratio"
xlabel(ax, "Custom text") # pass-through if not in dict
```

**R (ggplot2):**
```r
labs(x = sci_lab("time_days"), y = sci_lab("egfr"))
```

## Complete Label Dictionary

### Survival / Time-to-Event

| Key | Python Output | R Output |
|-----|--------------|----------|
| `time_days` | Time (days) | Time (days) |
| `time_months` | Time (months) | Time (months) |
| `time_years` | Time (years) | Time (years) |
| `survival` | Survival Probability | Survival Probability |
| `cumulative_inc` | Cumulative Incidence (%) | Cumulative Incidence (%) |
| `event_free` | Event-Free Survival | Event-Free Survival |

### Clinical Endpoints

| Key | Output |
|-----|--------|
| `change_baseline` | Change From Baseline |
| `pct_change` | Percent Change From Baseline (%) |
| `response_rate` | Response Rate |
| `proportion` | Proportion |

### Effect Sizes

| Key | Output |
|-----|--------|
| `hr` | Hazard Ratio (95% CI) |
| `or` | Odds Ratio (95% CI) |
| `rr` | Relative Risk (95% CI) |
| `aor` | Adjusted Odds Ratio (95% CI) |
| `mean_diff` | Mean Difference (95% CI) |

### Lab Values / Biomarkers

| Key | Python Output | R Output |
|-----|--------------|----------|
| `creatinine` | Serum Creatinine (mg/dL) | Serum Creatinine (mg/dL) |
| `egfr` | eGFR (mL/min/1.73 m²) | `expression(eGFR~(mL/min/1.73~m^2))` |
| `lactate` | Lactate (mmol/L) | Lactate (mmol/L) |
| `crp` | C-Reactive Protein (mg/L) | C-Reactive Protein (mg/L) |
| `hemoglobin` | Hemoglobin (g/dL) | Hemoglobin (g/dL) |
| `platelets` | Platelet Count (×10³/μL) | `expression(Platelet~Count~(x10^3/mu*L))` |
| `wbc` | WBC Count (×10³/μL) | `expression(WBC~Count~(x10^3/mu*L))` |
| `troponin` | Troponin I (ng/mL) | Troponin I (ng/mL) |
| `bnp` | NT-proBNP (pg/mL) | NT-proBNP (pg/mL) |
| `concentration` | Serum Concentration (ng/mL) | Serum Concentration (ng/mL) |
| `albumin` | Serum Albumin (g/dL) | Serum Albumin (g/dL) |
| `bilirubin` | Total Bilirubin (mg/dL) | Total Bilirubin (mg/dL) |
| `inr` | INR | INR |
| `glucose` | Blood Glucose (mg/dL) | Blood Glucose (mg/dL) |
| `hba1c` | Hemoglobin A1c (%) | Hemoglobin A1c (%) |

### Vitals / Physiology

| Key | Python Output | R Output |
|-----|--------------|----------|
| `map` | Mean Arterial Pressure (mm Hg) | Mean Arterial Pressure (mm Hg) |
| `sbp` | Systolic Blood Pressure (mm Hg) | Systolic Blood Pressure (mm Hg) |
| `dbp` | Diastolic Blood Pressure (mm Hg) | Diastolic Blood Pressure (mm Hg) |
| `heart_rate` | Heart Rate (beats/min) | Heart Rate (beats/min) |
| `spo2` | SpO₂ (%) | `expression(SpO[2]~("%"))` |
| `fio2` | FiO₂ (%) | `expression(FiO[2]~("%"))` |
| `temperature` | Temperature (°C) | Temperature (°C) |
| `rr_resp` | Respiratory Rate (breaths/min) | Respiratory Rate (breaths/min) |
| `weight` | Body Weight (kg) | Body Weight (kg) |
| `bmi` | BMI (kg/m²) | `expression(BMI~(kg/m^2))` |

### Pulmonary Function

| Key | Python Output | R Output |
|-----|--------------|----------|
| `fev1` | FEV₁ (L) | `expression(FEV[1]~(L))` |
| `fvc` | FVC (L) | FVC (L) |
| `fev1_fvc` | FEV₁/FVC Ratio | `expression(FEV[1]/FVC~Ratio)` |
| `dlco` | DLCO (mL/min/mm Hg) | DLCO (mL/min/mm Hg) |
| `pao2` | PaO₂ (mm Hg) | `expression(PaO[2]~(mm~Hg))` |
| `paco2` | PaCO₂ (mm Hg) | `expression(PaCO[2]~(mm~Hg))` |
| `pf_ratio` | PaO₂/FiO₂ Ratio | `expression(PaO[2]/FiO[2]~Ratio)` |
| `peep` | PEEP (cm H₂O) | `expression(PEEP~(cm~H[2]*O))` |
| `tidal_volume` | Tidal Volume (mL/kg PBW) | Tidal Volume (mL/kg PBW) |

### ICU / Critical Care

| Key | Output |
|-----|--------|
| `sofa` | SOFA Score |
| `apache` | APACHE II Score |
| `gcs` | Glasgow Coma Scale |
| `ventilator_days` | Ventilator-Free Days |
| `icu_los` | ICU Length of Stay (days) |
| `hospital_los` | Hospital Length of Stay (days) |
| `fluid_balance` | Cumulative Fluid Balance (mL) |
| `vasopressor_dose` | Norepinephrine Equivalent Dose (μg/kg/min) |

### Diagnostics / Model Performance

| Key | Output |
|-----|--------|
| `sensitivity` | Sensitivity |
| `specificity` | 1 − Specificity |
| `auc` | Area Under the ROC Curve |
| `predicted_prob` | Predicted Probability |
| `observed_freq` | Observed Frequency |
| `decile_risk` | Decile of Predicted Risk |
| `observed_rate` | Observed Event Rate (%) |
| `brier_score` | Brier Score |
| `calibration_slope` | Calibration Slope |

### Epidemiology / Population Health

| Key | Output |
|-----|--------|
| `calendar_year` | Calendar Year |
| `age` | Age (years) |
| `incidence_rate` | Incidence Rate (per 100,000) |
| `prevalence` | Prevalence (%) |
| `mortality_rate` | Mortality Rate (per 1,000) |
| `person_years` | Person-Years at Risk |
| `life_expectancy` | Life Expectancy (years) |

## Formatting Rules

1. **Always include units** in parentheses: `"Creatinine (mg/dL)"` not just `"Creatinine"`
2. **Sentence case**: Capitalize first word only — `"Systolic blood pressure (mm Hg)"` is also acceptable
3. **Subscripts**: Python uses `$_1$`, R uses `[1]` in expressions
4. **Superscripts**: Python uses `$^2$`, R uses `^2` in expressions
5. **Special characters**: Use proper symbols — `"×"` not `"x"`, `"μ"` not `"u"`, `"≥"` not `">="`, `"−"` not `"-"`
6. **Log scales**: Label the quantity, not the transform. Write `"Hazard Ratio"` and let the log ticks speak for themselves
7. **Percent**: Choose one format per figure — either 0–100 (label as `"%"`) or 0.0–1.0 (label as `"Proportion"`)
