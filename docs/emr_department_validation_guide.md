# HMS — Department EMR Validation Guide

This document acts as the definitive guide to EMR schema validation across all clinical departments in the HMS. It outlines how dynamic consultation forms are resolved, validated, and saved.

---

## 1. How EMR Validation is Resolved

When a clinician saves a form via `PUT /clinical/consultations/{consultation_id}/forms/{form_type}`, the backend:
1. Looks up the department associated with the visit/consultation.
2. Maps the department name to a **specialty prefix** using the rules below:

| Department Name Matches (Case-Insensitive) | Specialty Prefix |
| :--- | :--- |
| `dental` | `DENTAL` |
| `gynec`, `gynae`, `obgyn` | `GYNECOLOGY` |
| `pediat`, `paediat` | `PEDIATRICS` |
| `general physician`, `general medicine`, `gm` | `GENERAL_MEDICINE` |
| `proctology`, `hemorrhoid` | `PROCTOLOGY` |
| `cardio` | `CARDIOLOGY` |
| `nephro`, `nephri` | `NEPHROLOGY` |
| `neuro` | `NEUROLOGY` |
| `ortho`, `orthopaed` | `ORTHOPEDICS` |
| `ent`, `otolaryn` | `ENT` |
| `dermat` | `DERMATOLOGY` |
| `anaesth`, `anesth` | `ANAESTHESIA` |
| `pathol` | `PATHOLOGY` |
| `ophthal`, `opthal`, `optho` | `OPHTHALMOLOGY` |

3. Resolves the schema key by joining the specialty prefix and the form type (e.g., `DENTAL_CURRENT_SYMPTOMS`). If no specialty prefix is resolved, it falls back to the default fallback schemas.

---

## 2. Default Fallback Schemas (General/Fallback)

Used when the department does not match any of the defined specialties.

### 2.1 `CURRENT_SYMPTOMS`
* **Schema Class:** `CurrentSymptomsFormData`
* **Fields:**
  * `bleeding` (Object, Optional):
    * `on_set` (String, Optional)
    * `frequency` (String, Optional)
    * `volume` (String, Optional)
    * `colour` (String, Optional)
  * `pain` (Object, Optional):
    * `location` (String, Optional)
    * `nature` (String, Optional)
    * `severity` (Integer, Optional): Must be between `0` and `10`.
    * `trigger` (String, Optional)
  * `bowel_habits` (Object, Optional):
    * `frequency` (String, Optional)
    * `consistency` (String, Optional)
    * `straining` (Boolean, Optional)
    * `continence` (String, Optional)
    * `prolapse` (String, Optional)
    * `discharge` (String, Optional)
  * `hemorrhoid_grade` (Object, Optional):
    * `grade` (String, Optional): Allowed values: `"I"`, `"II"`, `"III"`, `"IV"`
  * `extra` (Dict, Optional)

### 2.2 `EXAMINATION_FINDINGS`
* **Schema Class:** `ExaminationFindingsFormData`
* **Fields:**
  * `anorectal` (Object, Optional):
    * `hemorrhoids` (String, Optional)
    * `fissure` (String, Optional)
    * `abscess` (String, Optional): Allowed values: `"Absent"`, `"Present"`
    * `external_opening` (String, Optional): Allowed values: `"Normal"`, `"Abnormal"`
    * `anorectal_tenderness` (String, Optional): Allowed values: `"None"`, `"Mild"`, `"Moderate"`, `"Severe"`
  * `dre` (Object, Optional):
    * `sphincter_tone` (String, Optional): Allowed values: `"Normal"`, `"Reduced"`, `"Increased"`
    * `squeeze_pressure` (String, Optional): Allowed values: `"Good"`, `"Reduced"`, `"Absent"`
    * `mass` (String, Optional): Allowed values: `"Not Palpable"`, `"Palpable"`
    * `mass_description` (String, Optional): Max 500 characters.
    * `dre_tenderness` (String, Optional): Allowed values: `"None"`, `"Mild"`, `"Moderate"`, `"Severe"`
  * `extra` (Dict, Optional)

### 2.3 `ASSESSMENT_PLAN`
* **Schema Class:** `AssessmentPlanFormData`
* **Fields:**
  * `clinical_summary` (String, Optional): Max 5000 characters.
  * `management_plan` (Object, Optional):
    * `approach` (String, Optional)
    * `urgency` (String, Optional): Allowed values: `"Selective"`, `"Semi-Urgent"`, `"Urgent"`, `"Emergency"`
  * `further_investigations` (String, Optional): Max 3000 characters.
  * `risks_benefits_consent` (String, Optional): Max 3000 characters.
  * `contingency_plans` (String, Optional): Max 3000 characters.
  * `extra` (Dict, Optional)

---

## 3. Department Specialty Validation Rules

### 3.1 Dental Specialty (`DENTAL`)

#### `DENTAL_CURRENT_SYMPTOMS`
* **Schema Class:** `DentalCurrentSymptomsFormData`
* **Fields:**
  * `patient_name` (String, Optional)
  * `uhid_number` (String, Optional)
  * `age` (Integer, Optional)
  * `sex` (String, Optional): Allowed values: `"M"`, `"F"`, `"T"`
  * `consulting_dentist` (String, Optional): Allowed values: `"General"`, `"Ortho"`, `"Endo"`, `"Pedo"`, `"Periodo"`, `"Prostho"`, `"OMFS"`, `"OMR"`
  * `primary_chief_complaint` (List of Strings, Optional)
  * `duration_of_complaint` (Integer, Optional)
  * `duration_unit` (String, Optional): Allowed values: `"Days"`, `"Weeks"`, `"Months"`, `"Years"`
  * `pain_character` (List of Strings, Optional)
  * `aggravating_factors` (List of Strings, Optional)
  * `past_dental_history` (List of Strings, Optional)
  * `past_medical_history` (List of Strings, Optional)
  * `current_blood_thinners` (String, Optional): Allowed values: `"Yes"`, `"No"`
  * `extra` (Dict, Optional)

