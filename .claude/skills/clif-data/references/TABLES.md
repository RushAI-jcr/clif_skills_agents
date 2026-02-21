# CLIF 2.1 Table Reference

## 16 Beta Tables (Production)

| Table | Key Columns | Purpose |
|---|---|---|
| `patient` | patient_id, sex_category, race_category, ethnicity_category, birth_date, death_dttm | Demographics |
| `hospitalization` | hospitalization_id, patient_id, admission_dttm, discharge_dttm, age_at_admission, discharge_category | Encounter spine |
| `adt` | hospitalization_id, location_category (icu/ed/ward), in_dttm, out_dttm | Location timeline |
| `vitals` | hospitalization_id, recorded_dttm, vital_category, vital_value | Time-series vitals |
| `labs` | hospitalization_id, lab_collect_dttm, lab_category, lab_value_numeric, reference_unit | Time-series labs |
| `respiratory_support` | hospitalization_id, recorded_dttm, device_category, mode_category, fio2_set, peep_set, tidal_volume_set, tidal_volume_obs | Ventilator data |
| `medication_admin_continuous` | hospitalization_id, admin_dttm, med_category, med_group, med_dose, med_dose_unit, mar_action_group | Drips/infusions |
| `medication_admin_intermittent` | hospitalization_id, admin_dttm, med_category, med_group, med_dose, med_dose_unit, mar_action_group | Bolus doses |
| `patient_assessments` | hospitalization_id, recorded_dttm, assessment_category, numerical_value | GCS, RASS, pain |
| `hospital_diagnosis` | hospitalization_id, diagnosis_code, diagnosis_code_format (ICD10CM), diagnosis_primary, poa_present | Billing diagnoses |
| `microbiology_culture` | hospitalization_id, collect_dttm, organism_category, organism_group, fluid_category | Culture results |
| `microbiology_susceptibility` | organism_id, antimicrobial_category, susceptibility_category | Antibiotic sensitivity |
| `position` | hospitalization_id, recorded_dttm, position_category (prone/not_prone) | Prone positioning |
| `crrt_therapy` | hospitalization_id, recorded_dttm, crrt_mode_category, blood_flow_rate | Dialysis |
| `code_status` | patient_id, start_dttm, code_status_category | Goals of care |
| `patient_procedures` | hospitalization_id, procedure_code, procedure_code_format | Billing procedures |

## Common _category Values

### vital_category
temp_c, heart_rate, sbp, dbp, spo2, map, resp_rate, height_cm, weight_kg

### lab_category
hemoglobin, wbc, platelets, creatinine, bilirubin_total, lactate, pao2, paco2, ph_arterial, sodium, potassium, glucose, bun, albumin, inr, fibrinogen, d_dimer, troponin, bnp, procalcitonin

### device_category
IMV, NIPPV, CPAP, High Flow NC, Low Flow NC, Tracheostomy Collar, Room Air

### med_category (continuous)
norepinephrine, vasopressin, epinephrine, phenylephrine, dopamine, dobutamine, milrinone, propofol, midazolam, fentanyl, dexmedetomidine, cisatracurium

### assessment_category
gcs_total, gcs_eye, gcs_verbal, gcs_motor, rass, cam_icu, pain_score

### location_category
icu, ed, ward, step_down, or, pacu, procedure

## 12 Concept Tables (Emerging, CLIF 2.2)
ecmo_mcs, intake_output, invasive_hemodynamics, medication_orders, microbiology_nonculture, transfusion, clinical_trial, patient_diagnosis, place_based_index, provider, therapy_details, key_icu_orders

## SOFA Score Components
| Component | Source Table | Column |
|---|---|---|
| Respiration (PaO2/FiO2) | labs + respiratory_support | lab_category='pao2' + fio2_set |
| Coagulation | labs | lab_category='platelets' |
| Liver | labs | lab_category='bilirubin_total' |
| Cardiovascular | vitals + medication_admin_continuous | vital_category='map' + vasopressor doses |
| CNS | patient_assessments | assessment_category='gcs_total' |
| Renal | labs | lab_category='creatinine' |
