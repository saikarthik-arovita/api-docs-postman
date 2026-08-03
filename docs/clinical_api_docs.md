# HMS — Clinical Service API Documentation

This document covers all specialty-specific dynamic consultation forms, medical history logging, and clinical progress notes (SOAP and daily progress notes) for the HMS **Clinical Service**.

---

## 1. Global Conventions

### Base URL
All requests are sent to the Clinical Service API Gateway stage:
```
https://<api-id>.execute-api.ap-south-1.amazonaws.com/<stage>
```

### Authorization Header
All endpoints require a valid JWT Access Token passed in the `Authorization` header:
```http
Authorization: Bearer <access_token>
```

### Universal Response Envelope
All API responses follow the standard envelope format:

**Success Response (200 OK / 201 Created):**
```json
{
  "success": true,
  "data": { ... },
  "message": "Optional descriptive success message"
}
```

---

## 2. Department Consultation Forms

Used by doctors and specialists to record symptoms, examination findings, clinical assessments, treatment plans, and surgical logistics. Forms validate automatically based on the department associated with the visit (e.g. Dental, Gynecology, Pediatrics, General Medicine, Proctology).

### 2.1 Save Consultation Form
* **Endpoint:** `PUT /clinical/consultations/{consultation_id}/forms/{form_type}`
* **Required Permission:** `clinical:consultation:edit` (authorized roles: `MED-001`/Doctor, `ADM-001`/Admin)
* **Path Parameters:**
  * `consultation_id` (UUID, Mandatory): The ID of the consultation session.
  * `form_type` (String, Mandatory): The type/step of form being saved. Valid values:
    * `CURRENT_SYMPTOMS`
    * `EXAMINATION_FINDINGS`
    * `ASSESSMENT_PLAN`
    * `SURGICAL_LOGISTICS`
* **Request Body:**
  | Field | Type | Required? | Description |
  | :--- | :--- | :--- | :--- |
  | `department_id` | UUID | **Mandatory** | Department UUID (to resolve specialty schemas) |
  | `form_data` | Object | **Mandatory** | Step-specific structured JSON payload matching the target specialty and form_type schema |

* **Example Request:**
```json
{
  "department_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
  "form_data": {
    "chief_complaints": "Acute toothache in lower right molar",
    "duration": "3 days",
    "pain_character": "Sharp, radiating",
    "aggravating_factors": "Cold fluids",
    "past_dental_history": "Root canal on upper left pre-molar 2 years ago",
    "blood_thinners": false
  }
}
```
* **Example Response (200 OK):**
```json
{
  "success": true,
  "message": "Consultation form saved successfully",
  "data": {
    "consultation_id": "c8542c47-f3e5-4ec0-8f24-b9f7e58c32f2",
    "form_type": "CURRENT_SYMPTOMS",
    "version": 1,
    "form_data": {
      "chief_complaints": "Acute toothache in lower right molar",
      "duration": "3 days",
      "pain_character": "Sharp, radiating",
      "aggravating_factors": "Cold fluids",
      "past_dental_history": "Root canal on upper left pre-molar 2 years ago",
      "blood_thinners": false
    },
    "updated_at": "2026-06-23T17:35:00Z"
  }
}
```

---

### 2.2 Get Consultation Form Step
* **Endpoint:** `GET /clinical/consultations/{consultation_id}/forms/{form_type}`
* **Required Permission:** `clinical:consultation:view`

---

### 2.3 Get All Consultation Forms
Retrieves all forms recorded for a given consultation.

* **Endpoint:** `GET /clinical/consultations/{consultation_id}/forms`
* **Required Permission:** `clinical:consultation:view`
* **Example Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "consultation_id": "c8542c47-f3e5-4ec0-8f24-b9f7e58c32f2",
    "forms": [
      {
        "form_type": "CURRENT_SYMPTOMS",
        "form_data": { ... },
        "updated_at": "2026-06-23T17:35:00Z"
      }
    ]
  }
}
---