#### `DENTAL_EXAMINATION_FINDINGS`
* **Schema Class:** `DentalExaminationFindingsFormData`
* **Fields:**
  * `blood_pressure` (String, Optional)
  * `pulse_rate` (Integer, Optional)
  * `extra_oral_examination` (List of Strings, Optional)
  * `fdi_tooth_numbering_selection` (List of Strings, Optional)
  * `hard_tissue_exam_per_tooth` (Dict of String -> List of Strings, Optional)
  * `soft_tissue_periodontal_findings` (List of Strings, Optional)
  * `diagnostic_radiographs_ordered` (List of Strings, Optional)
  * `extra` (Dict, Optional)

#### `DENTAL_ASSESSMENT_PLAN`
* **Schema Class:** `DentalAssessmentPlanFormData`
* **Fields:**
  * `primary_clinical_diagnosis` (String, Optional)
  * `management_strategy` (String, Optional): Allowed values: `"Preventive"`, `"Restorative"`, `"Endodontic"`, `"Surgical"`, `"Periodontal"`, `"Prosthetic"`, `"Orthodontic"`
  * `prescription_generator` (List of Objects, Optional):
    * `generic_name` (String, Optional)
    * `brand_name` (String, Optional)
    * `dosage` (String, Optional)
    * `route` (String, Optional)
    * `frequency` (String, Optional)
    * `duration` (String, Optional)
  * `common_dental_drugs` (List of Strings, Optional)
  * `oral_hygiene_diet_instructions` (List of Strings, Optional)
  * `extra` (Dict, Optional)

#### `DENTAL_SURGICAL_LOGISTICS`
* **Schema Class:** `DentalSurgicalLogisticsFormData`
* **Fields:**
  * `surgical_intervention_order` (String, Optional): Allowed values: `"Composite Restoration"`, `"Root Canal Treatment (RCT)"`, `"Routine Extraction"`, `"Surgical Disimpaction"`, `"Full Mouth Scaling"`, `"Fixed Crown / Bridge"`, `"Dental Implant Placement"`
  * `anaesthesia_modality` (String, Optional): Allowed values: `"LA"`, `"LA + Sedation"`, `"GA"`
  * `pre_op_fitness_evaluation` (String, Optional): Allowed values: `"Fit"`, `"Medical Clearance Required"`, `"Deferred"`
  * `post_surgical_care_instructions` (List of Strings, Optional)
  * `extra` (Dict, Optional)

---

### 3.2 Gynecology Specialty (`GYNECOLOGY`)

#### `GYNECOLOGY_CURRENT_SYMPTOMS`
* **Schema Class:** `GynecologyCurrentSymptomsFormData`
* **Fields:**
  * `patient_name` (String, Optional)
  * `uhid_number` (String, Optional)
  * `age` (Integer, Optional)
  * `sex` (String, Optional): Allowed values: `"Female"`, `"Transgender"`
  * `consulting_gynecologist` (String, Optional)
  * `opd_date_time` (DateTime, Optional): ISO 8601 format.
  * `referred_by` (String, Optional): Allowed values: `"Self-Referral"`, `"General Practitioner"`, `"Another Specialist"`, `"Emergency Department"`, `"AYUSH Practitioner"`
  * `primary_complaint` (List of Strings, Optional)
  * `duration_of_complaint` (Integer, Optional)
  * `duration_unit` (String, Optional): Allowed values: `"Days"`, `"Weeks"`, `"Months"`, `"Years"`
  * `mode_of_onset` (String, Optional): Allowed values: `"Acute / Sudden"`, `"Gradual / Insidious"`, `"Cyclical / Recurrent"`
  * `course_of_symptoms` (String, Optional): Allowed values: `"Worsening"`, `"Improving"`, `"Static"`, `"Fluctuating"`
  * `lmp` (Date, Optional): `YYYY-MM-DD`
  * `menstrual_cycle_duration` (Integer, Optional)
  * `duration_of_bleeding` (Integer, Optional)
  * `menstrual_cycle_regularity` (String, Optional): Allowed values: `"Regular"`, `"Irregular"`, `"Absent (Amenorrhoea)"`
  * `menstrual_flow_amount` (String, Optional): Allowed values: `"Scanty (Spotting only)"`, `"Normal"`, `"Heavy (Passing Clots / Flooding)"`, `"Severe (Requires double protection)"`
  * `dysmenorrhoea_severity` (String, Optional): Allowed values: `"Absent"`, `"Mild (Manageable without medication)"`, `"Moderate (Requires oral analgesics)"`, `"Severe (Incapacitating, limits daily activities)"`
  * `obstetric_history_gtpal` (Object/String, Optional): Auto-parses from a JSON string or descriptive string (e.g. `"G2 P1 T1 A0 L1"`) to:
    * `g` (Integer, Optional)
    * `p` (Integer, Optional)
    * `t` (Integer, Optional)
    * `a` (Integer, Optional)
    * `l` (Integer, Optional)
  * `mode_of_previous_deliveries` (List of Strings, Optional): Allowed values: `"Normal Vaginal Delivery (NVD)"`, `"Assisted Vaginal Delivery (Forceps / Ventouse)"`, `"Lower Segment Caesarean Section (LSCS)"`
  * `obstetric_complications` (List of Strings, Optional)
  * `lactation_status` (String, Optional): Allowed values: `"Currently Lactating"`, `"Non-Lactating"`
  * `contraceptive_history` (String, Optional): Allowed values: `"None"`, `"Barrier Methods (Condoms)"`, `"Oral Contraceptive Pills (OCPs)"`, `"Intrauterine Contraceptive Device (IUCD - Cu-T)"`, `"Injectable Contraceptives (DMPA)"`, `"Permanent Sterilization (Tubectomy / Laparoscopic Ligation)"`
  * `sexual_activity_status` (String, Optional): Allowed values: `"Sexually Active"`, `"Not Sexually Active"`, `"Not Disclosed"`
  * `gynaecological_comorbidities` (List of Strings, Optional)
  * `systemic_medical_conditions` (List of Strings, Optional)
  * `previous_surgical_history` (List of Strings, Optional)
  * `extra` (Dict, Optional)

