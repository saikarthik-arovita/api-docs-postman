# HMS — Nurse Service API Documentation

This document covers all nurse staffing shift assignments, ward handover logs, clinical observations tracking, and patient stay conversions for the HMS **Nurse Service** (integrated with inpatient/IPD modules).

---

## 1. Global Conventions

### Base URL
All requests are sent to the Inpatient/IPD stage:
```
https://<api-id>.execute-api.ap-south-1.amazonaws.com/<stage>
```

### Authorization Header
All endpoints require a valid JWT Access Token passed in the `Authorization` header:
```http
Authorization: Bearer <access_token>
```

---

## 2. Nursing Assignments & Shifts

### 2.1 Assign Nurse to Admission
Binds a nurse to an admitted patient for a shift.

* **Endpoint:** `POST /ipd/admissions/{admission_id}/nursing/assign`
* **Required Permission:** `ipd:nursing:manage` (authorized roles: Head Nurse `HNR-001`, Admin `ADM-001`)
* **Request Body:**
  | Field | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `nurse_id` | UUID | **Mandatory** | Nurse profile user UUID | |
  | `assignment_type` | String | Optional | Care role category | `PRIMARY`, `SHIFT` (defaults to `SHIFT`) |
  | `shift` | String | Optional | Shift period | `MORNING`, `AFTERNOON`, `NIGHT` (defaults to `MORNING`) |
  | `handover_notes` | String | Optional | Patient care notes (e.g. fall risks) | Max 2000 chars |
  | `effective_from` | DateTime | Optional | Timestamp | ISO 8601 UTC |

* **Example Request:**
```json
{
  "nurse_id": "nurse-uuid-8888",
  "shift": "MORNING",
  "handover_notes": "Patient is a fall risk. Maintain bed rails raised."
}
```
* **Example Response (200 OK):**
```json
{
  "success": true,
  "message": "Nurse assigned successfully",
  "data": {
    "admission_id": "admit-uuid-1111",
    "nurse_id": "nurse-uuid-8888",
    "shift": "MORNING",
    "is_active": true
  }
}
```

---

### 2.2 Get Patient Nursing Assignments
* **Endpoint:** `GET /ipd/admissions/{admission_id}/nursing`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "admission_id": "admit-uuid-1111",
    "current_assignments": [
      {
        "id": "assignment-uuid-9999",
        "nurse_id": "nurse-uuid-8888",
        "nurse_name": "Nurse Priya Shetty",
        "shift": "MORNING",
        "handover_notes": "Patient is a fall risk.",
        "is_active": true
      }
    ]
  }
}
```

---

## 3. Patient Observation Stays

Observations track patients undergoing temporary clinical assessment (e.g. in emergency wards or day care) before deciding on full admission or home discharge.

### 3.1 Record Observation Stay
* **Endpoint:** `POST /ipd/observations`
* **Request Body:**
```json
{
  "patient_id": "patient-uuid-2222",
  "ward_id": "ward-uuid-emergency",
  "bed_id": "bed-uuid-emg-01",
  "observation_reason": "Monitoring query anaphylaxis reaction post injection"
}
```
* **Success Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "id": "obs-uuid-5555",
    "patient_id": "patient-uuid-2222",
    "status": "OBSERVING",
    "created_at": "2026-06-23T18:20:00Z"
  }
}
```

---

### 3.2 List Patient Observations
* **Endpoint:** `GET /ipd/observations`
* **Query Parameters:**
  * `page` (optional): defaults to `1`
  * `page_size` (optional): defaults to `20`

---

### 3.3 Discharge from Observation
* **Endpoint:** `POST /ipd/observations/{observation_id}/discharge`
* **Request Body:**
```json
{
  "summary": "Symptoms completely resolved. Patient fit for home."
}
```

---

### 3.4 Convert Stay to Full Admission
Used when patient condition requires transitioning from day-care observation to full inpatient admission.

* **Endpoint:** `POST /ipd/observations/{observation_id}/convert-to-admission`
* **Request Body:**
```json
{
  "attending_doctor_id": "doc-uuid-cardio",
  "notes": "Admitting for continuous telemetry monitoring"
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Observation converted to admission successfully",
  "data": {
    "admission_id": "admit-uuid-8381",
    "admission_number": "IPD-2026-1024",
    "status": "ADMITTED"
  }
}
```
