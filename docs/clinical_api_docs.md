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