#### `GYNECOLOGY_EXAMINATION_FINDINGS`
* **Schema Class:** `GynecologyExaminationFindingsFormData`
* **Fields:**
  * `vitals_panel` (Object, Optional):
    * `bp` (String, Optional)
    * `pulse` (Integer, Optional)
    * `temp` (Float, Optional)
    * `weight` (Float, Optional)
    * `height` (Float, Optional)
  * `bmi` (String, Optional)
  * `general_physical_signs` (List of Strings, Optional)
  * `breast_examination` (String, Optional): Allowed values: `"Normal / No masses"`, `"Fibroadenoma / Palpable Mass"`, `"Nipple Discharge (Galactorrhoea)"`, `"Tenderness / Mastitis"`, `"Deferred / Patient Refused"`
  * `abdominal_examination` (List of Strings, Optional)
  * `mass_dimensions` (String, Optional)
  * `patient_examination_position` (String, Optional): Allowed values: `"Lithotomy Position"`, `"Dorsal Recumbent Position"`
  * `external_genitalia_inspection` (List of Strings, Optional)
  * `speculum_examination` (List of Strings, Optional)
  * `bimanual_vaginal_examination` (List of Strings, Optional)
  * `per_rectal_examination` (String, Optional): Allowed values: `"Not Indicated"`, `"Normal"`, `"Rectovaginal Septum Thickened (Deep Infiltrating Endometriosis)"`, `"Parametrial Infiltration Palpable"`
  * `urine_pregnancy_test` (String, Optional): Allowed values: `"Positive"`, `"Negative"`, `"Not Performed"`
  * `gynaecological_blood_labs` (List of Strings, Optional)
  * `pelvic_imaging_protocol` (String, Optional): Allowed values: `"Ultrasound (USG) Pelvis — Transabdominal"`, `"Ultrasonography — Transvaginal Scan (TVS)"`, `"Pelvic Magnetic Resonance Imaging (MRI)"`
  * `cervical_cancer_screening` (String, Optional): Allowed values: `"Pap Smear Collected"`, `"Liquid-Based Cytology (LBC)"`, `"High-Risk HPV DNA Testing"`, `"Deferred / Not Required"`
  * `extra` (Dict, Optional)

#### `GYNECOLOGY_ASSESSMENT_PLAN`
* **Schema Class:** `GynecologyAssessmentPlanFormData`
* **Fields:**
  * `primary_diagnosis` (String, Optional)
  * `diagnosis_certainty_status` (String, Optional): Allowed values: `"Provisional (Awaiting Scan / Labs)"`, `"Confirmed (Clinical + USG Evidence)"`
  * `management_approach_category` (String, Optional): Allowed values: `"Conservative Management (Observation / Lifestyle Modification)"`, `"Medical Management (Pharmacotherapy Only)"`, `"Combined Medical and Dietary Intervention"`, `"Minor OPD Procedure (No General Anaesthesia)"`, `"Elective Surgical Management (Operating Theatre Admission)"`, `"Emergency Surgical Intervention (Immediate Life-saving Open/Laparoscopic)"`
  * `opd_gynaec_procedures` (List of Strings, Optional)
  * `therapeutic_drug_entry` (List of Objects, Optional):
    * `generic_name` (String, Optional)
    * `brand_name` (String, Optional)
    * `dosage_form` (String, Optional)
    * `route` (String, Optional)
    * `frequency` (String, Optional)
    * `duration` (String, Optional)
  * `common_gynecology_prescribing_reference` (List of Strings, Optional)
  * `referral_destination` (String, Optional): Allowed values: `"General Surgery"`, `"Gynec-Oncology"`, `"Endocrinology"`, `"Clinical Nutritionist / Dietitian"`, `"Infertility / IVF Reproductive Suite"`, `"Radiology"`
  * `follow_up_interval` (Integer, Optional)
  * `follow_up_time_unit` (String, Optional): Allowed values: `"Days"`, `"Weeks"`, `"Months"`, `"As Needed (SOS)"`
  * `diet_lifestyle_counseling` (List of Strings, Optional)
  * `extra` (Dict, Optional)