### 2.4 Get Form Metadata & Specialty Schemas
Fetches the dynamic form JSON Schemas and field option enums for all specialty forms (`CURRENT_SYMPTOMS`, `EXAMINATION_FINDINGS`, `ASSESSMENT_PLAN`, `SURGICAL_LOGISTICS`). Frontend dynamic form renderers use this endpoint to inspect valid fields, field titles, input types, `$defs`, and allowable enum choices per specialty.

* **Endpoint:** `GET /clinical/forms/metadata`
* **Required Permission:** `clinical:consultation:view`
* **Query Parameters:**
  | Parameter | Type | Required? | Description |
  | :--- | :--- | :--- | :--- |
  | `specialty` | String | Optional | Filter by specialty key (e.g. `GYNECOLOGY`, `DENTAL`, `PEDIATRICS`, `PROCTOLOGY`, `GENERAL_MEDICINE`, `CARDIOLOGY`, `NEPHROLOGY`, `NEUROLOGY`, `ORTHOPEDICS`, `ENT`, `DERMATOLOGY`, `ANAESTHESIA`, `PATHOLOGY`, `OPHTHALMOLOGY`, `DEFAULT`). |
  | `department_id` | UUID | Optional | Department UUID to resolve the associated specialty automatically. |

* **Example Request:**
  `GET /clinical/forms/metadata?specialty=GYNECOLOGY`

* **Response Structure:**
  `data` returns a nested map: `form_type -> specialty -> JSON Schema`:
  ```json
  {
    "success": true,
    "code": 200,
    "data": {
      "CURRENT_SYMPTOMS": {
        "GYNECOLOGY": {
          "$defs": {
            "ObstetricHistoryGTPAL": {
              "type": "object",
              "properties": {
                "g": { "type": ["integer", "null"], "title": "G" },
                "p": { "type": ["integer", "null"], "title": "P" },
                "t": { "type": ["integer", "null"], "title": "T" },
                "a": { "type": ["integer", "null"], "title": "A" },
                "l": { "type": ["integer", "null"], "title": "L" }
              }
            }
          },
          "properties": {
            "primary_complaint": { "type": ["array", "null"], "items": { "type": "string" }, "title": "Primary Complaint" },
            "duration_of_complaint": { "type": ["integer", "null"], "title": "Duration Of Complaint" },
            "duration_unit": { "type": ["string", "null"], "enum": ["Days", "Weeks", "Months", "Years"], "title": "Duration Unit" },
            "mode_of_onset": { "type": ["string", "null"], "enum": ["Acute / Sudden", "Gradual / Insidious", "Cyclical / Recurrent"], "title": "Mode Of Onset" },
            "course_of_symptoms": { "type": ["string", "null"], "enum": ["Worsening", "Improving", "Static", "Fluctuating"], "title": "Course Of Symptoms" },
            "menstrual_cycle_regularity": { "type": ["string", "null"], "enum": ["Regular", "Irregular", "Absent (Amenorrhoea)"], "title": "Menstrual Cycle Regularity" },
            "menstrual_flow_amount": { "type": ["string", "null"], "enum": ["Scanty (Spotting only)", "Normal", "Heavy (Passing Clots / Flooding)", "Severe (Requires double protection)"], "title": "Menstrual Flow Amount" },
            "dysmenorrhoea_severity": { "type": ["string", "null"], "enum": ["Absent", "Mild (Manageable without medication)", "Moderate (Requires oral analgesics)", "Severe (Incapacitating, limits daily activities)"], "title": "Dysmenorrhoea Severity" },
            "mode_of_previous_deliveries": { "type": ["array", "null"], "items": { "enum": ["Normal Vaginal Delivery (NVD)", "Assisted Vaginal Delivery (Forceps / Ventouse)", "Lower Segment Caesarean Section (LSCS)"], "type": "string" } },
            "lactation_status": { "type": ["string", "null"], "enum": ["Currently Lactating", "Non-Lactating"] },
            "contraceptive_history": { "type": ["string", "null"], "enum": ["None", "Barrier Methods (Condoms)", "Oral Contraceptive Pills (OCPs)", "Intrauterine Contraceptive Device (IUCD - Cu-T)", "Injectable Contraceptives (DMPA)", "Permanent Sterilization (Tubectomy / Laparoscopic Ligation)"] },
            "sexual_activity_status": { "type": ["string", "null"], "enum": ["Sexually Active", "Not Sexually Active", "Not Disclosed"] }
          },
          "title": "GynecologyCurrentSymptomsFormData",
          "type": "object"
        }
      }
    }
  }
  ```

