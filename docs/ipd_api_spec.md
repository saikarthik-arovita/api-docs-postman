# HMS IPD Service — API Reference Specification

**Service:** `services/ipd` | **Base Path:** `/ipd/...`

This document is the complete, authoritative API reference for the In-Patient Department (IPD) module, derived directly from the service implementation.

---

## 🔑 Conventions

### Request Headers (all endpoints)
```
Authorization: Bearer <access_token>
Content-Type: application/json
```

### Standard Response Envelope

**Success**
```json
{ "success": true, "data": { ... } }
```

**Error**
```json
{ "success": false, "error": { "message": "...", "code": 400 } }
```

### Pagination (all list endpoints)
All list endpoints accept `page` (default: `1`) and `page_size` (default: `20`–`50`) query parameters and return:
```json
{ "total": 100, "page": 1, "page_size": 20, "total_pages": 5 }
```

---

## 📌 Complete Route Table

| Method | Path | Description |
|--------|------|-------------|
| **WARD & BED MASTER DATA** |||
| `GET` | `/ipd/wards` | List all wards |
| `GET` | `/ipd/wards/{ward_id}` | Get ward detail |
| `GET` | `/ipd/beds` | List all beds |
| `GET` | `/ipd/beds/availability` | Bed availability summary |
| `GET` | `/ipd/beds/{bed_id}` | Get bed detail |
| `PATCH` | `/ipd/beds/{bed_id}/status` | Update bed status |
| **ADMISSION REQUESTS** |||
| `GET` | `/ipd/admission-requests` | List admission requests |
| `POST` | `/ipd/admission-requests` | Create admission request |
| `GET` | `/ipd/admission-requests/{request_id}` | Get request detail |
| `PATCH` | `/ipd/admission-requests/{request_id}/status` | Update request status |
| **ADMISSIONS** |||
| `GET` | `/ipd/admissions` | List admissions |
| `POST` | `/ipd/admissions` | Create admission |
| `GET` | `/ipd/admissions/{id}` | Get admission detail |
| `GET` | `/ipd/admissions/{id}/clinical-summary` | Get clinical summary |
| `POST` | `/ipd/admissions/{id}/status` | Advance IPD lifecycle status |
| `POST` | `/ipd/admissions/{id}/transfer` | Internal ward/bed transfer |
| **DAILY NOTES** |||
| `POST` | `/ipd/admissions/{id}/notes` | Add daily note |
| `GET` | `/ipd/admissions/{id}/notes` | List daily notes |
| **NURSING** |||
| `POST` | `/ipd/admissions/{id}/nursing/assign` | Assign nurse |
| `GET` | `/ipd/admissions/{id}/nursing` | Get nursing assignments |
| **DISCHARGE** |||
| `POST` | `/ipd/admissions/{id}/discharge/initiate` | Initiate discharge |
| `POST` | `/ipd/admissions/{id}/discharge/complete` | Complete discharge |
| **DAMA** |||
| `GET` | `/ipd/admissions/{id}/dama` | Get DAMA record |
| `POST` | `/ipd/admissions/{id}/dama/initiate` | Initiate DAMA |
| `POST` | `/ipd/admissions/{id}/dama/consent` | Record DAMA consent |
| `POST` | `/ipd/admissions/{id}/dama/approve` | Approve DAMA |
| `POST` | `/ipd/admissions/{id}/dama/complete` | Complete DAMA discharge |
| `POST` | `/ipd/admissions/{id}/dama/cancel` | Cancel DAMA |
| **CLOSURE STATUSES** |||
| `POST` | `/ipd/admissions/{id}/absconded` | Record absconded patient |
| `POST` | `/ipd/admissions/{id}/death` | Record patient death |
| `POST` | `/ipd/admissions/{id}/transfer-out` | Record transfer to external facility |
| `POST` | `/ipd/admissions/{id}/close` | Close case |
| `GET` | `/ipd/admissions/{id}/closure` | Get case closure record |
| **PATIENT** |||
| `GET` | `/ipd/patients/{patient_id}/admissions` | Patient admission history |
| `GET` | `/ipd/patients/{patient_id}/location` | Current patient location |
| **OBSERVATIONS** |||
| `GET` | `/ipd/observations` | List observations |
| `POST` | `/ipd/observations` | Create observation |
| `GET` | `/ipd/observations/{obs_id}` | Get observation detail |
| `POST` | `/ipd/observations/{obs_id}/discharge` | Discharge from observation |
| `POST` | `/ipd/observations/{obs_id}/convert-to-admission` | Convert to full admission |
| **DASHBOARD** |||
| `GET` | `/ipd/dashboard` | Admission stats dashboard |
| `GET` | `/ipd/dashboard/wards` | Ward occupancy dashboard |
| `GET` | `/ipd/dashboard/tracking` | Live patient tracking |
| **OT CONSENT MANAGEMENT** |||
| `GET` | `/ipd/ot-cases/{ot_case_id}/consents` | List consents for OT case |
| `POST` | `/ipd/ot-cases/{ot_case_id}/consents` | Create consents for OT case |
| `PUT` | `/ipd/consents/{consent_id}` | Update consent metadata |
| `POST` | `/ipd/consents/{consent_id}/sign` | Mark consent as signed |
| `GET` | `/ipd/consents/{consent_id}/document` | Get consent document metadata |
| `GET` | `/ipd/consents/{consent_id}/audit` | Get consent audit trail |
| **EMERGENCY BED MANAGEMENT** |||
| `PATCH` | `/ipd/emergency/{visit_id}/bed` | Assign / transfer / release bed for an emergency visit |

---

## 🛏️ SECTION 1 — WARD & BED MASTER DATA

### 1.1 List Wards
`GET /ipd/wards`

| Query Param | Type | Default | Description |
|-------------|------|---------|-------------|
| `is_active` | boolean | `true` | Filter active/inactive wards |
| `ward_type` | string | — | `GENERAL` `SEMI_PRIVATE` `PRIVATE` `ICU` `NICU` `PICU` `OBSERVATION` `DAY_CARE` `ISOLATION` `HDU` `OTHER` |
| `page` | integer | `1` | |
| `page_size` | integer | `50` | |