#### `GYNECOLOGY_SURGICAL_LOGISTICS`
* **Schema Class:** `GynecologySurgicalLogisticsFormData`
* **Fields:**
  * `selected_procedure` (String, Optional)
  * `planned_date_of_surgery` (Date, Optional): `YYYY-MM-DD`
  * `anaesthesia_modality` (String, Optional): Allowed values: `"General Anaesthesia (GA) with Endotracheal Intubation"`, `"Spinal Anaesthesia (Sub-Arachnoid Block — SAB)"`, `"Epidural Anaesthesia"`, `"Monitored Anaesthesia Care (MAC) with Sedation"`, `"Local Anaesthesia (LA) Only"`
  * `patient_surgical_position` (String, Optional): Allowed values: `"Lithotomy Position with Trendelenburg Tilt"`, `"Supine Position"`, `"Lithotomy Position Standard"`
  * `pre_anaesthetic_check_clearance` (String, Optional): Allowed values: `"Fit for Procedure under Proposed Anaesthesia"`, `"Fit with High-Risk Optimization Required"`, `"Unfit / Deferred due to Comorbid Fluctuation"`, `"Fitness Pending (Awaiting Medical / Cardiac Opinion)"`
  * `asa_physical_status_score` (String, Optional): Allowed values: `"ASA Class I — Normal healthy patient"`, `"ASA Class II — Mild systemic disease"`, `"ASA Class III — Severe systemic disease with functional limitation"`, `"ASA Class IV — Constant threat to life"`
  * `pre_surgical_haemoglobin_check` (String, Optional): Allowed values: `"Yes (Hb ≥ 10 g/dL — Acceptable Elective Cutoff)"`, `"No (Hb < 10 g/dL — Mandates Blood Booking / Correction First)"`
  * `surgical_safety_checklist` (List of Strings, Optional)
  * `major_laparoscopic_stack_setup` (List of Strings, Optional)
  * `suture_materials_inventory` (List of Strings, Optional)
  * `extra` (Dict, Optional)

---

### 3.3 Pediatrics Specialty (`PEDIATRICS`)

#### `PEDIATRICS_CURRENT_SYMPTOMS`
* **Schema Class:** `PediatricsCurrentSymptomsFormData`
* **Fields:**
  * `patient_name` (String, Optional)
  * `uhid_number` (String, Optional)
  * `age_dob` (String, Optional)
  * `gender_sex` (String, Optional): Allowed values: `"Male"`, `"Female"`, `"Intersex"`
  * `guardian_name` (String, Optional)
  * `informant_identity` (String, Optional): Allowed values: `"Mother"`, `"Father"`, `"Grandparent"`, `"Legal Guardian"`, `"Sibling"`, `"Caregiver / Maid"`
  * `informant_reliability` (String, Optional): Allowed values: `"Good / Reliable"`, `"Fair / Moderate"`, `"Poor / Questionable"`
  * `opd_consultant_doctor` (String, Optional)
  * `appointment_token` (String, Optional)
  * `opd_visit_date_time` (DateTime, Optional)
  * `primary_chief_complaints` (List of Strings, Optional)
  * `duration_of_complaint` (Integer, Optional)
  * `duration_unit` (String, Optional): Allowed values: `"Hours"`, `"Days"`, `"Weeks"`, `"Months"`
  * `mode_of_symptom_onset` (String, Optional): Allowed values: `"Acute / Sudden"`, `"Sub-acute"`, `"Gradual / Insidious"`, `"Hyper-acute / Paroxysmal"`
  * `progression_course` (String, Optional): Allowed values: `"Worsening / Progressive"`, `"Improving"`, `"Fluctuating / Intermittent"`, `"Static / Unchanged"`
  * `prior_similar_episodes` (String, Optional): Allowed values: `"Yes"`, `"No"`
  * `count_of_prior_episodes` (Integer, Optional)
  * `feeding_history_under_2` (String, Optional): Allowed values: `"Exclusive Breastfeeding (EBF)"`, `"Predominant Breastfeeding"`, `"Mixed Feeding (Breast + Formula)"`, `"Formula Feeding Only"`, `"Complementary Feeding Started (Weaning)"`, `"Top Milk / Cow's Milk Initiated"`
  * `complementary_feeding_quality` (List of Strings, Optional)
  * `developmental_milestones_status` (String, Optional): Allowed values: `"Normal / Appropriate for Age"`, `"Global Developmental Delay (GDD)"`, `"Isolated Motor Delay"`, `"Isolated Speech / Language Delay"`, `"Developmental Regression / Loss of Milestones"`
  * `milestones_domain_specifics` (List of Strings, Optional)
  * `immunization_status` (String, Optional): Allowed values: `"Complete / Fully Immunized for Age"`, `"Partially Immunized / Delayed / Missed Doses"`, `"Completely Unimmunized"`, `"Optional Vaccines Taken"`
  * `missed_vaccinations_checklist` (List of Strings, Optional)
  * `birth_neonatal_history` (List of Strings, Optional)
  * `past_medical_comorbidities` (List of Strings, Optional)
  * `prior_surgical_interventions` (String, Optional)
  * `known_drug_food_allergies` (String, Optional)
  * `family_clinical_history` (List of Strings, Optional)
  * `socio_environmental_profile` (List of Strings, Optional)
  * `extra` (Dict, Optional)

#### `PEDIATRICS_EXAMINATION_FINDINGS`
* **Schema Class:** `PediatricsExaminationFindingsFormData`
* **Fields:**
  * `heart_rate_pulse_rate` (Integer, Optional)
  * `respiratory_rate` (Integer, Optional)
  * `axillary_temperature` (Float, Optional)
  * `spo2` (Integer, Optional)
  * `systolic_bp` (Integer, Optional)
  * `diastolic_bp` (Integer, Optional)
  * `capillary_refill_time` (String, Optional): Allowed values: `"< 2 Seconds (Normal)"`, `"2–3 Seconds (Delayed)"`, `"> 3 Seconds (Prolonged / Shock)"`
  * `weight` (Float, Optional)
  * `length_height` (Float, Optional)
  * `head_circumference` (Float, Optional)
  * `mid_upper_arm_circumference` (Float, Optional)
  * `who_growth_percentiles` (String, Optional)
  * `general_alertness_level` (String, Optional): Allowed values: `"Active and Alert"`, `"Lethargic / Drowsy"`, `"Irritable / Unconsolable"`, `"Obtunded / Comatose"`
  * `nutritional_status_tag` (String, Optional): Allowed values: `"Well Nourished"`, `"Moderately Malnourished (MAM)"`, `"Severely Malnourished (SAM)"`, `"Overweight / Obese"`
  * `hydration_status_evaluation` (String, Optional): Allowed values: `"No Dehydration"`, `"Some Dehydration"`, `"Severe Dehydration"`
  * `classic_clinical_signs_grid` (List of Strings, Optional)
  * `anterior_fontanelle_state` (String, Optional): Allowed values: `"Flat and Normotensive"`, `"Bulging / Tense (Raised ICP)"`, `"Sunken / Depressed (Dehydration)"`, `"Closed"`
  * `dermatological_rash_profile` (List of Strings, Optional)
  * `respiratory_system` (List of Strings, Optional)
  * `cardiovascular_system` (List of Strings, Optional)
  * `gastrointestinal_per_abdomen` (List of Strings, Optional)
  * `central_nervous_system` (List of Strings, Optional)
  * `systemic_examination_notes` (String, Optional)
  * `extra` (Dict, Optional)

