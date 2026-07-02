# OPD Gynecology Consultation Journey: API Specification

This document defines the REST API endpoints and data schemas required to build and integrate the end-to-end patient workflow—from front-desk registration to final gynecological clinical decision-making.

---

## 🚀 Workflow Sequence

```
[POST /patients/register] 
          │ (Creates patient profile, assigns UHID)
          ▼
[POST /opd/appointments] 
          │ (Drafts appointment in PENDING_PAYMENT status)
          ▼
[POST /opd/appointments/{id}/confirm] 
          │ (Records payment, updates status to SCHEDULED, auto-creates OPD visit)
          ▼
[POST /opd/visits/{id}/start] 
          │ (Doctor starts visit; changes status to IN_PROGRESS, drafts clinical notes)
          ▼
[PUT /clinical/consultations/{id}/forms/{form_type}] 
          │ (Saves structured specialty forms: Symptoms, Exams, Assessment, Surgery)
          ▼
[POST /opd/visits/{id}/complete] 
          │ (Closes consultation, sets status to COMPLETED, unlocks billing)
```

---

## 🔒 Shared Security and Tenant Headers

Every API request requires:
* **Header `Authorization`:** `Bearer <JWT_ACCESS_TOKEN>` containing caller identity, roles, and branch permissions.
* **Tenant Isolation:** Automatically resolved by the database routing layer based on the caller's active `branch_id`.

---

## ── 1. PATIENT REGISTRATION ──

### `POST /patients/register`
Creates a permanent demographic profile for a new patient and generates a Unique Health Identifier (UHID).

* **Permission Required:** `patients:create`
* **Content-Type:** `application/json`

#### Request Payload (`RegisterPatientRequest`)
```json
{
  "full_name": "Mrs. Shalini Iyer",
  "phone": "+919999988888",
  "dob": "1992-06-15",
  "age": 34,
  "gender": "FEMALE",
  "blood_group": "O+",
  "address": {
    "line1": "Flat 204, Green Glen Layout",
    "city": "Bengaluru",
    "state": "Karnataka",
    "pincode": "560103"
  },
  "abha_number": "91-1234-5678-9012",
  "abha_address": "shalini@abdm"
}
```

* **Constraints & Validation:**
  * `full_name`: String (2–100 chars). Whitespaces stripped.
  * `phone`: Valid phone digits (validated internally to ensure at least 10 digits). Duplication checks run against existing patients.
  * `gender`: Acceptable inputs: `M`, `F`, `O`, `MALE`, `FEMALE`, `OTHER`, `UNKNOWN` (mapped internally to `MALE`, `FEMALE`, `OTHER`, `UNKNOWN`).
  * `blood_group`: One of `A+`, `A-`, `B+`, `B-`, `AB+`, `AB-`, `O+`, `O-`, `UNKNOWN`.

#### Response Payload (`201 Created`)
```json
{
  "status": "success",
  "message": "Patient profile created successfully",
  "data": {
    "id": "e4b9e28f-7cb2-4752-9da5-3f3cfcd50dfa",
    "uhid": "PAT-2026-1048",
    "full_name": "Mrs. Shalini Iyer",
    "phone": "+919999988888",
    "dob": "1992-06-15",
    "age": 34,
    "gender": "FEMALE",
    "blood_group": "O+",
    "address": {
      "line1": "Flat 204, Green Glen Layout",
      "city": "Bengaluru",
      "state": "Karnataka",
      "pincode": "560103"
    },
    "abha_number": "91-1234-5678-9012",
    "abha_address": "shalini@abdm",
    "created_at": "2026-06-30T10:15:00Z"
  }
}
```

---

## ── 2. CREATE OPD TICKET (DRAFT) ──

### `POST /opd/appointments`
Drafts a preliminary appointment ticket in the system. The appointment is initialized as `PENDING_PAYMENT` until verified.

* **Permission Required:** `appointments:book`
* **Content-Type:** `application/json`

#### Request Payload (`CreateAppointmentRequest`)
```json
{
  "patient_id": "e4b9e28f-7cb2-4752-9da5-3f3cfcd50dfa",
  "doctor_id": "8b9e4a3d-24d1-4cb5-8d5c-d2dc1e6aef55",
  "department_id": "3d756c7b-5a64-423a-bb9d-d24491359b8a",
  "appointment_time": "2026-06-30T11:30:00Z",
  "appointment_type": "NEW",
  "is_emergency": false,
  "priority": 0,
  "notes": "Complaining of severe pelvic pain and heavy menstrual flow."
}
```