---

## 3. Patient Medical History

### 3.1 Record History Item
* **Endpoint:** `POST /clinical/patients/{patient_id}/history`
* **Required Permission:** `clinical:history:edit`
* **Request Body:**
  | Field | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `history_type` | String | **Mandatory** | Category of history item | `PAST_MEDICAL`, `FAMILY`, `SURGICAL`, `ALLERGY`, `SOCIAL` |
  | `condition` | String | **Mandatory** | Condition name or allergen | Min 2, max 500 chars |
  | `details` | Object | Optional | Structured details (AllergyDetails format required for `ALLERGY` type) | JSON |
  | `relation` | String | Optional | Affected relation | Max 100 chars |
  | `is_chronic` | Boolean | Optional | Chronic status toggle | Defaults to `false` |
  | `diagnosed_date` | Date | Optional | Date diagnosed | `YYYY-MM-DD` |
  | `severity` | String | Optional | Condition severity | `MILD`, `MODERATE`, `SEVERE`, `CRITICAL` |

* **AllergyDetails Object Structure (Required if `history_type == "ALLERGY"`):**
  * `allergen_type` (String, Optional): e.g. `DRUG`, `FOOD`, `ENVIRONMENTAL`
  * `reaction` (Array of Strings, Mandatory): List of symptoms (e.g. `["Rash", "Hives"]`)
  * `severity` (String, Optional): `MILD`, `MODERATE`, `SEVERE`, `CRITICAL`

* **Example Request:**
```json
{
  "history_type": "ALLERGY",
  "condition": "Penicillin",
  "severity": "SEVERE",
  "details": {
    "allergen_type": "DRUG",
    "reaction": ["Anaphylaxis", "Urticaria"]
  }
}
```
* **Example Response (201 Created):**
```json
{
  "success": true,
  "message": "Medical history details logged successfully",
  "data": {
    "id": "hist-uuid-5555",
    "patient_id": "patient-uuid-1111",
    "history_type": "ALLERGY",
    "condition": "Penicillin",
    "severity": "SEVERE",
    "is_active": true,
    "details": {
      "allergen_type": "DRUG",
      "reaction": ["Anaphylaxis", "Urticaria"]
    },
    "created_at": "2026-06-23T17:40:00Z"
  }
}
```

---

### 3.2 Update Medical History Item
* **Endpoint:** `PATCH /clinical/history/{history_id}`
* **Required Permission:** `clinical:history:edit`
* **Request Body:**
```json
{
  "is_active": false,
  "severity": "MODERATE"
}
```
* **Example Response (200 OK):**
```json
{
  "success": true,
  "message": "Medical history record updated successfully",
  "data": {
    "id": "hist-uuid-5555",
    "is_active": false,
    "severity": "MODERATE"
  }
}
```

---

### 3.3 Delete Medical History Item
* **Endpoint:** `DELETE /clinical/history/{history_id}`
* **Required Permission:** `clinical:history:edit`

---

## 4. Clinical Progress Notes (SOAP / IPD Notes)

Clinical notes capture structured patient-encounter records including subjective reports, objective findings, diagnosis assessments, and plans.