#### `PEDIATRICS_ASSESSMENT_PLAN`
* **Schema Class:** `PediatricsAssessmentPlanFormData`
* **Fields:**
  * `primary_diagnostic_summary` (String, Optional)
  * `primary_diagnosis_selector` (String, Optional)
  * `icd_10_code` (String, Optional)
  * `secondary_comorbidities` (String, Optional)
  * `diagnostic_certainty_label` (String, Optional): Allowed values: `"Confirmed / Definitive"`, `"Provisional / Presumptive"`, `"Differential Diagnosis Shortlist"`
  * `overall_management_strategy` (String, Optional): Allowed values: `"Outpatient Medical Management"`, `"Routine Immunization / Preventive Visit"`, `"Emergency Triage Stabilization and Immediate Admission"`, `"Ward / Inpatient Admission Ordered"`, `"Referral to Higher Tertiary Care Sub-specialist"`, `"Home Care / Supportive Observation Plan"`
  * `clinical_urgency_tier` (String, Optional): Allowed values: `"Routine / Elective"`, `"Priority / Urgent (Review within 2 hours)"`, `"Emergency / Resuscitation (Immediate Interventions Needed)"`
  * `prescription_items` (List of Objects, Optional):
    * `drug_name` (String, Optional)
    * `formulation_type` (String, Optional)
    * `target_mg_dosage` (String, Optional)
    * `calculated_volume` (String, Optional)
    * `frequency` (String, Optional)
    * `duration` (String, Optional)
    * `instructions` (String, Optional)
  * `referral_specialty` (String, Optional): Allowed values: `"Pediatric Surgery"`, `"Pediatric Cardiology"`, `"Pediatric Neurology"`, `"Child Development Centre (CDC)"`, `"Pediatric Gastroenterology"`, `"Lactation Consultant"`, `"Clinical Nutritionist / Dietitian"`
  * `reason_for_referral` (String, Optional)
  * `scheduled_follow_up` (Integer, Optional)
  * `follow_up_time_unit` (String, Optional): Allowed values: `"Days"`, `"Weeks"`, `"Months"`, `"Emergency SOS Attendance Only"`
  * `dietary_care_guidance` (List of Strings, Optional)
  * `red_flag_warning_indicators` (List of Strings, Optional)
  * `extra` (Dict, Optional)

#### `PEDIATRICS_SURGICAL_LOGISTICS`
* **Schema Class:** `PediatricsSurgicalLogisticsFormData`
* **Fields:**
  * `procedure_name` (String, Optional)
  * `planned_procedure_date` (Date, Optional)
  * `sedation_anesthesia_modality` (String, Optional): Allowed values: `"No Sedation / Local Topical Gel Only"`, `"Conscious Sedation (Oral Triclofos / Midazolam Drops)"`, `"Intravenous Sedation (Monitored Anesthesia Care)"`, `"General Anesthesia (Endotracheal Intubation / LMA)"`
  * `patient_operating_position` (String, Optional): Allowed values: `"Supine"`, `"Prone"`, `"Lateral Recumbent (For Lumbar Puncture)"`, `"Lithotomy Position"`
  * `surgical_care_setting` (String, Optional): Allowed values: `"OPD Procedure Room"`, `"Daycare Unit"`, `"Operation Theatre Inpatient Admission"`, `"Neonatal / Pediatric ICU"`
  * `estimated_surgery_duration` (String, Optional): Allowed values: `"< 15 Minutes"`, `"15–30 Minutes"`, `"30–60 Minutes"`, `"1–2 Hours"`, `"> 2 Hours"`
  * `parental_informed_consent_status` (List of Strings, Optional)
  * `pediatric_fasting_instructions_npo` (String, Optional): Allowed values: `"NPO for Clear Fluids — 2 Hours Minimum"`, `"NPO for Breast Milk — 4 Hours Minimum"`, `"NPO for Infant Formula — 6 Hours Minimum"`, `"NPO for Solid Meals / Non-human Milk — 6 Hours Minimum"`
  * `pre_anaesthetic_checkup_status` (String, Optional): Allowed values: `"Cleared / Fit for Sedation"`, `"Deferred — Requires Active Medical Optimization"`, `"Pending Pediatric Cardiology / Pulmonology Evaluation"`
  * `asa_physical_status_class` (String, Optional): Allowed values: `"ASA Class I — Normal Healthy Child"`, `"ASA Class II — Child with Mild Systemic Disease"`, `"ASA Class III — Severe Systemic Disease"`, `"ASA Class IV — Severe Systemic Disease with Constant Threat to Life"`
  * `pre_op_diagnostic_checks` (List of Strings, Optional)
  * `pre_operative_safety_checklist` (List of Strings, Optional)
  * `post_sedation_monitoring` (List of Strings, Optional)
  * `post_op_oral_refeeding` (String, Optional): Allowed values: `"Keep NPO until fully awake"`, `"Initiate Sips of Clear Water"`, `"Resume Breastfeeding / Formula Feed"`, `"Soft Age-Appropriate Diet"`
  * `daycare_discharge_readiness` (List of Strings, Optional)
  * `extra` (Dict, Optional)