* **Constraints & Validation:**
  * `appointment_time`: Must be a timezone-aware ISO string representing a future datetime.
  * `priority`: Value between `0` (routine) and `10` (highest urgency). If `is_emergency` is true, priority defaults to `10`.

#### Response Payload (`201 Created`)
```json
{
  "status": "success",
  "message": "Appointment draft created",
  "data": {
    "id": "f5e3b8a6-89d2-4cf3-a7b2-0dc2a45e90a1",
    "patient_id": "e4b9e28f-7cb2-4752-9da5-3f3cfcd50dfa",
    "doctor_id": "8b9e4a3d-24d1-4cb5-8d5c-d2dc1e6aef55",
    "department_id": "3d756c7b-5a64-423a-bb9d-d24491359b8a",
    "appointment_time": "2026-06-30T11:30:00Z",
    "status": "PENDING_PAYMENT",
    "priority": 0,
    "created_at": "2026-06-30T10:16:30Z"
  }
}
```

---

## ── 3. CONFIRM APPOINTMENT & CHECK-IN ──

### `POST /opd/appointments/{id}/confirm`
Logs fee transaction details, updates status to `SCHEDULED`, and inserts a linked record in `clinical.opd_visits` with state `SCHEDULED`, placing the patient in the queue.

* **Path Parameter:** `id` (UUID of the drafted appointment)
* **Permission Required:** `appointments:edit`
* **Content-Type:** `application/json`

#### Request Payload (`ConfirmAppointmentRequest`)
```json
{
  "payment_status": "PAID",
  "payment_mode": "UPI",
  "amount_paid": 500.00,
  "payment_ref": "TXN9876543210"
}
```

* **Constraints & Validation:**
  * `payment_mode`: Required if `payment_status` is `PAID` (must be `CASH`, `UPI`, or `CARD`).
  * `amount_paid`: Required and must be `>= 0` if `payment_status` is `PAID`.
  * If `payment_status` is `WAIVED`, `payment_mode` is automatically saved as `WAIVED`.

#### Response Payload (`200 OK`)
```json
{
  "status": "success",
  "message": "Payment confirmed and check-in processed",
  "data": {
    "appointment_id": "f5e3b8a6-89d2-4cf3-a7b2-0dc2a45e90a1",
    "opd_visit_id": "d043b1c6-a212-4fb3-bb0d-e2ea79496f8c",
    "token_number": 42,
    "status": "ARRIVED",
    "payment_status": "PAID",
    "amount_paid": 500.00
  }
}
```

---

## ── 4. START CLINICAL CONSULTATION ──

### `POST /opd/visits/{id}/start`
Triggered by the doctor when summoning the patient. Transitions the visit state to `IN_PROGRESS` and initializes an empty draft consultation file.

* **Path Parameter:** `id` (UUID of the `opd_visit_id` returned in the check-in confirmation)
* **Permission Required:** `appointments:edit`
* **Content-Type:** `application/json`

#### Request Payload (`StartConsultationRequest`)
```json
{
  "chief_complaint": "Heavy menstrual bleeding with large clots and severe menstrual cramping."
}
```

#### Response Payload (`200 OK`)
```json
{
  "status": "success",
  "message": "Consultation initialized",
  "data": {
    "visit_id": "d043b1c6-a212-4fb3-bb0d-e2ea79496f8c",
    "consultation_id": "a90b4d44-c78d-4e92-96c2-019d3fcf441e",
    "status": "IN_PROGRESS",
    "started_at": "2026-06-30T10:45:00Z"
  }
}
```

---

## ── 5. SAVE GYNECOLOGY SPECIALTY FORM STEPS ──

### `PUT /clinical/consultations/{id}/forms/{form_type}`
Saves or updates a specific form step within a clinical consultation. The system maps the payload based on the combination of department ID and the form type.