### 4.1 Create Patient Note
* **Endpoint:** `POST /clinical/notes`
* **Required Permission:** `clinical:notes:edit`
* **Request Body:**
  | Field | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `patient_id` | UUID | **Mandatory** | Patient UUID | |
  | `note_type` | String | **Mandatory** | Type of clinical note | `SOAP`, `IPD_DAILY`, `WARD_ROUND`, `OPERATIVE`, `DISCHARGE`, `OTHER` |
  | `priority` | String | Optional | Urgency status | `ROUTINE`, `URGENT`, `STAT` (defaults to `ROUTINE`) |
  | `title` | String | Optional | Note header title | Max 500 chars |
  | `note_text` | String | Optional | Free-form notes | Max 10000 chars (mandatory if not `SOAP`) |
  | `consultation_id`| UUID | Optional | Context links | |
  | `admission_id` | UUID | Optional | Context links | |
  | `department_id` | UUID | Optional | Context links | |
  | `soap_subjective`| Object | Optional | Subjective (S) segment | (Mandatory if `SOAP` note) |
  | `soap_objective` | Object | Optional | Objective (O) segment | |
  | `soap_assessment`| Object | Optional | Assessment (A) segment | |
  | `soap_plan` | Object | Optional | Plan (P) segment | |
  | `tags` | Array | Optional | Categorization tags | Array of strings |

* **Example Request:**
```json
{
  "patient_id": "patient-uuid-1111",
  "note_type": "SOAP",
  "title": "Initial General Medicine Followup",
  "consultation_id": "c8542c47-f3e5-4ec0-8f24-b9f7e58c32f2",
  "soap_subjective": {
    "text": "Patient reports minor cough, subsiding fever.",
    "items": ["Cough", "No active fever"]
  },
  "soap_objective": {
    "text": "Chest clear, vitals normal."
  },
  "soap_assessment": {
    "text": "Recovering from respiratory tract infection"
  },
  "soap_plan": {
    "text": "Discontinue antibiotics. Advise warm water."
  }
}
```
* **Example Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "id": "note-uuid-9999",
    "patient_id": "patient-uuid-1111",
    "doctor_id": "doc-uuid-2222",
    "doctor_name": "Dr. Priya Sharma",
    "note_type": "SOAP",
    "priority": "ROUTINE",
    "title": "Initial General Medicine Followup",
    "soap_subjective": {
      "text": "Patient reports minor cough, subsiding fever.",
      "items": ["Cough", "No active fever"]
    },
    "soap_objective": {
      "text": "Chest clear, vitals normal."
    },
    "soap_assessment": {
      "text": "Recovering from respiratory tract infection"
    },
    "soap_plan": {
      "text": "Discontinue antibiotics. Advise warm water."
    },
    "status": "ACTIVE",
    "created_at": "2026-06-23T17:45:00Z"
  }
}
```

---

### 4.2 Fetch Clinical Note Details
* **Endpoint:** `GET /clinical/notes/{note_id}`
* **Required Permission:** `clinical:notes:view`

---

### 4.3 Update Clinical Note
* **Endpoint:** `PUT /clinical/notes/{note_id}`
* **Required Permission:** `clinical:notes:edit`

---

### 4.4 Delete Clinical Note
* **Endpoint:** `DELETE /clinical/notes/{note_id}`
* **Required Permission:** `clinical:notes:delete`

---

### 4.5 List Notes by Patient, Admission, or Consultation

#### List Patient Notes
* **Endpoint:** `GET /clinical/patients/{patient_id}/clinical-notes`
* **Required Permission:** `clinical:notes:view`

#### List Admission Notes
* **Endpoint:** `GET /clinical/admissions/{admission_id}/clinical-notes`
* **Required Permission:** `clinical:notes:view`

#### List Consultation Notes
* **Endpoint:** `GET /clinical/consultations/{consultation_id}/clinical-notes`
* **Required Permission:** `clinical:notes:view`