---

### 3.4 General Medicine Specialty (`GENERAL_MEDICINE`)

#### `GENERAL_MEDICINE_CURRENT_SYMPTOMS`
* **Schema Class:** `GeneralMedicineCurrentSymptomsFormData`
* **Fields:** Includes comprehensive histories for fever, cardiovascular, respiratory, metabolic, gastrointestinal, and social parameters. Notable values:
  * `sex` (String, Optional): `"Male"`, `"Female"`, `"Transgender / Other"`
  * `referred_by` (String, Optional): `"Self"`, `"General Practitioner"`, `"Another Specialist"`, `"Emergency Department"`, `"AYUSH Practitioner"`, `"Online / Telemedicine"`
  * `mode_of_onset` (String, Optional): `"Sudden / Acute"`, `"Gradual / Insidious"`, `"Episodic / Recurrent"`, `"Progressive"`
  * `fever_onset` (String, Optional): `"Sudden High Fever"`, `"Gradual Low-grade Onset"`, `"Remittent"`, `"Intermittent"`, `"Continuous"`
  * `pattern_of_fever` (String, Optional): `"Daily"`, `"Alternate Day"`, `"Every 72 Hours"`, `"No Clear Pattern"`, `"Evening Rise"`, `"Saddle-back"`
  * `breathlessness_grade` (String, Optional): `"Grade 1"`, `"Grade 2"`, `"Grade 3"`, `"Grade 4"`
  * `chest_pain_severity` (Integer, Optional): `0` to `10`.
  * `known_hypertension` (String, Optional): `"Yes — On Treatment"`, `"Yes — Untreated / Newly Detected"`, `"No"`, `"Borderline / White Coat Hypertension"`
  * `known_diabetes` (String, Optional): `"Type 1 Diabetes Mellitus"`, `"Type 2 Diabetes Mellitus"`, `"Gestational Diabetes"`, `"Pre-diabetes"`, `"No"`
  * `tobacco_use` (String, Optional): `"Current Smoker"`, `"Ex-smoker"`, `"Smokeless Tobacco"`, `"Non-user"`
  * `alcohol_consumption` (String, Optional): `"None"`, `"Occasional"`, `"Moderate"`, `"Regular / Daily"`
  * `physical_activity_level` (String, Optional): `"Sedentary"`, `"< 30 Minutes"`, `"30–60 Minutes"`, `"Regular Exercise"`, `"Heavy Manual Work"`
  * `extra` (Dict, Optional)

#### `GENERAL_MEDICINE_EXAMINATION_FINDINGS`
* **Schema Class:** `GeneralMedicineExaminationFindingsFormData`
* **Fields:**
  * `pulse_character` (String, Optional): `"Regular"`, `"Irregular"`, `"Thready / Weak"`, `"Bounding"`
  * `temperature_route` (String, Optional): `"Oral"`, `"Axillary"`, `"Rectal"`, `"Tympanic / Ear"`
  * `bmi_category` (String, Optional): `"Underweight"`, `"Normal"`, `"Overweight"`, `"Obese Grade I"`, `"Obese Grade II"`
  * `general_appearance` (String, Optional): `"Well-nourished and Well-kempt"`, `"Moderately Nourished"`, `"Malnourished / Wasted"`, `"Cachexic"`, `"Acutely Ill-looking / Distressed"`
  * `pallor` (String, Optional): `"Absent"`, `"Mild Pallor"`, `"Moderate"`, `"Severe Pallor"`
  * `icterus` / `cyanosis` / `dehydration_signs` (String, Optional)
  * `clubbing` (String, Optional): `"Absent"`, `"Grade I"`, `"Grade II"`, `"Grade III"`, `"Grade IV"`
  * `oedema` (String, Optional): `"Absent"`, `"Mild / Pedal Oedema"`, `"Pitting Oedema up to Ankle"`, `"Pitting Oedema up to Knee"`, `"Generalised Anasarca"`
  * `cardiovascular_system` (String, Optional): `"S1 S2 Heard"`, `"Murmur Present"`, `"Added Sounds"`, `"Raised Jugular Venous Pressure"`, `"Not Examined"`
  * `extra` (Dict, Optional)

#### `GENERAL_MEDICINE_ASSESSMENT_PLAN`
* **Schema Class:** `GeneralMedicineAssessmentPlanFormData`
* **Fields:**
  * `diagnosis_certainty` (String, Optional): `"Confirmed"`, `"Provisional"`, `"Differential Diagnosis"`
  * `management_approach` (String, Optional): `"Conservative Management"`, `"Medical Management"`, `"Combined Medical + Lifestyle"`, `"Procedure / Investigation Required"`, `"Hospital Admission Required"`, `"Refer to Specialist"`, `"Ayurveda"`, `"Watch and Wait"`
  * `urgency_of_management` (String, Optional): `"Elective / Routine"`, `"Semi-urgent"`, `"Urgent"`, `"Emergency"`
  * `review_after_unit` (String, Optional): `"Days"`, `"Weeks"`, `"Months"`
  * `extra` (Dict, Optional)

---

### 3.5 Proctology Specialty (`PROCTOLOGY`)