* **Path Parameters:**
  * `id`: The `consultation_id` (UUID) returned in the start consultation response.
  * `form_type`: The specific workflow step. Acceptable values:
    * `CURRENT_SYMPTOMS` (mapped to `GYNECOLOGY_CURRENT_SYMPTOMS`)
    * `EXAMINATION_FINDINGS` (mapped to `GYNECOLOGY_EXAMINATION_FINDINGS`)
    * `ASSESSMENT_PLAN` (mapped to `GYNECOLOGY_ASSESSMENT_PLAN`)
    * `SURGICAL_LOGISTICS` (mapped to `GYNECOLOGY_SURGICAL_LOGISTICS`)
* **Permission Required:** `consultations:edit`
* **Content-Type:** `application/json`

#### Generic Wrapping Request Payload (`SaveConsultationFormRequest`)
```json
{
  "department_id": "3d756c7b-5a64-423a-bb9d-d24491359b8a",
  "form_data": {
     "//": "Step-specific JSON data"
  }
}
```

---

### Step A: `CURRENT_SYMPTOMS` Form Data
* **Endpoint:** `PUT /clinical/consultations/a90b4d44-c78d-4e92-96c2-019d3fcf441e/forms/CURRENT_SYMPTOMS`

#### Request Payload
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

---

### Step B: `EXAMINATION_FINDINGS` Form Data
* **Endpoint:** `PUT /clinical/consultations/a90b4d44-c78d-4e92-96c2-019d3fcf441e/forms/EXAMINATION_FINDINGS`

#### Request Payload
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

---

### Step C: `ASSESSMENT_PLAN` Form Data
* **Endpoint:** `PUT /clinical/consultations/a90b4d44-c78d-4e92-96c2-019d3fcf441e/forms/ASSESSMENT_PLAN`

#### Request Payload
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

---

### Step D: `SURGICAL_LOGISTICS` Form Data
* **Endpoint:** `PUT /clinical/consultations/a90b4d44-c78d-4e92-96c2-019d3fcf441e/forms/SURGICAL_LOGISTICS`

#### Request Payload
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

#### Shared Response Payload for Forms Saving (`200 OK`)
```json
{
  "status": "success",
  "message": "Form 'CURRENT_SYMPTOMS' saved",
  "data": {
    "id": "9c1b4ea5-c60e-4361-bd8c-8be9d52b1156",
    "consultation_id": "a90b4d44-c78d-4e92-96c2-019d3fcf441e",
    "department_id": "3d756c7b-5a64-423a-bb9d-d24491359b8a",
    "form_type": "CURRENT_SYMPTOMS",
    "form_data": {
      "//": "Echoes back the stored, validated JSON block"
    },
    "updated_at": "2026-06-30T10:49:15Z",
    "updated_by": "6a9e4d3a-24a1-4cb5-8d5c-d2dc1e6aef00"
  }
}
```

---

## ── 6. COMPLETE CONSULTATION (DISCHARGE) ──

### `POST /opd/visits/{id}/complete`
Closes the OPD visit and sets status metrics to `COMPLETED`. This endpoint starts a transaction that finalizes the consultation notes and triggers the release of the billing locks.

* **Path Parameter:** `id` (UUID of the `opd_visit_id`)
* **Permission Required:** `appointments:edit`
* **Content-Type:** `application/json`

#### Request Payload (`CompleteConsultationRequest`)
```json
{
  "decision": "SURGERY",
  "diagnosis": "Uterine Fibroids (Leiomyomas)",
  "follow_up_date": "2026-07-14",
  "follow_up_notes": "Review with preoperative blood work, PAC clearances, and pelvic MRI scan mapping."
}
```

* **Constraints & Validation:**
  * `decision`: Must be one of `MEDICAL`, `SURGERY`, `PROCEDURE`, `FOLLOW_UP`, `ADMIT`, `REFERRAL`, `OTHER`.
  * `follow_up_date`: Required if `decision` is set to `FOLLOW_UP`. Must be in the future.

#### Response Payload (`200 OK`)
```json
{
  "status": "success",
  "message": "Consultation finalized. Billing lock released.",
  "data": {
    "visit_id": "d043b1c6-a212-4fb3-bb0d-e2ea79496f8c",
    "appointment_id": "f5e3b8a6-89d2-4cf3-a7b2-0dc2a45e90a1",
    "status": "COMPLETED",
    "decision": "SURGERY",
    "completed_at": "2026-06-30T11:05:00Z"
  }
}
```
