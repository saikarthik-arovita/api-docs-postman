# Case Study: Patient Gynecology Consultation & Custom Form Records

This document presents a comprehensive clinical case study of a patient consulting a gynecologist, mapped to the structured fields of the Gynecology Specialty Consultation Form.

It includes:
1. **Clinical Case Narrative**: The patient's background, presentation, and surgical outcome.
2. **Specialty Consultation Form Records**: The structured representation of the form sections as populated during the visit.
3. **Structured API Payloads**: Exact JSON request payloads compliant with the system's Pydantic schemas for the backend integration (`CURRENT_SYMPTOMS`, `EXAMINATION_FINDINGS`, `ASSESSMENT_PLAN`, and `SURGICAL_LOGISTICS`).

---

## 👩‍⚕️ Clinical Case Summary

* **Patient Name:** Mrs. Shalini Iyer
* **UHID (Unique Health Identifier):** `ARV-2026-90412`
* **Age & Gender:** 34 Years | Female
* **Consulting Specialist:** Dr. Priya Vasudevan, MD, DGO (Senior Obstetrician & Gynecologist)
* **Date & Time of Consultation:** June 30, 2026, 10:15 AM
* **Referral Source:** Referred by General Practitioner (Dr. A. K. Sharma)
* **Clinical Diagnosis:** Uterine Leiomyoma / Intramural Fibroid (`ICD-10: D25.9`) with secondary Menorrhagia and moderate Iron-deficiency Anaemia.
* **Management Course:** Initial medical optimization with antifibrinolytics and oral iron therapy, followed by an elective Laparoscopic Myomectomy.

---

## 📋 Specialty Consultation Form (Populated)

### Section 1: Current Symptoms & History Profile
* **Patient Name:** Mrs. Shalini Iyer
* **UHID:** `ARV-2026-90412`
* **Age:** 34
* **Sex:** Female
* **Consulting Gynecologist:** Dr. Priya Vasudevan
* **OPD Date & Time:** 2026-06-30T10:15:00
* **Referred By:** General Practitioner
* **Primary Complaint:** Abnormal Uterine Bleeding (AUB): Menorrhagia (Heavy Menstrual Bleeding), Pain Profile: Dysmenorrhoea (Severe Menstrual Pain)
* **Duration of Complaint:** 6 Months
* **Mode of Onset:** Gradual / Insidious
* **Course of Symptoms:** Worsening
* **Last Menstrual Period (LMP):** 2026-06-18
* **Menstrual Cycle Duration:** 28 Days
* **Duration of Bleeding:** 9 Days (Normal: 3–5 days)
* **Menstrual Cycle Regularity:** Regular
* **Menstrual Flow Amount:** Heavy (Passing Clots / Flooding)
* **Dysmenorrhoea Severity:** Severe (Incapacitating, limits daily activities)
* **Obstetric History (GTPAL):**
  * Gravidity (G): 2
  * Parity (P): 1
  * Term Deliveries (T): 1
  * Preterm Deliveries (A): 0
  * Living Children (L): 1
* **Mode of Previous Deliveries:** Lower Segment Caesarean Section (LSCS)
* **Obstetric Complications:** None
* **Lactation Status:** Non-Lactating
* **Contraceptive History:** Barrier Methods (Condoms)
* **Sexual Activity Status:** Sexually Active
* **Gynaecological Comorbidities:** Uterine Fibroids (Leiomyomas)
* **Systemic Medical Conditions:** Hypothyroidism / Hyperthyroidism (Controlled on Tab. Levothyroxine)
* **Previous Surgical History:** Previous LSCS (2022)

### Section 2: Clinical Examination Findings & Investigations
* **Vitals Panel:**
  * **Blood Pressure (BP):** 110/70 mmHg
  * **Pulse Rate:** 78 bpm
  * **Temperature:** 98.4 °F
  * **Weight:** 68.0 kg
  * **Height:** 160.0 cm