#### `PROCTOLOGY_CURRENT_SYMPTOMS`
* **Schema Class:** `ProctologyCurrentSymptomsFormData`
* **Fields:**
  * `bleeding_severity` (String, Optional): `"Spotting / Drops on Tissue Paper"`, `"Dripping into Pan"`, `"Streaming / Gush"`, `"Clots Passed"`
  * `bleeding_relation_to_defaecation` (String, Optional): `"During Defaecation Only"`, `"After Defaecation"`, `"Spontaneous / No Relation"`, `"Both During and After"`
  * `prolapse_grade` (String, Optional): `"Grade I — Bleeding only; no prolapse"`, `"Grade II — Prolapse on straining; reduces spontaneously"`, `"Grade III — Prolapse on straining; requires manual reduction"`, `"Grade IV — Permanently prolapsed; irreducible"`
  * `reducibility` (String, Optional): `"Spontaneously Reducible"`, `"Manually Reducible"`, `"Irreducible / Strangulated"`
  * `pain_duration_after_defaecation` (String, Optional): `"< 30 Minutes"`, `"30 Minutes – 2 Hours"`, `"> 2 Hours"`, `"Constant"`
  * `straining_at_stool` (String, Optional): `"Yes — Regular"`, `"Yes — Occasional"`, `"No"`
  * `faecal_incontinence` (String, Optional): `"Yes — Gas Only"`, `"Yes — Liquid Stool"`, `"Yes — Solid Stool"`, `"No"`
  * `extra` (Dict, Optional)

#### `PROCTOLOGY_EXAMINATION_FINDINGS`
* **Schema Class:** `ProctologyExaminationFindingsFormData`
* **Fields:**
  * `patient_position` (String, Optional): `"Left Lateral (Sims' Position)"`, `"Knee-Chest (Genupectoral) Position"`, `"Lithotomy Position"`, `"Jack-knife / Prone Position"`
  * `prolapse_on_straining` (String, Optional): `"Not Present"`, `"Haemorrhoidal Tissue Prolapses"`, `"Rectal Mucosa Prolapses"`, `"Complete Rectal Prolapse (All Layers)"`
  * `dre_anal_tone` (String, Optional): `"Normal Tone"`, `"Hypertonic / Spasm"`, `"Hypotonic / Lax"`, `"Unable to Examine"`
  * `goodsalls_rule_assessment` (String, Optional): `"Anterior Opening"`, `"Posterior Opening"`, `"Not Applicable"`, `"Unable to Assess"`
  * `proctoscopy_performed` (String, Optional): `"Yes"`, `"No — Patient Refused"`, `"No — Deferred"`, `"No — Done at Outside Centre"`
  * `extra` (Dict, Optional)

#### `PROCTOLOGY_ASSESSMENT_PLAN`
* **Schema Class:** `ProctologyAssessmentPlanFormData`
* **Fields:**
  * `diagnosis_certainty` (String, Optional): `"Confirmed"`, `"Provisional"`, `"Differential Diagnosis"`
  * `management_approach` (String, Optional): `"Conservative Management"`, `"Medical Management"`, `"Combined Medical + Dietary"`, `"Minimally Invasive OPD Procedure"`, `"Surgical Management"`, `"Emergency Surgical Intervention"`, `"Ayurveda"`, `"Refer to Higher Centre"`, `"Watch and Wait"`
  * `urgency_of_management` (String, Optional): `"Selective / Elective"`, `"Semi-urgent"`, `"Urgent"`, `"Emergency"`
  * `extra` (Dict, Optional)

#### `PROCTOLOGY_SURGICAL_LOGISTICS`
* **Schema Class:** `ProctologySurgicalLogisticsFormData`
* **Fields:**
  * `procedure_setting` (String, Optional): `"Outpatient Department (OPD) Procedure"`, `"Day Surgery / Day Care"`, `"Short Stay"`, `"Inpatient / Operating Theatre"`, `"Emergency OT"`
  * `extra` (Dict, Optional)

---

### 3.6 Cardiology Specialty (`CARDIOLOGY`)

#### `CARDIOLOGY_CURRENT_SYMPTOMS`
* **Schema Class:** `CardiologyCurrentSymptomsFormData`
* **Fields:**
  * `dyspnoea_nyha_class` (String, Optional): `"Class I — No Limitation"`, `"Class II — Slight Limitation"`, `"Class III — Marked Limitation"`, `"Class IV — Symptoms at Rest"`
  * `extra` (Dict, Optional)

#### `CARDIOLOGY_EXAMINATION_FINDINGS`
* **Schema Class:** `CardiologyExaminationFindingsFormData`
* **Fields:**
  * `pulse_rhythm_character` (String, Optional): `"Regular"`, `"Irregularly Irregular"`, `"Regularly Irregular"`
  * `jvp` (String, Optional): `"Normal / Not Raised"`, `"Raised / Elevated"`
  * `pedal_oedema` (String, Optional): `"Absent"`, `"Grade 1+"`, `"Grade 2+"`, `"Grade 3+"`
  * `extra` (Dict, Optional)

#### `CARDIOLOGY_ASSESSMENT_PLAN`
* **Schema Class:** `CardiologyAssessmentPlanFormData`
* **Fields:**
  * `primary_diagnosis` (String, Optional): `"I21.9 — Acute Myocardial Infarction"`, `"I25.1 — Atherosclerotic Heart Disease (CAD)"`, `"I50.9 — Heart Failure"`, `"I10 — Essential Hypertension"`, `"I48.91 — Unspecified Atrial Fibrillation"`, `"I35.0 — Nonrheumatic Aortic Stenosis"`
  * `extra` (Dict, Optional)

#### `CARDIOLOGY_SURGICAL_LOGISTICS`
* **Schema Class:** `CardiologySurgicalLogisticsFormData`
* **Fields:**
  * `procedure_urgency_status` (String, Optional): `"Elective"`, `"Urgent"`, `"Emergency"`
  * `access_site_location` (String, Optional): `"Right Radial Artery"`, `"Left Radial Artery"`, `"Femoral Artery"`
  * `extra` (Dict, Optional)