**Response `200 OK`**
```json
{
  "success": true,
  "data": {
    "wards": [
      {
        "id": "ward-uuid",
        "ward_code": "ICU-A",
        "ward_name": "Intensive Care Unit - Wing A",
        "ward_type": "ICU",
        "gender_restriction": "MIXED",
        "capacity": 10,
        "total_beds": 10,
        "occupied_beds": 4,
        "available_beds": 5,
        "maintenance_beds": 1,
        "is_active": true,
        "created_at": "2026-06-17T10:00:00Z"
      }
    ],
    "total": 8,
    "page": 1,
    "page_size": 50,
    "total_pages": 1
  }
}
```

---

### 1.2 Get Ward Detail
`GET /ipd/wards/{ward_id}`

**Response `200 OK`** — same shape as a single `WardItem` above.

---

### 1.3 List Beds
`GET /ipd/beds`

| Query Param | Type | Default | Description |
|-------------|------|---------|-------------|
| `ward_id` | UUID | — | Filter by ward |
| `status` | string | — | `AVAILABLE` `RESERVED` `OCCUPIED` `MAINTENANCE` |
| `bed_type` | string | — | `STANDARD` `ICU` `NICU` `PICU` `ISOLATION` `SEMI_PRIVATE` `PRIVATE` `DAY_CARE` |
| `page` | integer | `1` | |
| `page_size` | integer | `20` | |

**Response `200 OK`**
```json
{
  "success": true,
  "data": {
    "beds": [
      {
        "id": "bed-uuid",
        "bed_number": "ICU-A-01",
        "ward_id": "ward-uuid",
        "ward_name": "ICU Wing A",
        "ward_type": "ICU",
        "bed_type": "ICU",
        "status": "AVAILABLE",
        "cleaning_status": null,
        "is_active": true,
        "current_patient_id": null,
        "current_patient_name": null,
        "current_admission_id": null,
        "updated_at": "2026-06-17T08:00:00Z"
      }
    ],
    "total": 120,
    "page": 1,
    "page_size": 20,
    "total_pages": 6
  }
}
```

---

### 1.4 Get Bed Availability Summary
`GET /ipd/beds/availability`

| Query Param | Type | Description |
|-------------|------|-------------|
| `ward_id` | UUID | Optional — filter to one ward |

**Response `200 OK`**
```json
{
  "success": true,
  "data": {
    "ward_id": "ward-uuid",
    "ward_name": "General Ward",
    "total": 30,
    "available": 8,
    "occupied": 18,
    "reserved": 2,
    "maintenance": 2,
    "beds": [ ... ]
  }
}
```

---

### 1.5 Get Bed Detail
`GET /ipd/beds/{bed_id}`

**Response `200 OK`** — same shape as a single `BedItem`.

---

### 1.6 Update Bed Status
`PATCH /ipd/beds/{bed_id}/status`

**Permission:** `ipd:beds:manage`

**Request Body**
```json
{
  "status": "MAINTENANCE",
  "reason": "Deep cleaning after discharge"
}
```

| Field | Type | Required | Values |
|-------|------|----------|--------|
| `status` | string | ✅ | `AVAILABLE` `RESERVED` `OCCUPIED` `MAINTENANCE` |
| `reason` | string | — | Max 500 chars |

---

## 📋 SECTION 2 — ADMISSION REQUESTS

Admission requests are pre-admission intake records created from OPD consultations, emergency visits, teleconsultations, or walk-ins. They are reviewed and approved before a full admission is created.

### 2.1 Create Admission Request
`POST /ipd/admission-requests`

**Permission:** `ipd:admissions:create`