* **Body Mass Index (BMI):** 26.56 kg/m² (Overweight - Indian Cutoff)
* **General Physical Signs:** Pallor (Conjunctival & Palmar Anaemia)
* **Breast Examination:** Normal / No masses
* **Abdominal Examination:** Palpable Mass Present, Localized Pelvic Tenderness
* **Mass Dimensions:** Firm mass of ~14 weeks gestational size felt in the hypogastric region, relatively mobile, non-tender.
* **Patient Examination Position:** Lithotomy Position
* **External Genitalia Inspection:** Normal
* **Speculum Examination:** Cervix Healthy, moderate clear vaginal discharge, no active bleeding seen today.
* **Bimanual Vaginal Examination (PV):** Uterus Bulky / Enlarged (~14 weeks size), regular outline, restricted mobility, bilateral forniceal tenderness present, no palpable adnexal masses.
* **Per Rectal Examination (PR):** Normal (Rectovaginal septum clear)
* **Urine Pregnancy Test (UPT):** Negative
* **Gynaecological & Blood Labs:** Complete Blood Count (CBC) with Haemoglobin, Thyroid Stimulating Hormone (TSH), Coagulation Profile (PT/INR)
* **Pelvic Imaging Protocol:** Ultrasonography — Transvaginal Scan (TVS) (Findings: Large posterior intramural fibroid measuring 6.8 cm x 5.5 cm distorting the endometrial cavity; endometrial thickness 7.2 mm).
* **Cervical Cancer Screening:** Deferred / Not Required (Pap smear done 1 year ago was Normal)

### Section 3: Clinical Assessment & Management Plan
* **Primary Diagnosis:** `D25.9 — Uterine Leiomyoma, Unspecified (Fibroids)`
* **Diagnosis Certainty Status:** Confirmed (Clinical + USG Evidence)
* **Management Approach Category:** Elective Surgical Management (Operating Theatre Admission)
* **OPD Gynaec Procedures:** None
* **Prescribed Therapeutics:**
  1. **Tab. Tranexamic Acid 500 mg** | Brand: Sysfol-T | Oral | TDS during menstrual flow | 5 Days (for bleeding control)
  2. **Tab. Mefenamic Acid 500 mg** | Brand: Meftal-500 | Oral | BD during menstrual pain | 5 Days (analgesic)
  3. **Tab. Levothyroxine Sodium 50 mcg** | Brand: Thyronorm 50 | Oral | OD (Morning, Empty stomach) | Ongoing (Hypothyroidism maintenance)
  4. **Tab. Ferrous Ascorbate + Folic Acid** | Brand: Autrin | Oral | OD (Post-meals) | 30 Days (for pallor/anaemia correction)
* **Common Gynecology Prescribing Reference:**
  * Non-Hormonal Antifibrinolytics: Tab. Tranexamic Acid 500 mg (TDS during flow)
  * Haematinics & Micronutrients: Tab. Ferrous Ascorbate + Folic Acid (Elemental Iron 100 mg)
* **Referral Destination:** Radiology (for pre-surgical pelvic MRI mapping of the fibroid)
* **Follow-Up Interval:** 2 Weeks (Review with pre-op blood reports and MRI scans)
* **Diet & Lifestyle Counseling:**
  * Iron-Rich Dietary Optimization (Green leafy vegetables, dates, jaggery for Anaemia control)
  * Regular pelvic floor checkups

### Section 4: Surgery, Procedure Order & Operational Logistics
* **Selected Procedure:** Laparoscopic / Open Myomectomy — Fibroid Excision [90 min]
* **Planned Date of Surgery:** 2026-07-15
* **Anaesthesia Modality:** General Anaesthesia (GA) with Endotracheal Intubation
* **Patient Surgical Position:** Lithotomy Position with Trendelenburg Tilt
* **Pre-Anaesthetic Check (PAC) Clearance:** Fit with High-Risk Optimization Required (Due to mild Anaemia Hb: 9.2 g/dL and previous laparotomy scarring)
* **ASA Physical Status Score:** ASA Class II — Mild systemic disease
* **Pre-Surgical Haemoglobin Cutoff Check:** No (Hb < 10 g/dL — Mandates Blood Booking / Correction First)
* **Surgical Safety Checklist:**
  * Written Informed Consent obtained in Vernacular Language
  * Blood / Packed Red Blood Cell (PRBC) Cross-Matching & Booking verified
  * Patient Identification Band matched with UHID Registry
  * Pre-Operative Antibiotic Prophylaxis ordered (Inj. Cefazolin 2g IV)
  * NPO (Nil Per Oral) status confirmed (Minimum 6 hours solids, 2 hours clear fluids)
* **Major Laparoscopic Stack Setup:**
  * High-Definition 3-Chip Laparoscopic Camera & Light Source
  * Carbon Dioxide (CO2) High-Flow Insufflator unit
  * Bipolar Electrocoagulation & Advanced Ultrasonic Shear Device (Harmonic)
  * Morcellator Unit (Required for Laparoscopic Myomectomy)
  * Uterine Manipulator (Makar / Clermont-Ferrand / RUMI Device)
