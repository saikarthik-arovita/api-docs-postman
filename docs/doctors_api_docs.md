# HMS — Doctors Service API Documentation

This document covers all queue management, EMR context readings, prescribing, IPD decisions, referral orders, round checklists, scheduling, and analytics for the HMS **Doctors Service**.

---

## 1. Global Conventions

### Base URL
All requests are sent to the Doctors Service API Gateway stage:
```
https://<api-id>.execute-api.ap-south-1.amazonaws.com/<stage>
```

### Authorization Header
All endpoints require a valid JWT Access Token passed in the `Authorization` header:
```http
Authorization: Bearer <access_token>
```

---

## 2. Doctor Dashboard & Queue

### 2.1 Doctor Dashboard Summary
* **Endpoint:** `GET /doctors/{doctor_id}/dashboard`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "doctor_id": "902d2bc4-f5ee-45df-98bd-674cd7bb0eef",
    "doctor_name": "Dr. Priya Sharma",
    "today_appointments": 15,
    "waiting_patients": 4,
    "in_consultation": 1,
    "completed_today": 10,
    "ipd_assigned_patients": 3,
    "revenue_today": 5000.00,
    "pending_notes": 2,
    "pending_discharge_summaries": 1,
    "followups_due_today": 3,
    "as_of": "2026-06-23T17:50:00Z"
  }
}
```

---

### 2.2 Get OPD Queue
* **Endpoint:** `GET /doctors/{doctor_id}/opd/queue`
* **Query Parameters:**
  * `date` (optional): Filter queue date (`YYYY-MM-DD`, defaults to today)
  * `status` (optional): Filter by queue token status
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "doctor_id": "902d2bc4-f5ee-45df-98bd-674cd7bb0eef",
    "queue_date": "2026-06-23",
    "items": [
      {
        "appointment_id": "appt-uuid-1111",
        "opd_visit_id": "visit-uuid-2222",
        "patient_id": "patient-uuid-3333",
        "patient_name": "Ravi Kumar",
        "patient_phone": "+919999888877",
        "token_number": 4,
        "appointment_time": "2026-06-23T10:30:00Z",
        "appointment_status": "CHECKED_IN",
        "visit_status": "WAITING"
      }
    ],
    "total": 1,
    "waiting": 1,
    "in_progress": 0,
    "completed": 0
  }
}
```

---

### 2.3 Call Next Patient
* **Endpoint:** `GET /doctors/{doctor_id}/opd/next`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "appointment_id": "appt-uuid-1111",
    "patient_name": "Ravi Kumar",
    "token_number": 4
  }
}
```

---

### 2.4 Mark No-Show
* **Endpoint:** `POST /doctors/opd/no-show`
* **Request Body:**
```json
{
  "appointment_id": "appt-uuid-1111"
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "message": "Appointment marked as no-show successfully"
  }
}
```

---

### 2.5 Emergency Insert
Forces a patient token slot directly into the active doctor queue.

* **Endpoint:** `POST /doctors/opd/emergency-insert`
* **Request Body:**
```json
{
  "patient_id": "patient-uuid-3333",
  "chief_complaint": "Severe chest pain",
  "department_id": "dept-uuid-cardiology",
  "reason": "Suspected myocardial infarction"
}
```
* **Success Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "appointment_id": "appt-uuid-emergency",
    "token_number": 1,
    "message": "Emergency patient inserted in queue"
  }
}
```

---

## 3. Patient EMR & Context Lookups

### 3.1 Patient EMR Context
Retrieves right-hand summary dashboard containing patient history, allergies, and last vitals.

* **Endpoint:** `GET /doctors/{doctor_id}/patients/{patient_id}/context`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "patient_id": "patient-uuid-3333",
    "full_name": "Ravi Kumar",
    "mrn": "MRN-2026-0042",
    "gender": "MALE",
    "allergies": ["Penicillin"],
    "latest_vitals": {
      "bp_systolic": 120,
      "bp_diastolic": 80,
      "pulse": 72,
      "temperature": 37.0
    }
  }
}
```

---

### 3.2 Patient Complete EMR History
Retrieves full clinical encounters log.

* **Endpoint:** `GET /doctors/{doctor_id}/patients/{patient_id}/emr`

---

## 4. Prescribing Workflows

### 4.1 Create Prescription
* **Endpoint:** `POST /doctors/{doctor_id}/patients/{patient_id}/prescriptions`
* **Request Body:**
```json
{
  "opd_visit_id": "visit-uuid-2222",
  "items": [
    {
      "medicine_name": "Paracetamol 650mg",
      "route": "ORAL",
      "dosage": "1 tablet",
      "frequency": "1-0-1",
      "duration_days": 5,
      "with_food": true,
      "instructions": "Take after breakfast and dinner"
    }
  ],
  "allergy_checked": true,
  "interaction_checked": true
}
```
* **Success Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "id": "rx-uuid-8888",
    "prescription_id": "RX-2026-0092",
    "status": "SIGNED",
    "items": [
      {
        "id": "item-uuid-1",
        "medicine_name": "Paracetamol 650mg",
        "route": "ORAL",
        "dosage": "1 tablet",
        "frequency": "1-0-1"
      }
    ]
  }
}
```

