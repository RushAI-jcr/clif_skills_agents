# CLIF 2.1 Table Reference

> Source: [CLIF Data Dictionary v2.1.0](https://clif-consortium.github.io/website/data-dictionary/data-dictionary-2.1.0.html) and [CLIF 2.1 MySQL DDL](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF/blob/main/ddl/2.1/CLIF2.1_MYSQL_ddl.sql)

All clinical tables link to `hospitalization` via `hospitalization_id`. The `patient` table links via `patient_id`. All datetimes are UTC (`YYYY-MM-DD HH:MM:SS+00:00`).

---

## Beta Tables (Stable)

### patient

| Column | Type | Allowed Values |
|---|---|---|
| patient_id | VARCHAR | — |
| race_name | VARCHAR | Site-specific |
| race_category | VARCHAR | Black or African American, White, American Indian or Alaska Native, Asian, Native Hawaiian or Other Pacific Islander, Unknown, Other |
| ethnicity_name | VARCHAR | Site-specific |
| ethnicity_category | VARCHAR | Hispanic, Non-Hispanic, Unknown |
| sex_name | VARCHAR | Site-specific |
| sex_category | VARCHAR | Male, Female, Unknown |
| birth_date | DATE | YYYY-MM-DD |
| death_dttm | DATETIME | UTC |
| language_name | VARCHAR | Site-specific |
| language_category | VARCHAR | ~60 categories (English, Spanish, French, Chinese, Arabic, etc.) |

### hospitalization

| Column | Type | Allowed Values |
|---|---|---|
| patient_id | VARCHAR | — |
| hospitalization_id | VARCHAR | — |
| hospitalization_joined_id | VARCHAR | Continuous inpatient stay ID (optional) |
| admission_dttm | DATETIME | UTC |
| discharge_dttm | DATETIME | UTC |
| age_at_admission | INT | — |
| admission_type_name | VARCHAR | Site-specific |
| admission_type_category | VARCHAR | ed, facility, osh, direct, elective, other |
| discharge_name | VARCHAR | Site-specific |
| discharge_category | VARCHAR | Home, Skilled Nursing Facility (SNF), Expired, Acute Inpatient Rehab Facility, Hospice, Long Term Care Hospital (LTACH), Acute Care Hospital, Group Home, Chemical Dependency, Against Medical Advice (AMA), Assisted Living, Still Admitted, Missing, Other, Psychiatric Hospital, Shelter, Jail |
| zipcode_nine_digit | VARCHAR | — |
| zipcode_five_digit | VARCHAR | — |
| census_block_code | VARCHAR | 15-digit FIPS |
| census_block_group_code | VARCHAR | 12-digit FIPS |
| census_tract | VARCHAR | 11-digit FIPS |
| state_code | VARCHAR | 2-digit FIPS |
| county_code | VARCHAR | 5-digit FIPS |
| fips_version | VARCHAR | 2000, 2010, 2020 |

### adt

| Column | Type | Allowed Values |
|---|---|---|
| hospitalization_id | VARCHAR | — |
| hospital_id | VARCHAR | — |
| hospital_type | VARCHAR | academic, community, LTACH |
| in_dttm | DATETIME | UTC |
| out_dttm | DATETIME | UTC |
| location_name | VARCHAR | Site-specific |
| location_category | VARCHAR | ed, ward, stepdown, icu, procedural, l&d, hospice, psych, rehab, radiology, dialysis, other |
| location_type | VARCHAR | general_icu, cardiac_icu, cardiothoracic_surgical_icu, mixed_cardiothoracic_icu, surgical_icu, burn_icu, neuro_icu, neurosurgical_icu, mixed_neuro_icu, medical_icu |

### vitals

| Column | Type | Allowed Values |
|---|---|---|
| hospitalization_id | VARCHAR | — |
| recorded_dttm | DATETIME | UTC |
| vital_name | VARCHAR | Site-specific |
| vital_category | VARCHAR | temp_c, heart_rate, sbp, dbp, spo2, respiratory_rate, map, height_cm, weight_kg |
| vital_value | FLOAT | See outlier thresholds below |
| meas_site_name | VARCHAR | Site-specific (optional) |

**Vital outlier thresholds:**

| vital_category | Unit | Lower | Upper |
|---|---|---|---|
| temp_c | °C | 32 | 44 |
| heart_rate | bpm | 0 | 300 |
| sbp | mmHg | 0 | 300 |
| dbp | mmHg | 0 | 200 |
| spo2 | % | 50 | 100 |
| respiratory_rate | breaths/min | 0 | 60 |
| map | mmHg | 0 | 250 |
| height_cm | cm | 76 | 255 |
| weight_kg | kg | 30 | 1100 |

### labs

| Column | Type | Allowed Values |
|---|---|---|
| hospitalization_id | VARCHAR | — |
| lab_order_dttm | DATETIME | UTC |
| lab_collect_dttm | DATETIME | UTC |
| lab_result_dttm | DATETIME | UTC |
| lab_order_name | VARCHAR | Site-specific |
| lab_order_category | VARCHAR | blood_gas, bmp, cbc, coags, lft, misc |
| lab_name | VARCHAR | Site-specific |
| lab_category | VARCHAR | See 54 lab categories below |
| lab_value | VARCHAR | Raw (may be non-numeric) |
| lab_value_numeric | DOUBLE | Parsed numeric |
| reference_unit | VARCHAR | Per lab category |
| lab_specimen_name | VARCHAR | Site-specific |
| lab_specimen_category | VARCHAR | blood/plasma/serum, urine, csf, other |
| lab_loinc_code | VARCHAR | — |

**54 lab categories with units and outlier thresholds:**

| lab_category | reference_unit | Order Group | Lower | Upper |
|---|---|---|---|---|
| albumin | g/dL | lft | 0 | 15 |
| alkaline_phosphatase | U/L | lft | 0 | 5000 |
| alt | U/L | lft | 0 | 20000 |
| ast | U/L | lft | 0 | 20000 |
| basophils_percent | % | cbc | 0 | 100 |
| basophils_absolute | 10^3/µL | cbc | 0 | 50 |
| bicarbonate | mmol/L | bmp | 0 | 50 |
| bilirubin_total | mg/dL | lft | 0 | 80 |
| bilirubin_conjugated | mg/dL | lft | 0 | 50 |
| bilirubin_unconjugated | mg/dL | lft | 0 | 50 |
| bun | mg/dL | bmp | 0 | 250 |
| calcium_total | mg/dL | bmp | 0 | 20 |
| calcium_ionized | mg/dL | misc | 0 | 20 |
| chloride | mmol/L | bmp | 50 | 140 |
| creatinine | mg/dL | bmp | 0 | 20 |
| crp | mg/L | misc | 0 | 1000 |
| eosinophils_percent | % | cbc | 0 | 100 |
| eosinophils_absolute | 10^3/µL | cbc | 0 | 50 |
| esr | mm/hour | misc | 0 | 1000 |
| ferritin | ng/mL | misc | 0 | 300000 |
| glucose_fingerstick | mg/dL | misc | 0 | 2000 |
| glucose_serum | mg/dL | bmp | 0 | 2000 |
| hemoglobin | g/dL | cbc | 2 | 25 |
| phosphate | mg/dL | misc | 0 | 15 |
| inr | (unitless) | coags | 0 | 15 |
| lactate | mmol/L | misc | 0 | 30 |
| ldh | U/L | misc | 0 | 10000 |
| lymphocytes_percent | % | cbc | 0 | 100 |
| lymphocytes_absolute | 10^3/µL | misc | 0 | 50 |
| magnesium | mg/dL | misc | 0 | 10 |
| monocytes_percent | % | cbc | 0 | 100 |
| monocytes_absolute | 10^3/µL | cbc | 0 | 50 |
| neutrophils_percent | % | cbc | 0 | 100 |
| neutrophils_absolute | 10^3/µL | cbc | 0 | 50 |
| pco2_arterial | mmHg | blood_gas | 0 | 250 |
| po2_arterial | mmHg | blood_gas | 0 | 700 |
| pco2_venous | mmHg | blood_gas | 0 | 250 |
| ph_arterial | (unitless) | blood_gas | 6 | 10 |
| ph_venous | (unitless) | blood_gas | 5 | 10 |
| platelet_count | 10^3/µL | cbc | 0 | 2000 |
| potassium | mmol/L | bmp | 0 | 15 |
| procalcitonin | ng/mL | misc | 0 | 1000 |
| pt | sec | coags | 1 | 200 |
| ptt | sec | coags | 1 | 200 |
| so2_arterial | % | blood_gas | 0 | 100 |
| so2_mixed_venous | % | blood_gas | 0 | 100 |
| so2_central_venous | % | blood_gas | 0 | 100 |
| sodium | mmol/L | bmp | 90 | 210 |
| total_protein | g/dL | lft | 0 | 20 |
| troponin_i | ng/L | misc | 0 | 10000 |
| troponin_t | ng/L | misc | 0 | 10000 |
| wbc | 10^3/µL | cbc | 0 | 500 |

### respiratory_support

| Column | Type | Allowed Values |
|---|---|---|
| hospitalization_id | VARCHAR | — |
| recorded_dttm | DATETIME | UTC |
| device_name | VARCHAR | Site-specific |
| device_id | VARCHAR | — |
| device_category | VARCHAR | IMV, NIPPV, CPAP, High Flow NC, Face Mask, Trach Collar, Nasal Cannula, Room Air, Other |
| vent_brand_name | VARCHAR | Optional |
| mode_name | VARCHAR | Site-specific |
| mode_category | VARCHAR | Assist Control-Volume Control, Pressure Control, Pressure-Regulated Volume Control, SIMV, Pressure Support/CPAP, Volume Support, Blow by, Other |
| tracheostomy | INT | 0 = No, 1 = Yes |
| fio2_set | FLOAT | 0.21–1.0 (decimal, never percentage) |
| lpm_set | FLOAT | 0–60 L/min |
| tidal_volume_set | FLOAT | 100–3000 mL |
| resp_rate_set | FLOAT | 0–200 bpm |
| pressure_control_set | FLOAT | -50–50 cmH2O |
| pressure_support_set | FLOAT | -50–50 cmH2O |
| flow_rate_set | FLOAT | -50–100 lpm |
| peak_inspiratory_pressure_set | FLOAT | -50–100 cmH2O |
| inspiratory_time_set | FLOAT | -1–50 sec |
| peep_set | FLOAT | 0–30 cmH2O |
| tidal_volume_obs | FLOAT | 100–3000 mL |
| resp_rate_obs | FLOAT | 0–200 bpm |
| plateau_pressure_obs | FLOAT | 0–100 cmH2O |
| peak_inspiratory_pressure_obs | FLOAT | -50–100 cmH2O |
| peep_obs | FLOAT | 0–50 cmH2O |
| minute_vent_obs | FLOAT | 0–40 L |
| mean_airway_pressure_obs | FLOAT | 0–50 cmH2O |

### medication_admin_continuous

| Column | Type | Allowed Values |
|---|---|---|
| hospitalization_id | VARCHAR | — |
| med_order_id | VARCHAR | — |
| admin_dttm | DATETIME | UTC |
| med_name | VARCHAR | Site-specific |
| med_category | VARCHAR | ~82 categories (see below) |
| med_group | VARCHAR | vasoactives, sedation, paralytics, diuretics, anticoagulation, fluids_electrolytes, cardiac, pulmonary vasodilators (IV), pulmonary vasodilators (inhaled), Inhaled, gastrointestinal, endocrine, others |
| med_route_name | VARCHAR | Site-specific |
| med_route_category | VARCHAR | im, inhaled, iv |
| med_dose | FLOAT | — |
| med_dose_unit | VARCHAR | Rate unit |
| mar_action_name | VARCHAR | Site-specific |
| mar_action_category | VARCHAR | dose_change, going, start, stop, verify, other |
| mar_action_group | VARCHAR | administered, not_administered, other |

**Continuous med_category values:** albumin, albuterol, alprostadil, aminocaproic_acid, aminophylline, amiodarone, angiotensin, argatroban, baclofen, bivalirudin, bumetanide, bupivacaine, cangrelor, cisatracurium, clevidipine, dexmedetomidine, dextrose_10_water, dextrose_5_water, dextrose_other, diltiazem, dobutamine, dopamine, epinephrine, epoprostenol (IV), epoprostenol (inhaled), eptifibatide, esmolol, esomeprazole, fentanyl, furosemide, heparin, hydromorphone, insulin, ipratropium, isoproterenol, ketamine, labetalol, lactated_ringers_solution, levothyroxine, lidocaine, liothyronine, lorazepam, magnesium_sulfate, midazolam, milrinone, morphine, naloxone, nicardipine, nitric_oxide, nitroglycerin, nitroprusside, norepinephrine, octreotide, oxytocin, pantoprazole, papaverine, pentobarbital, phentolamine, phenylephrine, pitocin, plasma_lyte, procainamide, propofol, remifentanil, rocuronium, ropivacaine, sodium_bicarbonate, sodium_chloride, tacrolimus, terbutaline (inhaled), terbutaline (IV), torsemide, tpn, treprostinil, vasopressin, vecuronium, zidovudine

### medication_admin_intermittent

| Column | Type | Allowed Values |
|---|---|---|
| hospitalization_id | VARCHAR | — |
| med_order_id | VARCHAR | — |
| admin_dttm | DATETIME | UTC |
| med_name | VARCHAR | Site-specific |
| med_category | VARCHAR | ~140 categories |
| med_group | VARCHAR | CMS_sepsis_qualifying_antibiotics, analgesia, steroid, sedation, paralytics, antipsychotic, anxiolytic, vasopressor, car_t, other |
| med_route_name | VARCHAR | Site-specific |
| med_route_category | VARCHAR | buccal_sublingual, enteral, im, intrapleural, iv |
| med_dose | FLOAT | — |
| med_dose_unit | VARCHAR | — |
| mar_action_name | VARCHAR | Site-specific |
| mar_action_category | VARCHAR | given, not_given, bolus, other |
| mar_action_group | VARCHAR | administered, not_administered, other |

### patient_assessments

| Column | Type | Allowed Values |
|---|---|---|
| hospitalization_id | VARCHAR | — |
| recorded_dttm | DATETIME | UTC |
| assessment_name | VARCHAR | Site-specific |
| assessment_category | VARCHAR | ~85 categories (see below) |
| assessment_group | VARCHAR | Mobility/Activity, Neurological, Pain, Delirium, Nursing Risk, Sedation/Agitation, Withdrawal, SAT Screen, SAT Delivery, SBT Screen, SBT Delivery, SBT Failure Reason |
| numerical_value | DOUBLE | — |
| categorical_value | VARCHAR | Pass/Fail, Yes/No, etc. |
| text_value | VARCHAR | Free text |

**Assessment groups and categories:**

| Group | Categories |
|---|---|
| Neurological | gcs_eye, gcs_motor, gcs_verbal, gcs_total, APGAR, AVPU, ICANS, TOF |
| Sedation/Agitation | RASS, SAS |
| Delirium | cam_inattention, cam_loc, cam_mental, cam_thinking, cam_total, ICSDC, icsdc_agitation, icsdc_disorientation, icsdc_hallucination, icsdc_inattention, icsdc_loc, icsdc_sleep, icsdc_speech, icsdc_symptoms, icsdc_total |
| Pain | BPS, cpot_body, cpot_facial, cpot_muscle, cpot_total, cpot_vocalization, DVPRS, NRS, NVPS, PAINAD, VAS |
| Mobility/Activity | AM-PAC, AMS, IMS, braden_mobility, braden_activity |
| Nursing Risk | braden_friction, braden_moisture, braden_nutrition, braden_sensory, braden_total, Morse Fall Scale |
| Withdrawal | CIWA, COWS, MINDS, WAT |
| SAT Screen | sat_escalating_sedation, sat_intracranial_pressure, sat_myocardial_ischemia, sat_neuromuscular_blockers, sat_screen_pass_fail, sat_screen_performed, sat_sedative_infusion |
| SAT Delivery | sat_delivery_pass_fail, sat_delivery_performed |
| SBT Screen | sbt_agitation, sbt_inadequate_oxygenation, sbt_intracranial_pressure, sbt_no_spontaneous_effort, sbt_screen_pass_fail, sbt_screen_performed, sbt_vasopressor_use |
| SBT Delivery | sbt_delivery_pass_fail, sbt_delivery_performed |
| SBT Failure Reason | sbt_fail_reason |

### hospital_diagnosis

| Column | Type | Allowed Values |
|---|---|---|
| hospitalization_id | VARCHAR | — |
| diagnosis_code | VARCHAR | ICD-9-CM or ICD-10-CM |
| diagnosis_code_format | VARCHAR | ICD10CM, ICD9CM |
| diagnosis_primary | INT | 1 = primary, 0 = secondary |
| poa_present | INT | 1 = Yes, 0 = No |

### microbiology_culture

| Column | Type | Allowed Values |
|---|---|---|
| patient_id | VARCHAR | — |
| hospitalization_id | VARCHAR | — |
| organism_id | VARCHAR | Links to susceptibility table |
| order_dttm | DATETIME | UTC |
| collect_dttm | DATETIME | UTC |
| result_dttm | DATETIME | UTC |
| fluid_name | VARCHAR | Site-specific |
| fluid_category | VARCHAR | ~45 NIH CDE categories (blood_buffy, bone_marrow, brain, cardiac, catheter_tip, central_nervous_system, feces_stool, joints, kidneys_renal_pelvis_ureters_bladder, meninges_csf, peritoneum, pleural_cavity_fluid, respiratory_tract, respiratory_tract_lower, skin_unspecified, woundsite, etc.) |
| method_name | VARCHAR | Site-specific |
| method_category | VARCHAR | culture, gram_stain, smear |
| organism_name | VARCHAR | Site-specific |
| organism_category | VARCHAR | ~540 genus-species categories |
| organism_group | VARCHAR | ~105 NIH CDE groups (acinetobacter, candida_albicans, enterococcus, escherichia, klebsiella, pseudomonas_wo_cepacia_maltophilia, staphylococcus_coag_pos, streptococcus, no_growth, etc.) |
| lab_loinc_code | VARCHAR | — |

### microbiology_nonculture

| Column | Type | Allowed Values |
|---|---|---|
| patient_id | VARCHAR | — |
| hospitalization_id | VARCHAR | — |
| result_dttm | DATETIME | UTC |
| collect_dttm | DATETIME | UTC |
| order_dttm | DATETIME | UTC |
| fluid_name | VARCHAR | Site-specific |
| fluid_category | VARCHAR | Same NIH CDE as culture |
| method_name | VARCHAR | Site-specific |
| method_category | VARCHAR | pcr |
| micro_order_name | VARCHAR | — |
| organism_category | VARCHAR | clostridium_difficile, sars_cov2, respiratory_syncytial_virus |
| organism_group | VARCHAR | clostridium_difficile, viral_other, respiratory_syncytial_virus |
| result_name | VARCHAR | Site-specific |
| result_category | VARCHAR | detected, not_detected, indeterminate |
| reference_low | DOUBLE | — |
| reference_high | DOUBLE | — |
| result_units | VARCHAR | — |
| lab_loinc_code | VARCHAR | — |

### microbiology_susceptibility

| Column | Type | Allowed Values |
|---|---|---|
| organism_id | VARCHAR | Links to microbiology_culture |
| antimicrobial_name | VARCHAR | Site-specific |
| antimicrobial_category | VARCHAR | ~170 antimicrobial categories |
| sensitivity_name | VARCHAR | MIC or raw result |
| susceptibility_name | VARCHAR | Site-specific |
| susceptibility_category | VARCHAR | susceptible, non_susceptible, indeterminate, NA |

### position

| Column | Type | Allowed Values |
|---|---|---|
| hospitalization_id | VARCHAR | — |
| recorded_dttm | DATETIME | UTC |
| position_name | VARCHAR | Site-specific |
| position_category | VARCHAR | prone, not_prone |

### crrt_therapy

| Column | Type | Allowed Values |
|---|---|---|
| hospitalization_id | VARCHAR | — |
| device_id | VARCHAR | — |
| recorded_dttm | DATETIME | UTC |
| crrt_mode_name | VARCHAR | Site-specific |
| crrt_mode_category | VARCHAR | scuf, cvvh, cvvhd, cvvhdf, avvh |
| dialysis_machine_name | VARCHAR | — |
| blood_flow_rate | FLOAT | 150–350 mL/min |
| pre_filter_replacement_fluid_rate | FLOAT | 0–10000 mL/hr |
| post_filter_replacement_fluid_rate | FLOAT | 0–10000 mL/hr |
| dialysate_flow_rate | FLOAT | 0–10000 mL/hr |
| ultrafiltration_out | FLOAT | 0–500 mL/hr |

### code_status

| Column | Type | Allowed Values |
|---|---|---|
| patient_id | VARCHAR | — |
| start_dttm | DATETIME | UTC |
| code_status_name | VARCHAR | Site-specific |
| code_status_category | VARCHAR | DNR, DNAR, UDNR, DNR/DNI, DNAR/DNI, DNI_only, AND, Full, Presume Full, Other |

### ecmo_mcs

| Column | Type | Allowed Values |
|---|---|---|
| hospitalization_id | VARCHAR | — |
| recorded_dttm | DATETIME | UTC |
| device_name | VARCHAR | Site-specific |
| device_category | VARCHAR | — |
| mcs_group | VARCHAR | — |
| ecmo_configuration_category | VARCHAR | vv, va, va_v, vv_a, etc. |
| control_parameter_name | VARCHAR | Site-specific |
| control_parameter_category | VARCHAR | Under development |
| control_parameter_value | FLOAT | — |
| flow | FLOAT | L/min |
| sweep_set | FLOAT | L/min (ECMO only) |
| fdO2_set | FLOAT | 0–1 (ECMO only) |

### medication_orders

| Column | Type | Allowed Values |
|---|---|---|
| hospitalization_id | VARCHAR | — |
| med_order_id | VARCHAR | — |
| order_start_dttm | DATETIME | UTC |
| order_end_dttm | DATETIME | UTC |
| ordered_dttm | DATETIME | UTC |
| med_name | VARCHAR | — |
| med_category | VARCHAR | Combined continuous + intermittent CDE |
| med_group | VARCHAR | — |
| med_order_status_name | VARCHAR | Site-specific |
| med_order_status_category | VARCHAR | Under development |
| med_route_name | VARCHAR | Site-specific |
| med_dose | DOUBLE | — |
| med_dose_unit | VARCHAR | — |
| med_frequency | VARCHAR | — |
| prn | BOOLEAN | 0 = No, 1 = Yes |

### patient_procedures

| Column | Type | Allowed Values |
|---|---|---|
| hospitalization_id | VARCHAR | — |
| billing_provider_id | VARCHAR | — |
| performing_provider_id | VARCHAR | — |
| procedure_code | VARCHAR | CPT, ICD-10-PCS, or HCPCS |
| procedure_code_format | VARCHAR | CPT, ICD10PCS, HCPCS |
| procedure_billed_dttm | DATETIME | UTC |

### intake_output

| Column | Type | Allowed Values |
|---|---|---|
| hospitalization_id | VARCHAR | — |
| intake_dttm | DATETIME | UTC |
| fluid_name | VARCHAR | — |
| amount | DOUBLE | mL |
| in_out_flag | INT | 0 = Output, 1 = Intake |

### invasive_hemodynamics

| Column | Type | Allowed Values |
|---|---|---|
| hospitalization_id | VARCHAR | — |
| recorded_dttm | DATETIME | UTC |
| measure_name | VARCHAR | Site-specific |
| measure_category | VARCHAR | cvp, ra, rv, pa_systolic, pa_diastolic, pa_mean, pcwp, cardiac_output_thermodilution, cardiac_output_fick |
| measure_value | DOUBLE | mmHg |

### transfusion

| Column | Type | Allowed Values |
|---|---|---|
| hospitalization_id | VARCHAR | — |
| transfusion_start_dttm | DATETIME | UTC |
| transfusion_end_dttm | DATETIME | UTC |
| component_name | VARCHAR | Red Blood Cells, Plasma, Platelets, etc. |
| attribute_name | VARCHAR | Leukocyte Reduced, Irradiated, etc. |
| volume_transfused | DOUBLE | mL |
| volume_units | VARCHAR | mL |
| product_code | VARCHAR | ISBT 128 code |

---

## Additional Tables (Newer in 2.1)

### patient_diagnosis

| Column | Type | Allowed Values |
|---|---|---|
| patient_id | VARCHAR | — |
| hospitalization_id | VARCHAR | — |
| diagnosis_code | VARCHAR | ICD-10-CM |
| diagnosis_code_format | VARCHAR | ICD10CM |
| source_type | VARCHAR | problem_list, medical_history, encounter_dx |
| start_dttm | DATETIME | UTC |
| end_dttm | DATETIME | UTC (NULL if ongoing) |

### provider

| Column | Type | Allowed Values |
|---|---|---|
| hospitalization_id | VARCHAR | — |
| provider_id | VARCHAR | — |
| start_dttm | DATETIME | UTC |
| stop_dttm | DATETIME | UTC |
| provider_role_name | VARCHAR | Site-specific |
| provider_role_category | VARCHAR | Under development |

### place_based_index

| Column | Type | Allowed Values |
|---|---|---|
| hospitalization_id | VARCHAR | — |
| index_name | VARCHAR | ADI, SVI, etc. |
| index_value | DOUBLE | — |
| index_version | VARCHAR | Year |

### clinical_trial

| Column | Type | Allowed Values |
|---|---|---|
| hospitalization_id | VARCHAR | — |
| trial_id | VARCHAR | NCT number |
| trial_name | VARCHAR | — |
| arm_id | VARCHAR | — |
| consent_dttm | DATETIME | UTC |
| randomized_dttm | DATETIME | UTC |
| withdrawal_dttm | DATETIME | UTC |

### key_icu_orders

| Column | Type | Allowed Values |
|---|---|---|
| hospitalization_id | VARCHAR | — |
| order_dttm | DATETIME | UTC |
| order_name | VARCHAR | Site-specific |
| order_category | VARCHAR | pt_evaluation, pt_treat, ot_evaluation, ot_treat (under development) |
| order_status_name | VARCHAR | sent, completed |

### therapy_details

| Column | Type | Allowed Values |
|---|---|---|
| hospitalization_id | VARCHAR | — |
| session_start_dttm | DATETIME | UTC |
| therapy_element_name | VARCHAR | — |
| therapy_element_category | VARCHAR | — |
| therapy_element_value | VARCHAR | — |

---

## SOFA Score Components

| Component | Source Table | Column/Filter |
|---|---|---|
| Respiration (PaO2/FiO2) | labs + respiratory_support | lab_category='po2_arterial' + fio2_set |
| Coagulation | labs | lab_category='platelet_count' |
| Liver | labs | lab_category='bilirubin_total' |
| Cardiovascular | vitals + medication_admin_continuous | vital_category='map' + vasopressor doses |
| CNS | patient_assessments | assessment_category='gcs_total' |
| Renal | labs | lab_category='creatinine' |

---

## mCIDE Reference

For exhaustive category lists (too long to embed), see the mCIDE CSV files at: https://github.com/Common-Longitudinal-ICU-data-Format/CLIF/tree/main/mCIDE