* **Suture Materials Inventory Array:**
  * Vicryl (Polyglactin 910) No. 1 / 0 — For vault closure / rectus closure
  * Barbed Suture V-Loc / Quill 2-0 / 0 — For Laparoscopic Myomectomy uterine wall restoration
  * Monocryl (Poliglecaprone 25) 3-0 / 4-0 — For cosmetic skin closures

---

## 🔗 Technical Integration (API Payloads)

Below are the exact JSON request bodies for saving each stage of this consultation to the `/clinical/consultations/{id}/forms/{form_type}` endpoints.

### 1. Stage: `CURRENT_SYMPTOMS`
**Endpoint:** `PUT /clinical/consultations/f5e3b8a6-89d2-4cf3-a7b2-0dc2a45e90a1/forms/CURRENT_SYMPTOMS`
```json
{
  "department_id": "3d756c7b-5a64-423a-bb9d-d24491359b8a",
  "form_data": {
    "patient_name": "Mrs. Shalini Iyer",
    "uhid_number": "ARV-2026-90412",
    "age": 34,
    "sex": "Female",
    "consulting_gynecologist": "Dr. Priya Vasudevan",
    "opd_date_time": "2026-06-30T10:15:00Z",
    "referred_by": "General Practitioner",
    "primary_complaint": [
      "Abnormal Uterine Bleeding (AUB): Menorrhagia (Heavy Menstrual Bleeding)",
      "Pain Profile: Dysmenorrhoea (Severe Menstrual Pain)"
    ],
    "duration_of_complaint": 6,
    "duration_unit": "Months",
    "mode_of_onset": "Gradual / Insidious",
    "course_of_symptoms": "Worsening",
    "lmp": "2026-06-18",
    "menstrual_cycle_duration": 28,
    "duration_of_bleeding": 9,
    "menstrual_cycle_regularity": "Regular",
    "menstrual_flow_amount": "Heavy (Passing Clots / Flooding)",
    "dysmenorrhoea_severity": "Severe (Incapacitating, limits daily activities)",
    "obstetric_history_gtpal": {
      "g": 2,
      "p": 1,
      "t": 1,
      "a": 0,
      "l": 1
    },
    "mode_of_previous_deliveries": [
      "Lower Segment Caesarean Section (LSCS)"
    ],
    "obstetric_complications": [
      "None"
    ],
    "lactation_status": "Non-Lactating",
    "contraceptive_history": "Barrier Methods (Condoms)",
    "sexual_activity_status": "Sexually Active",
    "gynaecological_comorbidities": [
      "Uterine Fibroids (Leiomyomas)"
    ],
    "systemic_medical_conditions": [
      "Hypothyroidism / Hyperthyroidism"
    ],
    "previous_surgical_history": [
      "Previous LSCS (2022)"
    ]
  }
}
```

### 2. Stage: `EXAMINATION_FINDINGS`
**Endpoint:** `PUT /clinical/consultations/f5e3b8a6-89d2-4cf3-a7b2-0dc2a45e90a1/forms/EXAMINATION_FINDINGS`
```json
{
  "department_id": "3d756c7b-5a64-423a-bb9d-d24491359b8a",
  "form_data": {
    "vitals_panel": {
      "bp": "110/70 mmHg",
      "pulse": 78,
      "temp": 98.4,
      "weight": 68.0,
      "height": 160.0
    },
    "bmi": "26.56 kg/m² (Overweight - Indian Cutoff)",
    "general_physical_signs": [
      "Pallor (Conjunctival / Palmar Anaemia)"
    ],
    "breast_examination": "Normal / No masses",
    "abdominal_examination": [
      "Palpable Mass Present",
      "Localized Pelvic Tenderness"
    ],
    "mass_dimensions": "Firm mass of ~14 weeks gestational size felt in hypogastric region",
    "patient_examination_position": "Lithotomy Position",
    "external_genitalia_inspection": [
      "Normal"
    ],
    "speculum_examination": [
      "Cervix Healthy"
    ],
    "bimanual_vaginal_examination": [
      "Uterus Bulky / Enlarged (Fibroids / Adenomyosis)",
      "Forniceal Tenderness Present"
    ],
    "per_rectal_examination": "Normal",
    "urine_pregnancy_test": "Negative",
    "gynaecological_blood_labs": [
      "Complete Blood Count (CBC) with Haemoglobin",
      "Thyroid Stimulating Hormone (TSH)",
      "Coagulation Profile (PT/INR)"
    ],
    "pelvic_imaging_protocol": "Ultrasonography — Transvaginal Scan (TVS)",
    "cervical_cancer_screening": "Deferred / Not Required"
  }
}
```