---

### 4.2 Edit Prescription (Full Replace / Header Updates)
* **Endpoint:** `PATCH /doctors/{doctor_id}/patients/{patient_id}/prescriptions/{prescription_id}`
* **Request Body:**
```json
{
  "valid_till": "2026-07-01",
  "items": [
    {
      "medicine_name": "Paracetamol 650mg",
      "route": "ORAL",
      "dosage": "1 tablet",
      "frequency": "1-1-1",
      "duration_days": 3
    }
  ]
}
```

---

### 4.3 Edit Individual Prescription Item
* **Endpoint:** `PATCH /doctors/{doctor_id}/patients/{patient_id}/prescriptions/{prescription_id}/items/{item_id}`
* **Request Body:**
```json
{
  "dosage": "1/2 tablet",
  "instructions": "Take only if pain persists"
}
```

---

### 4.4 Discontinue Medicine
Marks medicine as discontinued while preserving the prescription timeline.

* **Endpoint:** `POST /doctors/prescriptions/items/{item_id}/discontinue`
* **Request Body:**
```json
{
  "reason": "Patient developed gastric irritation"
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Medicine discontinued successfully"
}
```

---

## 5. Inpatient Care, Referrals, & Ward Rounds

### 5.1 Discharge Approval
Doctor records discharge summary, recovery parameters, and follow-up guidance.

* **Endpoint:** `POST /doctors/ipd/discharge`
* **Request Body:**
```json
{
  "admission_id": "admit-uuid-1234",
  "discharge_summary": "Patient recovered completely post appendectomy. Wound dressing clean.",
  "follow_up_date": "2026-06-30",
  "actual_discharge_at": "2026-06-23T17:50:00Z"
}
```

---

### 5.2 Request Ward / Bed Transfer
* **Endpoint:** `POST /doctors/ipd/transfer`
* **Request Body:**
```json
{
  "admission_id": "admit-uuid-1234",
  "new_ward_id": "ward-uuid-general-b",
  "new_bed_id": "bed-uuid-gw-b02",
  "reason": "Transitioning patient from ICU to step-down general ward"
}
```

---

### 5.3 Request Referral
* **Endpoint:** `POST /doctors/referral`
* **Request Body:**
```json
{
  "patient_id": "patient-uuid-3333",
  "referred_to_dept_id": "dept-uuid-cardiology",
  "opd_visit_id": "visit-uuid-2222",
  "referral_reason": "For expert cardiac consultation and ECG review",
  "urgency": "URGENT"
}
```

---

### 5.4 Save Daily Round Checklist
Tracks checking off patient metrics during ward visits.

* **Endpoint:** `POST /doctors/rounds/checklist`
* **Request Body:**
```json
{
  "admission_id": "admit-uuid-1234",
  "round_date": "2026-06-23",
  "vitals_reviewed": true,
  "labs_reviewed": true,
  "meds_reviewed": true,
  "plan_updated": true,
  "round_notes": "Patient active, vitals within normal parameters."
}
```

---

## 6. Schedules & availability overrides

### 6.1 Create Schedule Profile
* **Endpoint:** `POST /doctors/{doctor_id}/schedule`
* **Request Body:**
```json
{
  "day_of_week": 1,
  "start_time": "09:00:00",
  "end_time": "13:00:00",
  "department_id": "dept-uuid-opd",
  "slot_duration": 15,
  "max_patients": 16
}
```

---

### 6.2 Set Availability Override (e.g. Block slot or Extra session)
* **Endpoint:** `POST /doctors/{doctor_id}/schedule/overrides`
* **Request Body:**
```json
{
  "override_date": "2026-06-25",
  "override_type": "BLOCK",
  "reason": "Attending annual medical conference"
}
```
