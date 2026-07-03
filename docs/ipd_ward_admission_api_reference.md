# IPD Ward Admission & Discharge API Reference

This document defines the request and response contracts for the backend APIs represented across the Inpatient Department (IPD) Dashboards, Admission Folder, Clinical Handovers, and Case Closure screens.

---

## 1. Ward & Bed Dashboard APIs

These APIs power the ward layouts, bed grid visualizations, and real-time occupancy metrics shown in the dashboard.

### A. Real-time Ward Bed Summary
Retrieves capacity, occupancy, and status summary metrics by ward.
* **Endpoint**: `GET /admin/beds/summary`
* **Method**: `GET`
* **Headers**: `Authorization: Bearer <token>`
* **Response (200 OK)**:
```json
{
  "success": true,
  "data": [
    {
      "ward_id": "4100a692-6605-452d-baca-314892ba002d",
      "ward_name": "Emergency Ward",
      "ward_type": "EMERGENCY",
      "capacity": 20,
      "total_beds": 20,
      "available": 14,
      "occupied": 4,
      "reserved": 1,
      "maintenance": 1
    }
  ]
}
```

### B. List / Search Beds
Query beds with multi-field search and filters.
* **Endpoint**: `GET /ipd/beds`
* **Method**: `GET`
* **Headers**: `Authorization: Bearer <token>`
* **Query Parameters**:
  - `ward_id` (Optional): Filter by ward
  - `status` (Optional): Filter by status (`AVAILABLE`, `OCCUPIED`, `RESERVED`, `MAINTENANCE`)
  - `bed_type` (Optional): Filter by bed type (`STANDARD`, `ICU`, `HDU`, `PRIVATE`, `EMERGENCY`, etc.)
  - `search` (Optional): Fuzzy search by bed number
* **Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "3aa3856b-c80c-4ba5-9d5a-bfbbd4961a1f",
        "ward_id": "4100a692-6605-452d-baca-314892ba002d",
        "ward_name": "Emergency Ward",
        "bed_number": "ER-02",
        "bed_type": "EMERGENCY",
        "status": "AVAILABLE",
        "is_active": true
      }
    ],
    "meta": {
      "total": 1,
      "page": 1,
      "page_size": 50
    }
  }
}
```

### C. Update Bed Status
Update the operational state of a bed.
* **Endpoint**: `PATCH /ipd/beds/{bed_id}/status`
* **Method**: `PATCH`
* **Headers**:
  - `Content-Type: application/json`
  - `Authorization: Bearer <token>`
* **Request Body**:
```json
{
  "status": "MAINTENANCE",
  "reason": "O2 wall nozzle malfunctioning"
}
```
* **Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "bed_id": "3aa3856b-c80c-4ba5-9d5a-bfbbd4961a1f",
    "status": "MAINTENANCE"
  }
}
```

---

## 2. Admission Requests & Approvals

Manages the incoming queue of patients from OPD/Emergency requiring inpatient care.

### A. List Admission Requests
Queries pending admission requests.
* **Endpoint**: `GET /ipd/admission-requests`
* **Method**: `GET`
* **Headers**: `Authorization: Bearer <token>`
* **Query Parameters**:
  - `status` (Optional): Filter by status (`REQUESTED`, `UNDER_REVIEW`, `READY`)
  - `patient_id` (Optional): Filter by patient UUID
  - `source_type` (Optional): Filter by origin (`OPD`, `EMERGENCY`)
* **Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "requests": [
      {
        "id": "e00bc192-fa42-4eb3-81ef-8b26e55fc110",
        "patient_id": "f5b8d234-c712-4eb3-81ef-8b26e55fc244",
        "patient_name": "Rajesh Kumar",
        "source_type": "OPD",
        "source_reference": "opd-visit-1002",
        "recommended_ward_type": "GENERAL",
        "priority": "URGENT",
        "status": "REQUESTED",
        "created_at": "2026-07-03T10:15:30Z"
      }
    ]
  }
}
```

### B. Approve / Update Request Status
Transitions request status through the admission review lifecycle.
* **Endpoint**: `PATCH /ipd/admission-requests/{request_id}/status`
* **Method**: `PATCH`
* **Headers**:
  - `Content-Type: application/json`
  - `Authorization: Bearer <token>`
* **Request Body**:
```json
{
  "status": "READY",
  "reason": "Bed available in General Ward"
}
```
* **Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "request_id": "e00bc192-fa42-4eb3-81ef-8b26e55fc110",
    "status": "READY"
  }
}
```

---

## 3. Patient Inpatient Admission Folder

These APIs power the core admission file interface showing clinical notes, vitals, daily logs, and nurse handovers.