### 3. Stage: `ASSESSMENT_PLAN`
**Endpoint:** `PUT /clinical/consultations/f5e3b8a6-89d2-4cf3-a7b2-0dc2a45e90a1/forms/ASSESSMENT_PLAN`
```json
{
  "department_id": "3d756c7b-5a64-423a-bb9d-d24491359b8a",
  "form_data": {
    "primary_diagnosis": "D25.9 — Uterine Leiomyoma, Unspecified (Fibroids)",
    "diagnosis_certainty_status": "Confirmed (Clinical + USG Evidence)",
    "management_approach_category": "Elective Surgical Management (Operating Theatre Admission)",
    "opd_gynaec_procedures": [],
    "therapeutic_drug_entry": [
      {
        "generic_name": "Tranexamic Acid",
        "brand_name": "Sysfol-T 500 mg",
        "dosage_form": "Tablet",
        "route": "Oral",
        "frequency": "TDS during flow",
        "duration": "5 days"
      },
      {
        "generic_name": "Mefenamic Acid",
        "brand_name": "Meftal-500",
        "dosage_form": "Tablet",
        "route": "Oral",
        "frequency": "BD",
        "duration": "5 days"
      },
      {
        "generic_name": "Levothyroxine Sodium",
        "brand_name": "Thyronorm 50 mcg",
        "dosage_form": "Tablet",
        "route": "Oral",
        "frequency": "OD (Empty stomach)",
        "duration": "Ongoing"
      },
      {
        "generic_name": "Ferrous Ascorbate + Folic Acid",
        "brand_name": "Autrin",
        "dosage_form": "Tablet",
        "route": "Oral",
        "frequency": "OD",
        "duration": "30 days"
      }
    ],
    "common_gynecology_prescribing_reference": [
      "Non-Hormonal Antifibrinolytics: Tab. Tranexamic Acid 500 mg (TDS during flow)",
      "Haematinics & Micronutrients: Tab. Ferrous Ascorbate + Folic Acid (Elemental Iron 100 mg)"
    ],
    "referral_destination": "Radiology",
    "follow_up_interval": 2,
    "follow_up_time_unit": "Weeks",
    "diet_lifestyle_counseling": [
      "Iron-Rich Dietary Optimization (Green leafy vegetables, dates, jaggery for Anaemia control)"
    ]
  }
}
```

### 4. Stage: `SURGICAL_LOGISTICS`
**Endpoint:** `PUT /clinical/consultations/f5e3b8a6-89d2-4cf3-a7b2-0dc2a45e90a1/forms/SURGICAL_LOGISTICS`
```json
{
  "department_id": "3d756c7b-5a64-423a-bb9d-d24491359b8a",
  "form_data": {
    "selected_procedure": "Laparoscopic / Open Myomectomy — Fibroid Excision [90 min]",
    "planned_date_of_surgery": "2026-07-15",
    "anaesthesia_modality": "General Anaesthesia (GA) with Endotracheal Intubation",
    "patient_surgical_position": "Lithotomy Position with Trendelenburg Tilt",
    "pre_anaesthetic_check_clearance": "Fit with High-Risk Optimization Required",
    "asa_physical_status_score": "ASA Class II — Mild systemic disease",
    "pre_surgical_haemoglobin_check": "No (Hb < 10 g/dL — Mandates Blood Booking / Correction First)",
    "surgical_safety_checklist": [
      "Written Informed Consent obtained in Vernacular Language",
      "Blood / Packed Red Blood Cell (PRBC) Cross-Matching & Booking verified",
      "Patient Identification Band matched with UHID Registry",
      "Pre-Operative Antibiotic Prophylaxis ordered (Inj. Cefazolin 2g IV)",
      "NPO (Nil Per Oral) status confirmed (Minimum 6 hours solids, 2 hours clear fluids)"
    ],
    "major_laparoscopic_stack_setup": [
      "High-Definition 3-Chip Laparoscopic Camera & Light Source",
      "Carbon Dioxide (CO2) High-Flow Insufflator unit",
      "Bipolar Electrocoagulation & Advanced Ultrasonic Shear Device (Harmonic)",
      "Morcellator Unit (Required for Laparoscopic Myomectomy)",
      "Uterine Manipulator (Makar / Clermont-Ferrand / RUMI Device)"
    ],
    "suture_materials_inventory": [
      "Vicryl (Polyglactin 910) No. 1 / 0 — For vault closure / rectus closure",
      "Barbed Suture V-Loc / Quill 2-0 / 0 — For Laparoscopic Myomectomy uterine wall restoration",
      "Monocryl (Poliglecaprone 25) 3-0 / 4-0 — For cosmetic skin closures"
    ]
  }
}
```