---

### 3.7 Nephrology Specialty (`NEPHROLOGY`)

#### `NEPHROLOGY_CURRENT_SYMPTOMS`
* **Schema Class:** `NephrologyCurrentSymptomsFormData`
* **Fields:**
  * `urine_output_characterization` (String, Optional): `"Urine Volume — Normal Output"`, `"Oliguria — Reduced Volume"`, `"Anuria — Severe Shutdown"`
  * `renal_replacement_therapy_status` (String, Optional): `"RRT History — Conservative"`, `"RRT History — Maintenance Hemodialysis"`, `"RRT History — Peritoneal Dialysis"`, `"RRT History — Post-Renal Transplant"`
  * `extra` (Dict, Optional)

#### `NEPHROLOGY_EXAMINATION_FINDINGS`
* **Schema Class:** `NephrologyExaminationFindingsFormData`
* **Fields:**
  * `pedal_edema_grading` (String, Optional): `"Edema Grade — Grade 0"`, `"Edema Grade — Grade 1+"`, `"Edema Grade — Grade 2+"`, `"Edema Grade — Grade 3+"`
  * `auscultation_chest_findings` (String, Optional): `"Bilateral Clear Air Entry"`, `"Bilateral Basal Crepitations"`
  * `urinary_protein_profile` (String, Optional): `"Urinalysis — Nil / Negative"`, `"Urinalysis — 1+ Proteinuria"`, `"Urinalysis — 2+ Proteinuria"`, `"Urinalysis — 3+ / 4+ Nephrotic-Range Proteinuria"`
  * `extra` (Dict, Optional)

#### `NEPHROLOGY_ASSESSMENT_PLAN`
* **Schema Class:** `NephrologyAssessmentPlanFormData`
* **Fields:**
  * `primary_nephrology_diagnosis` (String, Optional): `"N18.3 — CKD Stage 3"`, `"N18.4 — CKD Stage 4"`, `"N18.5 — ESRD"`, `"N17.9 — AKI"`, `"N04.9 — Nephrotic Syndrome"`, `"N13.3 — Obstructive Uropathy"`, `"I12.0 — Hypertensive CKD"`
  * `extra` (Dict, Optional)

#### `NEPHROLOGY_SURGICAL_LOGISTICS`
* **Schema Class:** `NephrologySurgicalLogisticsFormData`
* **Fields:**
  * `specialty_procedure_selector_matrix` (String, Optional): `"Hemodialysis Session Onset"`, `"Renal Biopsy"`, `"AVF Creation"`, `"Permcath Insertion"`, `"Double-Lumen Catheterization"`, `"CAPD Placement"`, `"Renal Transplantation"`
  * `operative_priority_state` (String, Optional): `"Elective"`, `"Urgent"`, `"Emergency"`
  * `anesthetic_delivery_profile` (String, Optional): `"Local Anesthesia Only"`, `"Local + IV Sedation"`, `"Spinal Anesthesia"`, `"General Anaesthesia"`
  * `extra` (Dict, Optional)

---

### 3.8 Neurology Specialty (`NEUROLOGY`)

#### `NEUROLOGY_EXAMINATION_FINDINGS`
* **Schema Class:** `NeurologyExaminationFindingsFormData`
* **Fields:**
  * `higher_mental_functions` (String, Optional): `"Mental State — Alert"`, `"Mental State — Encephalopathic"`, `"Mental State — Stuporous"`, `"Mental State — Comatose"`
  * `extra` (Dict, Optional)

#### `NEUROLOGY_ASSESSMENT_PLAN`
* **Schema Class:** `NeurologyAssessmentPlanFormData`
* **Fields:**
  * `primary_diagnosis` (String, Optional): `"I63.9 — Cerebral Infarction"`, `"I61.9 — Intracerebral Hemorrhage"`, `"G40.909 — Epilepsy"`, `"G43.909 — Migraine"`, `"G20 — Parkinson's"`, `"G61.0 — GBS"`, `"G35 — MS"`, `"G45.9 — TIA"`
  * `extra` (Dict, Optional)

---

### 3.9 Orthopedics Specialty (`ORTHOPEDICS`)

#### `ORTHOPEDICS_ASSESSMENT_PLAN`
* **Schema Class:** `OrthopedicsAssessmentPlanFormData`
* **Fields:**
  * `primary_diagnosis` (String, Optional): `"M17.9: Osteoarthritis of Knee"`, `"M54.5: Low Back Pain"`, `"M50.2: Cervical Disc Displacement"`, `"S83.2: Tear of Meniscus"`, `"S82.9: Fracture of Lower Leg"`, `"M81.9: Osteoporosis"`, `"M75.0: Adhesive Capsulitis"`
  * `extra` (Dict, Optional)

---

### 3.10 Ophthalmology Specialty (`OPHTHALMOLOGY`)

#### `OPHTHALMOLOGY_CURRENT_SYMPTOMS`
* **Schema Class:** `OphthalmologyCurrentSymptomsFormData`
* **Fields:**
  * `sex` (String, Optional): `"M"`, `"F"`, `"T"`
  * `visual_acuity_complaint` (List of Strings, Optional)
  * `eye_pain_severity` (Integer, Optional)
  * `extra` (Dict, Optional)

#### `OPHTHALMOLOGY_EXAMINATION_FINDINGS`
* **Schema Class:** `OphthalmologyExaminationFindingsFormData`
* **Fields:**
  * `visual_acuity_od` / `visual_acuity_os` (String, Optional)
  * `intraocular_pressure_od` / `intraocular_pressure_os` (Float, Optional)
  * `extra` (Dict, Optional)