### A. Get Inpatient Admission Folder
Fetches active treatment file, medical histories, bed information, and indicators.
* **Endpoint**: `GET /ipd/admissions/{id}`
* **Method**: `GET`
* **Headers**: `Authorization: Bearer <token>`
* **Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "id": "b453258f-23a7-4047-b316-ab5a655fd898",
    "admission_number": "IPD-2026-00042",
    "patient_id": "f5b8d234-c712-4eb3-81ef-8b26e55fc244",
    "patient_name": "John Doe",
    "patient_uhid": "UHID123456",
    "ward_id": "4100a692-6605-452d-baca-314892ba002d",
    "ward_name": "General Ward",
    "bed_id": "3aa3856b-c80c-4ba5-9d5a-bfbbd4961a1f",
    "bed_number": "B-GEN-09",
    "attending_doctor_id": "a90df234-601e-4cb8-9bde-452f38abf001",
    "attending_doctor_name": "Dr. Supreeth",
    "admission_reason": "Post-op cardiac recovery",
    "status": "ADMITTED",
    "ipd_status": "UNDER_TREATMENT",
    "admitted_at": "2026-07-03T17:03:00Z"
  }
}
```

### B. Add Daily Inpatient Note
Appends clinical observation/shift handovers logs.
* **Endpoint**: `POST /ipd/admissions/{id}/notes`
* **Method**: `POST`
* **Headers**:
  - `Content-Type: application/json`
  - `Authorization: Bearer <token>`
* **Request Body**:
```json
{
  "note_type": "DOCTOR",
  "note_text": "Vitals stable, continue saline IV. Wound healing fine.",
  "is_confidential": false
}
```
* **Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "id": "782f234-c712-4eb3-81ef-9c26e55fc301",
    "admission_id": "b453258f-23a7-4047-b316-ab5a655fd898",
    "note_type": "DOCTOR",
    "note_text": "Vitals stable, continue saline IV. Wound healing fine.",
    "recorded_by": "fcf18321-5f85-4a9c-817f-a5392e612a34",
    "note_date": "2026-07-03"
  }
}
```

### C. List Daily Inpatient Notes
Retrieves treatment/shift logs.
* **Endpoint**: `GET /ipd/admissions/{id}/notes`
* **Method**: `GET`
* **Headers**: `Authorization: Bearer <token>`
* **Response (200 OK)**:
```json
{
  "success": true,
  "data": [
    {
      "id": "782f234-c712-4eb3-81ef-9c26e55fc301",
      "note_type": "DOCTOR",
      "note_text": "Vitals stable, continue saline IV. Wound healing fine.",
      "recorded_by_name": "Dr. Supreeth",
      "note_date": "2026-07-03"
    }
  ]
}
```

---

## 4. Nurse Handover & Duty Assignment

Tracks shift scheduling and nursing care team assignments.

### A. Assign Nurse to Patient
Allocates duty nursing staff to a patient's care plan.
* **Endpoint**: `POST /ipd/admissions/{id}/nursing/assign`
* **Method**: `POST`
* **Headers**:
  - `Content-Type: application/json`
  - `Authorization: Bearer <token>`
* **Request Body**:
```json
{
  "nurse_id": "673f412-fa42-4eb3-81ef-8b26e55fd002",
  "shift": "NIGHT",
  "assignment_date": "2026-07-03",
  "notes": "Monitor oxygen saturation every 2 hours"
}
```
* **Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "id": "890fa31-c712-4eb3-81ef-8b26e55fc888",
    "admission_id": "b453258f-23a7-4047-b316-ab5a655fd898",
    "nurse_id": "673f412-fa42-4eb3-81ef-8b26e55fd002",
    "shift": "NIGHT",
    "assignment_date": "2026-07-03"
  }
}
```

---

## 5. Discharge Clearance & Case Closure

Manages discharge checklist clearances, gate approvals, and case folder closures.

### A. Initiate Patient Discharge Plan
Kicks off the checkout checklist.
* **Endpoint**: `POST /ipd/admissions/{id}/discharge/initiate`
* **Method**: `POST`
* **Headers**: `Authorization: Bearer <token>`
* **Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "admission_id": "b453258f-23a7-4047-b316-ab5a655fd898",
    "ipd_status": "READY_FOR_DISCHARGE"
  }
}
```

### B. Clear Gate Clearance Status
Confirms gate approval from billing, nursing, pharmacy, or the attending doctor.
* **Endpoint**: `POST /ipd/admissions/{id}/discharge/clearance`
* **Method**: `POST`
* **Headers**:
  - `Content-Type: application/json`
  - `Authorization: Bearer <token>`
* **Request Body**:
```json
{
  "gate_name": "BILLING",
  "status": "CLEARED",
  "remarks": "Invoices fully settled."
}
```
* **Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "admission_id": "b453258f-23a7-4047-b316-ab5a655fd898",
    "gate_name": "BILLING",
    "status": "CLEARED"
  }
}
```

### C. Complete Inpatient Discharge
Final step that checks out the patient and releases their bed.
* **Endpoint**: `POST /ipd/admissions/{id}/discharge/complete`
* **Method**: `POST`
* **Headers**:
  - `Content-Type: application/json`
  - `Authorization: Bearer <token>`
* **Request Body**:
```json
{
  "discharge_type": "NORMAL",
  "discharge_summary": "Recovered post-op. Prescribed medication course."
}
```
* **Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "admission_id": "b453258f-23a7-4047-b316-ab5a655fd898",
    "status": "DISCHARGED",
    "actual_discharge_at": "2026-07-03T17:10:00Z"
  }
}
```

### D. Close Inpatient Case File
Closes the medical file record folder permanently.
* **Endpoint**: `POST /ipd/admissions/{id}/close`
* **Method**: `POST`
* **Headers**: `Authorization: Bearer <token>`
* **Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "admission_id": "b453258f-23a7-4047-b316-ab5a655fd898",
    "closed_at": "2026-07-03T17:15:00Z"
  }
}
```