**Request Body**
```json
{
  "patient_id": "patient-uuid",
  "source_type": "OPD",
  "source_reference": "opd-visit-uuid",
  "request_reason": "Requires inpatient monitoring for acute chest pain",
  "admission_type": "EMERGENCY",
  "priority": "URGENT",
  "preferred_ward_id": "ward-uuid",
  "preferred_bed_type": "ICU",
  "notes": "Patient allergic to penicillin"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `patient_id` | UUID | ✅ | |
| `source_type` | string | — | `OPD` `EMERGENCY` `TELECONSULTATION` `REFERRAL` `WALK_IN` `DIALYSIS` `ONCOLOGY` `DAY_CARE` |
| `source_reference` | UUID | — | OPD visit ID, emergency visit ID, etc. |
| `request_reason` | string | ✅ | Min 5, max 2000 chars |
| `admission_type` | string | — | `ELECTIVE` `EMERGENCY` `TRANSFER` (default: `ELECTIVE`) |
| `priority` | string | — | `ROUTINE` `URGENT` `EMERGENCY` (default: `ROUTINE`) |
| `preferred_ward_id` | UUID | — | |
| `preferred_bed_type` | string | — | |
| `notes` | string | — | Max 2000 chars |

**Response `201 Created`**
```json
{
  "success": true,
  "data": {
    "id": "request-uuid",
    "patient_id": "patient-uuid",
    "patient_name": "Ravi Kumar",
    "patient_mrn": "MRN-2026-0042",
    "source_type": "OPD",
    "source_reference": "opd-visit-uuid",
    "request_reason": "Requires inpatient monitoring for acute chest pain",
    "admission_type": "EMERGENCY",
    "priority": "URGENT",
    "status": "REQUESTED",
    "preferred_ward_id": "ward-uuid",
    "preferred_bed_type": "ICU",
    "notes": "Patient allergic to penicillin",
    "created_at": "2026-06-17T10:30:00Z"
  }
}
```

---

### 2.2 List Admission Requests
`GET /ipd/admission-requests`

| Query Param | Type | Description |
|-------------|------|-------------|
| `status` | string | `REQUESTED` `UNDER_REVIEW` `READY` `CANCELLED` `EXPIRED` `CONVERTED` |
| `priority` | string | `ROUTINE` `URGENT` `EMERGENCY` |
| `page` | integer | |
| `page_size` | integer | |

---

### 2.3 Get Admission Request
`GET /ipd/admission-requests/{request_id}`

---

### 2.4 Update Admission Request Status
`PATCH /ipd/admission-requests/{request_id}/status`

**Permission:** `ipd:admissions:manage`

**Request Body**
```json
{
  "status": "READY",
  "reason": "Bed confirmed in General Ward"
}
```

| Status | Meaning |
|--------|---------|
| `UNDER_REVIEW` | Being reviewed by admission desk |
| `READY` | Bed assigned, patient can be admitted |
| `CANCELLED` | Request cancelled |
| `EXPIRED` | Request expired without action |

---

## 🏥 SECTION 3 — ADMISSIONS

### 3.1 Create Admission
`POST /ipd/admissions`

**Permission:** `ipd:admissions:create`

**Request Body**
```json
{
  "patient_id": "patient-uuid",
  "admission_request_id": "request-uuid",
  "opd_visit_id": "opd-visit-uuid",
  "ward_id": "ward-uuid",
  "bed_id": "bed-uuid",
  "attending_doctor_id": "doctor-uuid",
  "admission_type": "ELECTIVE",
  "admission_reason": "Post-operative monitoring required after appendectomy",
  "expected_discharge": "2026-06-22",
  "notes": "Patient has latex allergy"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `patient_id` | UUID | ✅ | |
| `ward_id` | UUID | ✅ | |
| `bed_id` | UUID | ✅ | Must be `AVAILABLE` or `RESERVED` |
| `attending_doctor_id` | UUID | ✅ | |
| `admission_reason` | string | ✅ | Min 5, max 2000 chars |
| `admission_request_id` | UUID | — | Links to Phase 3 request |
| `opd_visit_id` | UUID | — | Links from OPD consultation |
| `emergency_visit_id` | UUID | — | Links from emergency |
| `admission_type` | string | — | `ELECTIVE` `EMERGENCY` `TRANSFER` (default: `ELECTIVE`) |
| `expected_discharge` | date | — | `YYYY-MM-DD` |
| `notes` | string | — | Max 2000 chars |

**Response `201 Created`**
```json
{
  "success": true,
  "data": {
    "id": "admission-uuid",
    "admission_number": "IPD-2026-0001",
    "patient_id": "patient-uuid",
    "patient_name": "Ravi Kumar",
    "patient_mrn": "MRN-2026-0042",
    "ward_id": "ward-uuid",
    "ward_name": "General Ward",
    "ward_type": "GENERAL",
    "bed_id": "bed-uuid",
    "bed_number": "GW-12",
    "attending_doctor_id": "doctor-uuid",
    "attending_doctor_name": "Dr. Anjali Sharma",
    "admission_type": "ELECTIVE",
    "admission_reason": "Post-operative monitoring required after appendectomy",
    "expected_discharge": "2026-06-22",
    "status": "ADMITTED",
    "ipd_status": "ADMITTED",
    "admitted_at": "2026-06-17T11:00:00Z",
    "created_at": "2026-06-17T11:00:00Z"
  }
}
```

---

### 3.2 List Admissions
`GET /ipd/admissions`

| Query Param | Type | Description |
|-------------|------|-------------|
| `status` | string | `ADMITTED` `DISCHARGED` `ABSCONDED` `DECEASED` `TRANSFERRED_OUT` `DAMA_DISCHARGED` |
| `ipd_status` | string | `ADMITTED` `UNDER_TREATMENT` `READY_FOR_DISCHARGE` `DISCHARGED` |
| `ward_id` | UUID | Filter by ward |
| `admission_type` | string | `ELECTIVE` `EMERGENCY` `TRANSFER` |
| `attending_doctor_id` | UUID | |
| `date_from` | date | `YYYY-MM-DD` |
| `date_to` | date | `YYYY-MM-DD` |
| `search` | string | Patient name or MRN |
| `page` | integer | |
| `page_size` | integer | |

---

### 3.3 Get Admission Detail
`GET /ipd/admissions/{id}`

Returns the full `AdmissionResponse` object including patient info, ward/bed, lifecycle status, and timestamps.

---

### 3.4 Advance IPD Lifecycle Status
`POST /ipd/admissions/{id}/status`

**Permission:** `ipd:admissions:manage`

Moves the **clinical care plan status** forward through the IPD lifecycle.

```
ADMITTED → UNDER_TREATMENT → READY_FOR_DISCHARGE → DISCHARGED
```

**Request Body**
```json
{
  "status": "UNDER_TREATMENT",
  "notes": "Patient stabilised, treatment plan initiated"
}
```

| Field | Type | Required | Values |
|-------|------|----------|--------|
| `status` | string | ✅ | `ADMITTED` `UNDER_TREATMENT` `READY_FOR_DISCHARGE` `DISCHARGED` |
| `notes` | string | — | Max 1000 chars |

> ⚠️ Only forward transitions are allowed. Transitions are enforced by the state machine.

---

### 3.5 Internal Ward / Bed Transfer
`POST /ipd/admissions/{id}/transfer`

**Permission:** `ipd:admissions:manage`

Moves a patient to a different ward or bed within the same hospital.

**Request Body**
```json
{
  "new_ward_id": "icu-ward-uuid",
  "new_bed_id": "icu-bed-uuid",
  "transfer_reason": "Patient condition deteriorated, requires ICU monitoring",
  "responsible_doctor_id": "doctor-uuid",
  "notes": "Moved from General Ward to ICU"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `new_ward_id` | UUID | ✅ | Destination ward |
| `new_bed_id` | UUID | ✅ | Destination bed (must be `AVAILABLE`) |
| `transfer_reason` | string | ✅ | Min 5, max 1000 chars |
| `responsible_doctor_id` | UUID | — | Defaults to attending doctor |
| `notes` | string | — | Max 2000 chars |

**Response `200 OK`**
```json
{
  "success": true,
  "data": {
    "id": "transfer-uuid",
    "admission_id": "admission-uuid",
    "from_ward_id": "gen-ward-uuid",
    "from_ward_name": "General Ward",
    "from_bed_id": "gen-bed-uuid",
    "from_bed_number": "GW-12",
    "to_ward_id": "icu-ward-uuid",
    "to_ward_name": "ICU Wing A",
    "to_bed_id": "icu-bed-uuid",
    "to_bed_number": "ICU-A-03",
    "transfer_reason": "Patient condition deteriorated, requires ICU monitoring",
    "transfer_type": "ICU",
    "transferred_at": "2026-06-17T14:00:00Z"
  }
}

---

### 3.6 Get Clinical Summary
`GET /ipd/admissions/{id}/clinical-summary`

Returns the clinical summary of the patient's admission, including active prescriptions, pending/completed labs, radiology orders, and cancelled items.

**Response `200 OK`**
```json
{
  "success": true,
  "data": {
    "active_prescriptions": [
      {
        "id": "prescription-uuid",
        "doctor_id": "doctor-uuid",
        "patient_id": "patient-uuid",
        "context_type": "IPD_ADMISSION",
        "context_id": "admission-uuid",
        "source_module": "IPD",
        "encounter_type": "WARD_ROUND",
        "status": "SIGNED",
        "version_no": 2,
        "items": [
          {
            "id": "item-uuid-1",
            "medicine_name": "Pantoprazole 40mg",
            "route": "ORAL",
            "dosage": "1 tablet",
            "frequency": "Once daily before food"
          }
        ]
      }
    ],
    "pending_labs": [
      {
        "id": "lab-item-uuid-1",
        "lab_test_id": "test-uuid-1",
        "test_name": "Complete Blood Count (CBC)",
        "status": "SAMPLE_COLLECTED",
        "notes": "Urgent review required"
      }
    ],
    "completed_labs": [],
    "radiology_orders": [],
    "cancelled_orders": [
      {
        "type": "PRESCRIPTION",
        "id": "prescription-uuid-cancelled",
        "status": "CANCELLED",
        "created_at": "2026-06-17T11:00:00Z",
        "items": []
      }
    ]
  }
}
```
```

---

## 📝 SECTION 4 — DAILY PROGRESS NOTES

### 4.1 Add Daily Note
`POST /ipd/admissions/{id}/notes`

**Permission:** `ipd:notes:create`

**Request Body**
```json
{
  "note_type": "DOCTOR",
  "note_text": "Patient responding well to IV antibiotics. Fever subsided. Plan to continue current regime for 48 hours.",
  "is_confidential": false
}
```

| Field | Type | Required | Values |
|-------|------|----------|--------|
| `note_type` | string | — | `DOCTOR` `NURSE` `PROCEDURE` `ASSESSMENT` `HANDOVER` `OTHER` (default: `DOCTOR`) |
| `note_text` | string | ✅ | Min 5, max 5000 chars |
| `is_confidential` | boolean | — | Default: `false` |

---

### 4.2 List Daily Notes
`GET /ipd/admissions/{id}/notes`

| Query Param | Type | Description |
|-------------|------|-------------|
| `note_type` | string | Filter by type |
| `date` | date | Filter by date `YYYY-MM-DD` |

**Response `200 OK`**
```json
{
  "success": true,
  "data": {
    "admission_id": "admission-uuid",
    "notes": [
      {
        "id": "note-uuid",
        "admission_id": "admission-uuid",
        "note_date": "2026-06-17",
        "note_type": "DOCTOR",
        "note_text": "Patient responding well...",
        "is_confidential": false,
        "recorded_by": "doctor-uuid",
        "recorded_by_name": "Dr. Anjali Sharma",
        "created_at": "2026-06-17T14:30:00Z"
      }
    ],
    "total": 5
  }
}
```

---

## 👩‍⚕️ SECTION 5 — NURSING MANAGEMENT

### 5.1 Assign Nurse
`POST /ipd/admissions/{id}/nursing/assign`

**Permission:** `ipd:nursing:manage`

**Request Body**
```json
{
  "nurse_id": "nurse-uuid",
  "assignment_type": "SHIFT",
  "shift": "MORNING",
  "handover_notes": "Patient has fall risk. Bed rails must remain up.",
  "effective_from": "2026-06-17T06:00:00Z"
}
```

| Field | Type | Required | Values |
|-------|------|----------|--------|
| `nurse_id` | UUID | ✅ | |
| `assignment_type` | string | — | `PRIMARY` `SHIFT` (default: `SHIFT`) |
| `shift` | string | — | `MORNING` `AFTERNOON` `NIGHT` (default: `MORNING`) |
| `handover_notes` | string | — | Max 2000 chars |
| `effective_from` | datetime | — | ISO 8601 UTC |

---

### 5.2 Get Nursing Assignments
`GET /ipd/admissions/{id}/nursing`

**Response `200 OK`**
```json
{
  "success": true,
  "data": {
    "admission_id": "admission-uuid",
    "current_assignments": [
      {
        "id": "assignment-uuid",
        "nurse_id": "nurse-uuid",
        "nurse_name": "Nurse Priya Shetty",
        "assignment_type": "SHIFT",
        "shift": "MORNING",
        "handover_notes": "Patient has fall risk.",
        "effective_from": "2026-06-17T06:00:00Z",
        "effective_to": null,
        "is_active": true
      }
    ],
    "history": [ ... ]
  }
}
```

---

## 🚪 SECTION 6 — DISCHARGE WORKFLOW

Discharge is a two-step process: the doctor initiates it, then it is completed at the desk.

### Discharge Lifecycle
```
ADMITTED → UNDER_TREATMENT → READY_FOR_DISCHARGE  [doctor initiates]
READY_FOR_DISCHARGE → DISCHARGED                   [desk completes]
```

### 6.1 Initiate Discharge
`POST /ipd/admissions/{id}/discharge/initiate`

**Permission:** `ipd:discharge:initiate`

The doctor sets the patient as `READY_FOR_DISCHARGE` and provides a discharge summary.

**Request Body**
```json
{
  "discharge_summary": "Patient recovered from acute appendicitis. Vitals stable. Ready for home discharge.",
  "follow_up_date": "2026-06-25",
  "follow_up_instructions": "Return immediately if fever > 38°C or wound discharge observed.",
  "notes": "Advised soft diet for 1 week"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `discharge_summary` | string | — | Max 10000 chars |
| `follow_up_date` | date | — | Must be in the future |
| `follow_up_instructions` | string | — | Max 2000 chars |
| `notes` | string | — | Max 2000 chars |

---

### 6.2 Complete Discharge
`POST /ipd/admissions/{id}/discharge/complete`

**Permission:** `ipd:discharge:complete`

Finalises discharge: releases the bed, sets admission status to `DISCHARGED`.

**Request Body**
```json
{
  "discharge_summary": "Patient recovered from acute appendicitis. Vitals stable.",
  "follow_up_date": "2026-06-25",
  "follow_up_instructions": "Return immediately if fever > 38°C.",
  "medications_on_discharge": "Tab. Amoxicillin 500mg BD × 5 days\nTab. Paracetamol 650mg SOS",
  "diet_instructions": "Soft diet for 1 week. Avoid spicy food."
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `discharge_summary` | string | ✅ | Min 10, max 10000 chars |
| `follow_up_date` | date | — | Must be in the future |
| `follow_up_instructions` | string | — | Max 2000 chars |
| `medications_on_discharge` | string | — | Max 5000 chars |
| `diet_instructions` | string | — | Max 2000 chars |

**Response `200 OK`**
```json
{
  "success": true,
  "data": {
    "admission_id": "admission-uuid",
    "admission_number": "IPD-2026-0001",
    "patient_id": "patient-uuid",
    "patient_name": "Ravi Kumar",
    "discharge_type": "NORMAL",
    "discharge_summary": "Patient recovered...",
    "follow_up_date": "2026-06-25",
    "admitted_at": "2026-06-17T11:00:00Z",
    "actual_discharge_at": "2026-06-22T10:00:00Z",
    "length_of_stay_days": 5,
    "status": "DISCHARGED",
    "ipd_status": "DISCHARGED"
  }
}
```

---

## ⚠️ SECTION 7 — DAMA WORKFLOW

**Discharge Against Medical Advice (DAMA)** — patient or guardian insists on leaving despite doctor's recommendation.

### DAMA State Machine
```
INITIATED → PENDING_CONSENT → PENDING_APPROVAL → APPROVED → COMPLETED
         ↘            ↘               ↘            ↘
          CANCELLED    CANCELLED       CANCELLED    CANCELLED
```

> If `requires_approval = false`, the flow skips `PENDING_APPROVAL` and goes directly to `APPROVED`.

---

### 7.1 Get DAMA Record
`GET /ipd/admissions/{id}/dama`

---

### 7.2 Initiate DAMA
`POST /ipd/admissions/{id}/dama/initiate`

**Permission:** `ipd:dama:manage`

**Request Body**
```json
{
  "reason": "Patient insisting on leaving due to personal reasons. Risks explained but patient refuses.",
  "clinical_summary": "Patient is mid-treatment for pneumonia. BP stable, fever persisting.",
  "requires_approval": false
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `reason` | string | ✅ | Min 10, max 2000 chars |
| `clinical_summary` | string | — | Max 5000 chars |
| `requires_approval` | boolean | — | If `true`, needs senior approval step before completion. Default: `false` |

---

### 7.3 Record DAMA Consent
`POST /ipd/admissions/{id}/dama/consent`

**Request Body**
```json
{
  "consent_by": "Ramesh Kumar",
  "consent_relation": "SPOUSE",
  "consent_document_key": "s3/dama-consent/2026-06-17/admission-uuid.pdf",
  "risks_explained": true
}
```

| `consent_relation` | Values |
|--------------------|--------|
| | `SELF` `SPOUSE` `PARENT` `GUARDIAN` `OTHER` |

---

### 7.4 Approve DAMA
`POST /ipd/admissions/{id}/dama/approve`

**Request Body**
```json
{ "approval_notes": "Reviewed by Dr. HOD. Patient fully informed of risks." }
```

---

### 7.5 Complete DAMA
`POST /ipd/admissions/{id}/dama/complete`

Finalises the DAMA discharge. Releases the bed. Sets admission status to `DAMA_DISCHARGED`.

**Request Body**
```json
{ "discharge_notes": "Patient left with signed DAMA form." }
```

---

### 7.6 Cancel DAMA
`POST /ipd/admissions/{id}/dama/cancel`

**Request Body**
```json
{ "cancellation_reason": "Patient agreed to continue treatment after counselling." }
```

**Response `200 OK` (all DAMA endpoints)**
```json
{
  "success": true,
  "data": {
    "id": "dama-uuid",
    "admission_id": "admission-uuid",
    "patient_id": "patient-uuid",
    "status": "PENDING_CONSENT",
    "reason": "Patient insisting on leaving...",
    "requires_approval": false,
    "consent_obtained": false,
    "risks_explained": false,
    "approved_by": null,
    "created_at": "2026-06-17T15:00:00Z"
  }
}
```

---

## 🔴 SECTION 8 — CLOSURE STATUSES

These endpoints mark special terminal discharge types. Each releases the bed and closes the admission with the corresponding `status`.

### 8.1 Record Absconded
`POST /ipd/admissions/{id}/absconded`

**Permission:** `ipd:admissions:manage`

**Request Body**
```json
{
  "noticed_at": "2026-06-17T03:00:00Z",
  "last_seen_at": "2026-06-17T01:00:00Z",
  "notes": "Patient found missing during night rounds.",
  "police_informed": true,
  "family_informed": true
}
```

---

### 8.2 Record Death
`POST /ipd/admissions/{id}/death`

**Permission:** `ipd:admissions:manage`

**Request Body**
```json
{
  "time_of_death": "2026-06-17T02:35:00Z",
  "cause_of_death": "Septic shock secondary to pneumonia",
  "death_summary": "Patient deteriorated rapidly despite ICU management.",
  "declared_by_doctor_id": "doctor-uuid",
  "witness_name": "Dr. Sunita Rao",
  "police_informed": false,
  "notes": "Family notified at 03:00 AM"
}
```

> `time_of_death` cannot be in the future.

---

### 8.3 Record Transfer Out (External)
`POST /ipd/admissions/{id}/transfer-out`

**Permission:** `ipd:admissions:manage`

**Request Body**
```json
{
  "destination_facility": "AIIMS Delhi",
  "transfer_reason": "Requires advanced cardiac surgery not available here",
  "transfer_summary": "Patient stable for transfer. All records handed over.",
  "transported_by": "108 Ambulance",
  "transfer_time": "2026-06-17T16:00:00Z",
  "notes": "Referral letter sent with patient"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `destination_facility` | string | ✅ | Min 3, max 500 chars |
| `transfer_reason` | string | ✅ | Min 5, max 2000 chars |
| `transfer_summary` | string | — | Max 5000 chars |
| `transported_by` | string | — | Max 200 chars |
| `transfer_time` | datetime | — | ISO 8601 |
| `notes` | string | — | Max 2000 chars |

---

### 8.4 Close Case
`POST /ipd/admissions/{id}/close`

**Permission:** `ipd:admissions:close`

Run after any discharge type (normal, DAMA, death, absconded, transfer-out) to administratively close the case.

**Request Body**
```json
{
  "billing_cleared": true,
  "closure_notes": "All dues paid. Discharge summary handed to patient."
}
```

---

### 8.5 Get Case Closure
`GET /ipd/admissions/{id}/closure`

**Response `200 OK`**
```json
{
  "success": true,
  "data": {
    "id": "closure-uuid",
    "admission_id": "admission-uuid",
    "patient_id": "patient-uuid",
    "discharge_type": "NORMAL",
    "billing_cleared": true,
    "closure_notes": "All dues paid.",
    "closed_by": "receptionist-uuid",
    "closed_at": "2026-06-22T11:00:00Z"
  }
}
```

---

## 👤 SECTION 9 — PATIENT ENDPOINTS

### 9.1 Patient Admission History
`GET /ipd/patients/{patient_id}/admissions`

Returns all admissions for a patient (active + historical).

---

### 9.2 Patient Current Location
`GET /ipd/patients/{patient_id}/location`

**Response `200 OK`**
```json
{
  "success": true,
  "data": {
    "patient_id": "patient-uuid",
    "patient_name": "Ravi Kumar",
    "admission_id": "admission-uuid",
    "current_location": "ICU Wing A — Bed ICU-A-03",
    "current_ward_id": "ward-uuid",
    "current_ward_name": "ICU Wing A",
    "current_ward_type": "ICU",
    "current_bed_id": "bed-uuid",
    "current_bed_number": "ICU-A-03",
    "location_since": "2026-06-17T14:00:00Z",
    "location_history": [
      {
        "from_ward_name": "General Ward",
        "from_bed_number": "GW-12",
        "to_ward_name": "ICU Wing A",
        "to_bed_number": "ICU-A-03",
        "transfer_reason": "Condition deteriorated",
        "transferred_at": "2026-06-17T14:00:00Z"
      }
    ]
  }
}
```

---

## 👁️ SECTION 10 — OBSERVATIONS

Observation is a short-term monitoring state before a full admission decision is made.

### Observation Lifecycle
```
MONITORING → DISCHARGED         (patient sent home)
MONITORING → CONVERTED_TO_ADMISSION  (patient formally admitted)
```

### 10.1 Create Observation
`POST /ipd/observations`

**Permission:** `ipd:observations:create`

**Request Body**
```json
{
  "patient_id": "patient-uuid",
  "bed_id": "bed-uuid",
  "ward_id": "obs-ward-uuid",
  "attending_doctor_id": "doctor-uuid",
  "reason": "Chest pain. Monitoring for 6 hours before admission decision.",
  "notes": "ECG done. Troponin pending."
}
```

---

### 10.2 List Observations
`GET /ipd/observations`

| Query Param | Description |
|-------------|-------------|
| `status` | `MONITORING` `DISCHARGED` `CONVERTED_TO_ADMISSION` |
| `page` | |
| `page_size` | |

---

### 10.3 Get Observation Detail
`GET /ipd/observations/{obs_id}`

---

### 10.4 Discharge from Observation
`POST /ipd/observations/{obs_id}/discharge`

**Request Body**
```json
{ "discharge_notes": "ECG normal, troponin negative. Patient discharged home." }
```

---

### 10.5 Convert Observation to Admission
`POST /ipd/observations/{obs_id}/convert-to-admission`

**Request Body**
```json
{
  "ward_id": "ward-uuid",
  "bed_id": "bed-uuid",
  "attending_doctor_id": "doctor-uuid",
  "admission_type": "EMERGENCY",
  "admission_reason": "Troponin positive. Formal admission for cardiac monitoring."
}
```

---

## 📊 SECTION 11 — DASHBOARD

### 11.1 Admission Stats Dashboard
`GET /ipd/dashboard`

**Permission:** `dashboard:view`

**Response `200 OK`**
```json
{
  "success": true,
  "data": {
    "tenant_id": "tenant-uuid",
    "date": "2026-06-17",
    "pending_requests": 5,
    "ready_for_admission": 3,
    "admissions_today": 8,
    "occupied_beds": 72,
    "available_beds": 18,
    "maintenance_beds": 6,
    "total_beds": 96,
    "occupancy_pct": 75.0,
    "elective_count": 5,
    "emergency_count": 2,
    "transfer_count": 1,
    "discharged_today": 4,
    "dama_today": 0,
    "deceased_today": 1
  }
}
```

---

### 11.2 Ward Occupancy Dashboard
`GET /ipd/dashboard/wards`

**Permission:** `dashboard:view`

**Response `200 OK`**
```json
{
  "success": true,
  "data": {
    "wards": [
      {
        "ward_id": "ward-uuid",
        "ward_name": "General Ward",
        "ward_type": "GENERAL",
        "total_beds": 30,
        "occupied_beds": 22,
        "available_beds": 6,
        "maintenance_beds": 2,
        "occupancy_pct": 73.3
      },
      {
        "ward_id": "icu-uuid",
        "ward_name": "ICU Wing A",
        "ward_type": "ICU",
        "total_beds": 10,
        "occupied_beds": 9,
        "available_beds": 1,
        "maintenance_beds": 0,
        "occupancy_pct": 90.0
      }
    ],
    "total_beds": 96,
    "total_occupied": 72,
    "total_available": 18,
    "total_maintenance": 6,
    "overall_occupancy_pct": 75.0
  }
}
```

---

### 11.3 Patient Tracking Dashboard
`GET /ipd/dashboard/tracking`

**Permission:** `dashboard:view`

| Query Param | Description |
|-------------|-------------|
| `ward_id` | Filter to one ward |
| `ipd_status` | `ADMITTED` `UNDER_TREATMENT` `READY_FOR_DISCHARGE` |
| `page` | |
| `page_size` | |

**Response `200 OK`**
```json
{
  "success": true,
  "data": {
    "tenant_id": "tenant-uuid",
    "total_patients": 72,
    "ward_patients": 55,
    "icu_patients": 9,
    "ot_patients": 2,
    "observation_patients": 4,
    "day_care_patients": 2,
    "patients": [
      {
        "admission_id": "admission-uuid",
        "admission_number": "IPD-2026-0001",
        "patient_id": "patient-uuid",
        "patient_name": "Ravi Kumar",
        "patient_mrn": "MRN-2026-0042",
        "ward_name": "General Ward",
        "ward_type": "GENERAL",
        "bed_number": "GW-12",
        "attending_doctor_name": "Dr. Anjali Sharma",
        "ipd_status": "UNDER_TREATMENT",
        "admission_type": "ELECTIVE",
        "admitted_at": "2026-06-15T10:00:00Z",
        "length_of_stay_days": 2
      }
    ],
    "page": 1,
    "page_size": 20,
    "total_pages": 4
  }
}
```

---

## 📋 SECTION 13 — OT CONSENT MANAGEMENT

Before a patient can undergo **PAC (Pre-Anesthesia Checkup)**, all mandatory surgical consents must be completed (signed). 

* **Validation Rules**:
  * PAC clearance (`POST /ipd/ot-sessions/{id}/pac`) will be blocked and return a `400 Bad Request` with an error message listing the unsigned mandatory consents if any remain unsigned.
  * **Emergency Override**: The PAC clearance can proceed in emergency overrides when `emergency_override` is set to `true`, along with an `override_reason` and validation signatures/IDs of two distinct doctors (`primary_doctor_id` and `second_doctor_id` referencing `identity.users(id)`).

---

### 13.1 List OT Session Consents
`GET /ipd/ot-cases/{ot_case_id}/consents`

Returns all consent records associated with an OT Case.

**Response `200 OK`**
```json
{
  "success": true,
  "data": [
    {
      "id": "consent-uuid-1",
      "branch_id": "branch-uuid",
      "ot_case_id": "ot-session-uuid",
      "admission_id": "admission-uuid",
      "patient_id": "patient-uuid",
      "consent_code": "SURGICAL",
      "mandatory": true,
      "status": "PENDING",
      "language": "en",
      "version": 1,
      "created_at": "2026-06-25T10:16:14Z",
      "updated_at": "2026-06-25T10:16:14Z"
    }
  ]
}
```

### 13.2 Create OT Session Consents
`POST /ipd/ot-cases/{ot_case_id}/consents`

Creates consent records for the OT Case. Returns all consents.

**Request Body**
```json
{
  "consent_codes": ["SURGICAL", "ANAESTHESIA", "BLOOD_TRANSFUSION", "PHYSICIAN_FITNESS"]
}
```

**Response `201 Created`**
```json
{
  "success": true,
  "data": [
    {
      "id": "consent-uuid-1",
      "ot_case_id": "ot-session-uuid",
      "consent_code": "SURGICAL",
      "mandatory": true,
      "status": "PENDING"
    }
  ]
}
```

### 13.3 Update Consent Metadata
`PUT /ipd/consents/{consent_id}`

Update metadata like language, guardian details, or remarks.

**Request Body**
```json
{
  "language": "en",
  "remarks": "Patient preferred English template",
  "guardian_name": "Ravi Kumar",
  "guardian_relation": "Brother"
}
```

**Response `200 OK`**
```json
{
  "success": true,
  "data": {
    "id": "consent-uuid-1",
    "language": "en",
    "guardian_name": "Ravi Kumar",
    "guardian_relation": "Brother"
  }
}
```

### 13.4 Mark Consent as Signed
`POST /ipd/consents/{consent_id}/sign`

Marks the consent status as SIGNED. Automatically triggers audit entry and PDF document metadata generation.

**Request Body**
```json
{
  "signature_type": "DIGITAL",
  "signed_by": "user-uuid",
  "doctor_id": "doctor-uuid",
  "witness_id": "witness-uuid",
  "remarks": "Signed in person"
}
```

**Response `200 OK`**
```json
{
  "success": true,
  "data": {
    "id": "consent-uuid-1",
    "status": "SIGNED",
    "signed_at": "2026-06-25T10:18:00Z"
  }
}
```

### 13.5 Get Consent Document Metadata
`GET /ipd/consents/{consent_id}/document`

Returns the generated document URL (mock S3 path) and hash. If not signed, returns a draft preview.

**Response `200 OK`**
```json
{
  "success": true,
  "data": {
    "id": "doc-uuid-1",
    "consent_id": "consent-uuid-1",
    "document_url": "https://hms-documents.s3.amazonaws.com/consents/consent_consent-uuid-1.pdf",
    "document_hash": "a9f8e7d6...",
    "created_at": "2026-06-25T10:18:00Z",
    "consent_details": {
      "id": "consent-uuid-1",
      "status": "SIGNED"
    }
  }
}
```

### 13.6 Get Consent Audit History
`GET /ipd/consents/{consent_id}/audit`

Returns the audit timeline of all modifications, prints, signatures, and views.

**Response `200 OK`**
```json
{
  "success": true,
  "data": [
    {
      "id": "audit-uuid-1",
      "consent_id": "consent-uuid-1",
      "action": "SIGNED",
      "performed_by": "user-uuid",
      "performed_at": "2026-06-25T10:18:00Z",
      "remarks": "Consent signed via DIGITAL"
    }
  ]
}
```

---

## 🔁 SECTION 12 — STATUS & LIFECYCLE REFERENCE

### IPD Lifecycle Statuses

| Field | Values | Description |
|-------|--------|-------------|
| `status` | `ADMITTED` | Active inpatient record |
| | `DISCHARGED` | Normal discharge |
| | `DAMA_DISCHARGED` | Discharge against medical advice |
| | `ABSCONDED` | Patient left without notice |
| | `DECEASED` | Patient passed away |
| | `TRANSFERRED_OUT` | Moved to external facility |
| `ipd_status` | `ADMITTED` | Initial state after admission |
| | `UNDER_TREATMENT` | Treatment plan active |
| | `READY_FOR_DISCHARGE` | Doctor has cleared patient |
| | `DISCHARGED` | Fully discharged |

### IPD Lifecycle Flow
```
[Admission Created]
       ↓
   ADMITTED  ──(auto)──►  UNDER_TREATMENT
                               ↓
                     READY_FOR_DISCHARGE   ←── Discharge Initiated
                               ↓
                          DISCHARGED       ←── Discharge Completed
```

### Closure Type Flow
```
ADMITTED ──► DISCHARGED         (normal discharge)
         ──► DAMA_DISCHARGED    (DAMA completed)
         ──► ABSCONDED          (patient ran away)
         ──► DECEASED           (patient died)
         ──► TRANSFERRED_OUT    (sent to external facility)
```

### DAMA Workflow Statuses
```
INITIATED → PENDING_CONSENT → PENDING_APPROVAL → APPROVED → COMPLETED
                                                           ↓
                            CANCELLED ←──────────────────(any active state)
```

### Admission Request Statuses
| Status | Description |
|--------|-------------|
| `REQUESTED` | Newly created, pending review |
| `UNDER_REVIEW` | Admission desk is reviewing |
| `READY` | Bed confirmed, ready to admit |
| `CANCELLED` | Cancelled by staff |
| `EXPIRED` | Not acted upon within time window |
| `CONVERTED` | Successfully converted to admission |

### Bed Statuses
| Status | Description |
|--------|-------------|
| `AVAILABLE` | Ready to assign |
| `RESERVED` | Held for a pending admission |
| `OCCUPIED` | Patient currently in bed |
| `MAINTENANCE` | Undergoing maintenance or deep cleaning |

---

## 🚨 SECTION 13 — EMERGENCY BED MANAGEMENT

Emergency patients arriving at the hospital may be assigned a bay or bed immediately on arrival. Since beds are an IPD-owned resource, all bed assignment, transfer, and release operations for emergency visits are handled by the IPD service.

> The OPD service creates and manages the **emergency visit record** (triage, status, doctor assignment).  
> The IPD service handles all **bed state changes** for those visits.

### 13.1 Assign / Transfer / Release Emergency Bed

`PATCH /ipd/emergency/{visit_id}/bed`

Atomically manages the bed for an emergency visit in a single DB transaction:

| Scenario | What happens |
|----------|--------------|
| `bed_id` provided, different from current | Old bed → `AVAILABLE`. New bed validated (must be `AVAILABLE` and active). New bed → `OCCUPIED`. Visit record updated. |
| `bed_id` omitted or `null` | Current bed → `AVAILABLE`. Visit `assigned_bed_id` cleared. |
| New bed is not `AVAILABLE` | `409 Conflict` — no partial changes. |
| New bed is inactive | `400 Validation Error`. |
| Visit not found | `404 Not Found`. |

#### Path Parameters
| Parameter | Type | Description |
|-----------|------|-------------|
| `visit_id` | UUID | Emergency visit ID (from OPD service `POST /opd/emergency`). |

#### Request Body
| Field | Type | Required? | Description |
|-------|------|-----------|-------------|
| `bed_id` | UUID | Optional | Target bed to assign. Omit or set `null` to release the current bed. |

#### Example — Assign a Bed
```json
{
  "bed_id": "bed-uuid-5678"
}
```

#### Example — Transfer to a Different Bed
```json
{
  "bed_id": "bed-uuid-9999"
}
```

#### Example — Release Bed on Discharge
```json
{
  "bed_id": null
}
```

#### Response `200 OK`
```json
{
  "success": true,
  "message": "Emergency bed assignment updated",
  "data": {
    "visit_id": "emg-visit-uuid-001",
    "previous_bed_id": "bed-uuid-5678",
    "assigned_bed_id": "bed-uuid-9999"
  }
}
```

#### Error Responses
| Code | Scenario |
|------|----------|
| `400` | Bed is inactive. |
| `404` | Visit or bed not found. |
| `409` | Target bed is not `AVAILABLE`. |

#### Workflow Integration
```
[OPD] POST /opd/emergency           → creates visit, initial bed (optional)
[IPD] PATCH /ipd/emergency/{id}/bed → assigns / transfers / releases bed
[OPD] PATCH /opd/emergency/{id}/status → updates triage / clinical status
[IPD] PATCH /ipd/emergency/{id}/bed (bed_id: null) → release on discharge
```

---

*IPD Service — `services/ipd` | Last Updated: June 2026*

