# HMS Receptionist Dashboard — API Endpoint Reference

This document is the authoritative API reference for every endpoint consumed by the Receptionist Dashboard across all four operational views: **OPD**, **IPD**, **Emergency (ER)**, and **Operating Theatre (OT)**.

All endpoints require:
```
Authorization: Bearer <access_token>
Content-Type: application/json
```

Standard response envelope:
```json
{ "success": true, "data": { ... }, "message": "..." }
{ "success": false, "error": { "message": "...", "code": 4xx } }
```

---

## Quick Reference — All Dashboard Endpoints

| # | Method | Endpoint | Service | Dashboard View |
|---|--------|----------|---------|----------------|
| 1 | `GET` | `/patients/search` | Patient | OPD — Search |
| 2 | `POST` | `/patients/register` | Patient | OPD — Register |
| 3 | `POST` | `/patients/register-temp` | Patient | ER — Emergency Intake |
| 4 | `POST` | `/patients/{id}/promote` | Patient | ER — Promote Temp Patient |
| 5 | `GET` | `/patients/{id}` | Patient | OPD / IPD — Patient Card |
| 6 | `POST` | `/patients/{id}/attendant` | Patient | IPD — Attendant Setup |
| 7 | `POST` | `/patients/{id}/insurance` | Patient | IPD — Insurance Setup |
| 8 | `GET` | `/opd/dashboard` | OPD | OPD — KPI Cards |
| 9 | `GET` | `/opd/dashboard/date/{YYYY-MM-DD}` | OPD | OPD — Date Dashboard |
| 10 | `GET` | `/opd/appointments` | OPD | OPD — Appointment List |
| 11 | `GET` | `/opd/appointments/today` | OPD | OPD — Today Schedule |
| 12 | `GET` | `/opd/appointments/date/{YYYY-MM-DD}` | OPD | OPD — Schedule by Date |
| 13 | `GET` | `/opd/appointments/{id}` | OPD | OPD — Appointment Detail |
| 14 | `POST` | `/opd/appointments` | OPD | OPD — Book Appointment |
| 15 | `POST` | `/opd/appointments/{id}/confirm` | OPD | OPD — Confirm & Check-In |
| 16 | `POST` | `/opd/appointments/{id}/status` | OPD | OPD — Update Appt Status |
| 17 | `GET` | `/opd/appointments/{id}/position` | OPD | OPD — Token Position |
| 18 | `POST` | `/opd/walkin` | OPD | OPD — Walk-In Registration |
| 19 | `GET` | `/opd/queue/{doctor_id}` | OPD | OPD — Doctor Queue |
| 20 | `GET` | `/opd/queue/{doctor_id}/token-preview` | OPD | OPD — Next Token Preview |
| 21 | `GET` | `/opd/departments` | OPD | OPD — Department List |
| 22 | `GET` | `/opd/departments/{id}/doctors` | OPD | OPD — Doctors in Department |
| 23 | `GET` | `/opd/doctors/{id}/slots` | OPD | OPD — Doctor Slot Availability |
| 24 | `POST` | `/opd/emergency` | OPD | ER — Create Emergency Visit |
| 25 | `PATCH` | `/opd/emergency/{id}/status` | OPD | ER — Update Visit Status |
| 26 | `GET` | `/opd/emergency` | OPD | ER — Emergency Dashboard |
| 27 | `PATCH` | `/ipd/emergency/{visit_id}/bed` | IPD | ER — Bed Assignment / Transfer |
| 28 | `GET` | `/ipd/wards` | IPD | IPD — Ward List |
| 29 | `GET` | `/ipd/beds` | IPD | IPD — Bed List |
| 30 | `GET` | `/ipd/beds/availability` | IPD | IPD — Bed Availability Summary |
| 31 | `GET` | `/ipd/beds/{bed_id}` | IPD | IPD — Bed Detail |
| 32 | `PATCH` | `/ipd/beds/{bed_id}/status` | IPD | IPD — Update Bed Status |
| 33 | `GET` | `/ipd/admission-requests` | IPD | IPD — Admission Inbox |
| 34 | `POST` | `/ipd/admission-requests` | IPD | IPD — Create Admission Request |
| 35 | `PATCH` | `/ipd/admission-requests/{id}/status` | IPD | IPD — Approve/Reject Request |
| 36 | `POST` | `/ipd/admissions` | IPD | IPD — Create Admission |
| 37 | `GET` | `/ipd/admissions` | IPD | IPD — Admission List |
| 38 | `GET` | `/ipd/admissions/{id}` | IPD | IPD — Admission Detail |
| 39 | `POST` | `/ipd/admissions/{id}/transfer` | IPD | IPD — Bed Transfer |
| 40 | `POST` | `/ipd/admissions/{id}/discharge/initiate` | IPD | IPD — Initiate Discharge |
| 41 | `POST` | `/ipd/admissions/{id}/discharge/clearance` | IPD | IPD — Update Clearance Gate |
| 42 | `POST` | `/ipd/admissions/{id}/discharge/complete` | IPD | IPD — Complete Discharge |
| 43 | `GET` | `/ipd/dashboard` | IPD | IPD — Admission Stats |
| 44 | `GET` | `/ipd/dashboard/wards` | IPD | IPD — Ward Occupancy |
| 45 | `GET` | `/ipd/dashboard/tracking` | IPD | IPD — Live Patient Tracking |
| 46 | `GET` | `/billing/clearance/{admission_id}` | Billing | IPD — Billing Clearance Check |
| 47 | `POST` | `/billing/clearance/{admission_id}/clear` | Billing | IPD — Force Billing Clearance |
| 48 | `POST` | `/billing/collect` | Billing | OPD/IPD — Collect Payment |
| 49 | `POST` | `/billing/invoices` | Billing | OPD/IPD — Generate Invoice |
| 50 | `GET` | `/billing/invoices/{invoice_id}` | Billing | OPD/IPD — Invoice Detail |
| 51 | `POST` | `/ipd/ot-sessions` | IPD | OT — Create OT Session |
| 52 | `GET` | `/ipd/ot-sessions/{id}` | IPD | OT — Session Detail |
| 53 | `POST` | `/ipd/ot-sessions/{id}/pac` | IPD | OT — PAC Clearance |
| 54 | `POST` | `/ipd/ot-sessions/{id}/pre-op` | IPD | OT — Pre-Op Complete |
| 55 | `POST` | `/ipd/ot-sessions/{id}/start` | IPD | OT — Start Surgery |
| 56 | `POST` | `/ipd/ot-sessions/{id}/complete` | IPD | OT — Complete + Transfer |
| 57 | `GET` | `/ipd/ot-cases/{ot_case_id}/consents` | IPD | OT — List Consents |
| 58 | `POST` | `/ipd/ot-cases/{ot_case_id}/consents` | IPD | OT — Create Consents |
| 59 | `POST` | `/ipd/consents/{consent_id}/sign` | IPD | OT — Sign Consent |

---

## SECTION 1 — OPD RECEPTIONIST DASHBOARD

The OPD dashboard supports patient registration, appointment booking, real-time queue management, and payment collection at the outpatient reception desk.

### 1.1 Patient Search
Quick lookup used as the first step for every patient interaction. Searches by name, phone, or UHID.

- **Method:** `GET`
- **Endpoint:** `/patients/search?q={query}&page=1&page_size=20`
- **Service:** Patient Service
- **Permission:** `patients:search`

**Query Parameters:**
| Param | Required | Description |
|-------|----------|-------------|
| `q` | Yes | Name, phone number, or UHID |
| `page` | No | Default: 1 |
| `page_size` | No | Default: 20, Max: 100 |

**Response 200 OK:**
```json
{
  "success": true,
  "data": {
    "items": [{ "id": "uuid", "uhid": "PAT-2026-0010", "full_name": "Jane Doe", "phone": "+91...", "is_temporary": false }],
    "total": 1, "page": 1, "page_size": 20
  }
}
```

---

### 1.2 Register New Patient

- **Method:** `POST`
- **Endpoint:** `/patients/register`
- **Service:** Patient Service
- **Permission:** `patients:create`

**Body:**
| Field | Required | Description |
|-------|----------|-------------|
| `full_name` | Yes | Min 2, Max 100 chars |
| `phone` | Yes | Min 10 digits |
| `age` | Yes | 0-150 |
| `gender` | No | MALE / FEMALE / OTHER / UNKNOWN |
| `blood_group` | No | A+ B+ O+ AB+ etc. |
| `dob` | No | YYYY-MM-DD |
| `address` | No | { line1, city, state, pincode } |
| `abha_number` | No | National Health ID |

**Response 201 Created:**
```json
{ "success": true, "message": "Patient registered successfully", "data": { "id": "uuid", "uhid": "PAT-2026-0042", "is_temporary": false } }
```

---

### 1.3 Get Patient Card
- **Method:** `GET`
- **Endpoint:** `/patients/{patient_id}`
- **Service:** Patient Service
- **Permission:** `patients:view`

---

### 1.4 OPD Dashboard — Today (KPI Cards)
- **Method:** `GET`
- **Endpoint:** `/opd/dashboard`
- **Service:** OPD Service
- **Permission:** `dashboard:view`

**Response 200 OK:**
```json
{
  "success": true,
  "data": {
    "date": "2026-06-25",
    "total_appointments": 42,
    "scheduled": 18, "in_progress": 5, "completed": 14,
    "no_show": 3, "cancelled": 2,
    "emergency_count": 1, "walkin_count": 4, "pending_payment": 6,
    "doctors": [{ "doctor_id": "uuid", "doctor_name": "Dr. Anjali Sharma", "scheduled_count": 10, "completed_count": 6, "is_available": true }]
  }
}
```

---

### 1.5 OPD Dashboard — By Date
- **Method:** `GET`
- **Endpoint:** `/opd/dashboard/date/{YYYY-MM-DD}`
- **Service:** OPD Service
- **Permission:** `dashboard:view`

---

### 1.6 List / Filter Appointments
- **Method:** `GET`
- **Endpoint:** `/opd/appointments`
- **Service:** OPD Service
- **Permission:** `appointments:view`

**Query Parameters:** `date` · `doctor_id` · `department_id` · `status` · `appointment_type` · `page` · `page_size`

**Status values:** `SCHEDULED` `CONFIRMED` `IN_PROGRESS` `COMPLETED` `CANCELLED` `NO_SHOW`

---

### 1.7 Today's Appointments
- **Method:** `GET`
- **Endpoint:** `/opd/appointments/today`
- **Service:** OPD Service
- **Permission:** `appointments:view`

---

### 1.8 Appointments by Date
- **Method:** `GET`
- **Endpoint:** `/opd/appointments/date/{YYYY-MM-DD}`
- **Service:** OPD Service
- **Permission:** `appointments:view`

---

### 1.9 Book Appointment
- **Method:** `POST`
- **Endpoint:** `/opd/appointments`
- **Service:** OPD Service
- **Permission:** `appointments:create`

**Body:**
| Field | Required | Description |
|-------|----------|-------------|
| `patient_id` | Yes | Patient UUID |
| `doctor_id` | Yes | Doctor UUID |
| `department_id` | Yes | Department UUID |
| `appointment_date` | Yes | YYYY-MM-DD |
| `appointment_type` | No | OPD (default) |
| `slot_time` | No | Preferred slot time |
| `notes` | No | Additional notes |

---

### 1.10 Confirm Appointment and Check-In

Confirms payment and checks the patient into the queue. Generates queue token server-side. Creates the OPD visit record.

- **Method:** `POST`
- **Endpoint:** `/opd/appointments/{id}/confirm`
- **Service:** OPD Service
- **Permission:** `appointments:edit`

**Body:**
| Field | Required | Description |
|-------|----------|-------------|
| `payment_status` | Yes | Must be PAID to generate token |
| `payment_mode` | No | CASH / UPI / CARD |

> Token number is ALWAYS generated server-side. Never supply a token from the frontend.

---

### 1.11 Update Appointment Status
- **Method:** `POST`
- **Endpoint:** `/opd/appointments/{id}/status`
- **Service:** OPD Service
- **Permission:** `appointments:edit`

**Body:** `{ "status": "NO_SHOW", "reason": "Patient did not arrive" }`

**Valid statuses:** SCHEDULED → CONFIRMED → ARRIVED → IN_PROGRESS → COMPLETED / NO_SHOW / CANCELLED

---

### 1.12 Walk-In Registration

Creates a walk-in OPD appointment without prior booking. Starts in PENDING_PAYMENT state.

- **Method:** `POST`
- **Endpoint:** `/opd/walkin`
- **Service:** OPD Service
- **Permission:** `appointments:create`

**Body:**
| Field | Required | Description |
|-------|----------|-------------|
| `patient_id` | Yes | Existing patient UUID |
| `doctor_id` | Yes | Doctor UUID |
| `department_id` | Yes | Department UUID |
| `chief_complaint` | No | Primary presenting concern |
| `notes` | No | Additional notes |

---

### 1.13 Doctor Queue
- **Method:** `GET`
- **Endpoint:** `/opd/queue/{doctor_id}?date=YYYY-MM-DD`
- **Service:** OPD Service
- **Permission:** `appointments:view`

Returns real-time consultation queue with token, patient name, status, and wait position.

---

### 1.14 Next Token Preview
- **Method:** `GET`
- **Endpoint:** `/opd/queue/{doctor_id}/token-preview`
- **Service:** OPD Service
- **Permission:** `appointments:view`

Returns the next token number that would be assigned for a doctor before payment confirmation.

---

### 1.15 Queue Position
- **Method:** `GET`
- **Endpoint:** `/opd/appointments/{id}/position`
- **Service:** OPD Service
- **Permission:** `appointments:view`

---

### 1.16 List Departments
- **Method:** `GET`
- **Endpoint:** `/opd/departments`
- **Service:** OPD Service
- **Permission:** `appointments:view`

---

### 1.17 Doctors in Department
- **Method:** `GET`
- **Endpoint:** `/opd/departments/{id}/doctors`
- **Service:** OPD Service
- **Permission:** `appointments:view`

Returns available doctors for the selected department including real-time queue lengths.

---

### 1.18 Doctor Slot Availability
- **Method:** `GET`
- **Endpoint:** `/opd/doctors/{id}/slots?date=YYYY-MM-DD`
- **Service:** OPD Service
- **Permission:** `appointments:view`

---

### 1.19 Collect Payment (OPD Cashier)
- **Method:** `POST`
- **Endpoint:** `/billing/collect`
- **Service:** Billing Service
- **Permission:** `billing:payment:collect`

**Body:**
| Field | Required | Description |
|-------|----------|-------------|
| `patient_id` | Yes | Patient UUID |
| `invoice_id` | No | Invoice to settle |
| `payment_mode` | Yes | CASH / UPI / CARD / INSURANCE_TPA / NET_BANKING |
| `amount` | Yes | Amount in INR (greater than 0) |
| `reference_no` | No | UPI ref / cheque number |

---

## SECTION 2 — IPD RECEPTIONIST DASHBOARD

The IPD dashboard manages the interactive bed map, admission requests, bed allocation and transfers, attendant and insurance setup, and the multi-gate discharge clearance workflow.

### 2.1 IPD Dashboard — Stats
- **Method:** `GET`
- **Endpoint:** `/ipd/dashboard`
- **Service:** IPD Service
- **Permission:** `dashboard:view`

Includes: total_admitted, emergency_count, elective_count, pending_discharge, transfers_today.

---

### 2.2 Ward Occupancy Dashboard
- **Method:** `GET`
- **Endpoint:** `/ipd/dashboard/wards`
- **Service:** IPD Service
- **Permission:** `dashboard:view`

---

### 2.3 Live Patient Tracking
- **Method:** `GET`
- **Endpoint:** `/ipd/dashboard/tracking`
- **Service:** IPD Service
- **Permission:** `dashboard:view`

Returns all admitted patients with ward, bed, attending doctor, and length of stay.

---

### 2.4 List Wards
- **Method:** `GET`
- **Endpoint:** `/ipd/wards?ward_type=GENERAL&is_active=true`
- **Service:** IPD Service
- **Permission:** `ipd:view`

**Ward types:** GENERAL / SEMI_PRIVATE / PRIVATE / ICU / NICU / PICU / OBSERVATION / DAY_CARE / ISOLATION / HDU

---

### 2.5 List Beds (Bed Map Grid)
- **Method:** `GET`
- **Endpoint:** `/ipd/beds?ward_id={uuid}&status=AVAILABLE`
- **Service:** IPD Service
- **Permission:** `ipd:view`

**Bed status to UI color mapping:**
| Status | Color |
|--------|-------|
| AVAILABLE | Green (Emerald) |
| OCCUPIED | Red (Crimson) |
| RESERVED | Amber |
| MAINTENANCE | Slate Gray |

---

### 2.6 Bed Availability Summary
- **Method:** `GET`
- **Endpoint:** `/ipd/beds/availability?ward_id={uuid}`
- **Service:** IPD Service
- **Permission:** `ipd:view`

**Response:** `{ total, available, occupied, reserved, maintenance }`

---

### 2.7 Update Bed Status (Manual)
- **Method:** `PATCH`
- **Endpoint:** `/ipd/beds/{bed_id}/status`
- **Service:** IPD Service
- **Permission:** `beds:edit`

**Body:** `{ "status": "MAINTENANCE", "reason": "Deep cleaning in progress" }`

---

### 2.8 Admission Request Inbox
- **Method:** `GET`
- **Endpoint:** `/ipd/admission-requests?status=REQUESTED`
- **Service:** IPD Service
- **Permission:** `admissions:view`

**Status filters:** REQUESTED / UNDER_REVIEW / READY / CANCELLED / CONVERTED

---

### 2.9 Create Admission Request
- **Method:** `POST`
- **Endpoint:** `/ipd/admission-requests`
- **Service:** IPD Service
- **Permission:** `admissions:create`

**Body:** `patient_id` (required) · `request_reason` (required) · `priority` (ROUTINE/URGENT/EMERGENCY) · `source_type` (OPD/EMERGENCY/REFERRAL/WALK_IN)

---

### 2.10 Approve / Update Admission Request
- **Method:** `PATCH`
- **Endpoint:** `/ipd/admission-requests/{request_id}/status`
- **Service:** IPD Service
- **Permission:** `admissions:edit`

**Body:** `{ "status": "READY", "reason": "Bed B-12 confirmed in General Ward" }`

---

### 2.11 Create Admission (Allocate Bed)

Formally admits the patient and atomically marks the selected bed as OCCUPIED. Generates admission_number.

- **Method:** `POST`
- **Endpoint:** `/ipd/admissions`
- **Service:** IPD Service
- **Permission:** `admissions:create`

**Body:**
| Field | Required | Description |
|-------|----------|-------------|
| `patient_id` | Yes | Patient UUID |
| `ward_id` | Yes | Target ward UUID |
| `bed_id` | Yes | Target bed UUID — must be AVAILABLE |
| `admitting_doctor_id` | Yes | Doctor UUID |
| `admission_request_id` | No | Links from admission inbox |
| `emergency_visit_id` | No | Links from ER visit |
| `admission_type` | No | ELECTIVE / EMERGENCY / TRANSFER |
| `diagnosis` | No | Primary diagnosis note |

---

### 2.12 List Admissions
- **Method:** `GET`
- **Endpoint:** `/ipd/admissions?status=ADMITTED&ward_id={uuid}`
- **Service:** IPD Service
- **Permission:** `admissions:view`

---

### 2.13 Internal Bed Transfer

Moves an admitted patient from current bed to another bed. Previous bed auto-releases to AVAILABLE, new bed becomes OCCUPIED — both in one transaction.

- **Method:** `POST`
- **Endpoint:** `/ipd/admissions/{id}/transfer`
- **Service:** IPD Service
- **Permission:** `admissions:edit`

**Body:**
| Field | Required | Description |
|-------|----------|-------------|
| `to_ward_id` | Yes | Target ward UUID |
| `to_bed_id` | Yes | Target bed UUID (must be AVAILABLE) |
| `reason` | Yes | Clinical reason for transfer |

---

### 2.14 Link Attendant
- **Method:** `POST`
- **Endpoint:** `/patients/{patient_id}/attendant`
- **Service:** Patient Service
- **Permission:** `patients:edit`

**Body:** `full_name` (required) · `relationship` (required) · `phone` (required) · `alternate_phone`

---

### 2.15 Link Insurance Policy
- **Method:** `POST`
- **Endpoint:** `/patients/{patient_id}/insurance`
- **Service:** Patient Service
- **Permission:** `patients:edit`

**Body:** `provider_name` (required) · `policy_number` (required) · `scheme_type` (ROHINI/TPA/CGHS/ESI) · `valid_from` · `valid_till`

---

### 2.16 Initiate Discharge

Creates four parallel clearance gates: NURSING, DIAGNOSTIC, PHARMACY, BILLING — all default to PENDING.

- **Method:** `POST`
- **Endpoint:** `/ipd/admissions/{id}/discharge/initiate`
- **Service:** IPD Service
- **Permission:** `discharge:initiate`

---

### 2.17 Update Discharge Clearance Gate

Marks a specific gate as CLEARED. Receptionist handles the BILLING gate after invoice settlement.

- **Method:** `POST`
- **Endpoint:** `/ipd/admissions/{id}/discharge/clearance`
- **Service:** IPD Service
- **Permission:** `discharge:clearance`

**Body:**
```json
{ "gate": "BILLING", "status": "CLEARED", "cleared_by": "receptionist-uuid", "notes": "Invoice settled via UPI" }
```

**Gates:** NURSING · DIAGNOSTIC · PHARMACY · BILLING

> Diagnostic Gate: If no lab tests were ordered or taken, the receptionist may manually mark DIAGNOSTIC as CLEARED.

---

### 2.18 Check Billing Clearance
- **Method:** `GET`
- **Endpoint:** `/billing/clearance/{admission_id}`
- **Service:** Billing Service
- **Permission:** `billing:clearance:view`

**Response key fields:** `total_charges` · `total_paid` · `outstanding_balance` · `is_cleared`

---

### 2.19 Force Billing Clearance (Override)

For charity, corporate credit, or emergency exemptions. Admin/Accountant only.

- **Method:** `POST`
- **Endpoint:** `/billing/clearance/{admission_id}/clear`
- **Service:** Billing Service
- **Permission:** `billing:clearance:manage`

**Body:**
| Field | Required | Description |
|-------|----------|-------------|
| `clearance_type` | Yes | CREDIT_GUARANTEE / CHARITY / EMPLOYEE_BENEFIT / FORCE_MAJEURE / SETTLED |
| `approved_by` | Yes | Admin UUID authorizing the override |
| `notes` | No | Justification notes |

---

### 2.20 Complete Discharge

Finalizes discharge once ALL 4 gates are CLEARED. Releases bed to AVAILABLE.

- **Method:** `POST`
- **Endpoint:** `/ipd/admissions/{id}/discharge/complete`
- **Service:** IPD Service
- **Permission:** `discharge:complete`

> Returns 400 if any clearance gate remains PENDING.

---

## SECTION 3 — EMERGENCY (ER) RECEPTIONIST DASHBOARD

Optimized for sub-10-second case creation, color-coded triage sorting, and instant trauma bed dispatch.

### Triage Level Reference

| Level | Colour | Description | Target Response |
|-------|--------|-------------|-----------------|
| RED | Pulsing Crimson | Resuscitation — immediate life threat | Immediate |
| ORANGE | Dark Orange | Emergent — very urgent | Within 10 min |
| YELLOW | Amber | Urgent | Within 30 min |
| GREEN | Blue-Gray | Non-urgent | Within 120 min |

---

### 3.1 Register Temporary Emergency Patient

Creates a placeholder patient record when identity is unknown. Generates TEMP-EMG-YYYYMMDD-XXX UHID. All fields optional — designed for zero-obstruction intake.

- **Method:** `POST`
- **Endpoint:** `/patients/register-temp`
- **Service:** Patient Service
- **Permission:** `patients:create`

**Body (all optional):** `{ "full_name": "Unknown Male Patient", "age": 40, "gender": "MALE" }`

An empty `{}` body is valid for completely unknown patients.

---

### 3.2 Create Emergency Visit
- **Method:** `POST`
- **Endpoint:** `/opd/emergency`
- **Service:** OPD Service
- **Permission:** `emergency:create`

**Body:**
| Field | Required | Description |
|-------|----------|-------------|
| `patient_id` | No | Temporary or full patient UUID |
| `full_name` | No | If patient not yet registered |
| `triage_level` | Yes | RED / ORANGE / YELLOW / GREEN |
| `arrival_mode` | Yes | WALK_IN / AMBULANCE / REFERRAL / TRANSFER |
| `chief_complaint` | No | Primary presenting complaint |
| `assigned_doctor_id` | No | Attending emergency doctor |
| `notes` | No | Additional notes |

---

### 3.3 Emergency Dashboard
- **Method:** `GET`
- **Endpoint:** `/opd/emergency?date=YYYY-MM-DD`
- **Service:** OPD Service
- **Permission:** `emergency:view`

Returns all visits sorted by triage level (RED at top) with counts, elapsed time, and bed assignments.

---

### 3.4 Update Emergency Visit Status

Updates clinical status and doctor assignment. Does NOT manage bed state (use IPD service for that).

- **Method:** `PATCH`
- **Endpoint:** `/opd/emergency/{visit_id}/status`
- **Service:** OPD Service
- **Permission:** `emergency:update`

**Status flow:** ARRIVED → TRIAGED → UNDER_TREATMENT → ADMITTED / DISCHARGED

**Body:** `{ "status": "UNDER_TREATMENT", "assigned_doctor_id": "uuid", "notes": "Patient responding to treatment" }`

---

### 3.5 Assign / Transfer / Release Emergency Bed

Atomically manages bed state for emergency patients. Owned by IPD service (beds are an IPD resource).

- **Method:** `PATCH`
- **Endpoint:** `/ipd/emergency/{visit_id}/bed`
- **Service:** IPD Service
- **Permission:** `beds:edit`

| Action | Body |
|--------|------|
| Assign bed | `{ "bed_id": "<available-bed-uuid>" }` |
| Transfer to new bed | `{ "bed_id": "<new-available-bed-uuid>" }` |
| Release bed | `{ "bed_id": null }` |

Returns 409 if target bed is OCCUPIED · Returns 400 if bed is inactive · Returns 404 if visit or bed not found.

---

### 3.6 Promote Temporary Patient to Full Registration

Upgrades TEMP-EMG-... UHID to permanent PAT-YYYY-XXXX. All clinical records remain linked via the same patient UUID.

- **Method:** `POST`
- **Endpoint:** `/patients/{patient_id}/promote`
- **Service:** Patient Service
- **Permission:** `patients:edit`

**Body:**
| Field | Required | Description |
|-------|----------|-------------|
| `full_name` | Yes | Min 2, Max 100 chars |
| `phone` | Yes | Min 10 digits |
| `age` | Yes | 0-150 |
| `gender` | No | MALE / FEMALE / OTHER / UNKNOWN |
| `blood_group` | No | Standard blood group |
| `address` | No | { line1, city, state, pincode } |
| `abha_number` | No | ABHA Health ID |

Returns 400 if patient is already a full registration.

---

## SECTION 4 — OPERATING THEATRE (OT) RECEPTIONIST DASHBOARD

Manages the surgical calendar, pre-operative checklist, consent management, and post-op recovery transfer.

### 4.1 Create OT Session
- **Method:** `POST`
- **Endpoint:** `/ipd/ot-sessions`
- **Service:** IPD Service
- **Permission:** `ot:create`

**Body:**
| Field | Required | Description |
|-------|----------|-------------|
| `surgery_request_id` | Yes | UUID from surgery orders list |
| `ot_room_id` | Yes | Target OT room UUID |
| `scheduled_start` | Yes | ISO 8601 DateTime |
| `scheduled_end` | Yes | ISO 8601 DateTime |
| `surgeon_id` | Yes | Primary surgeon UUID |
| `anaesthetist_id` | No | Anaesthetist UUID |
| `scrub_nurse_id` | No | Scrub nurse UUID |

---

### 4.2 Get OT Session Detail
- **Method:** `GET`
- **Endpoint:** `/ipd/ot-sessions/{id}`
- **Service:** IPD Service
- **Permission:** `ot:view`

---

### 4.3 List Consents for OT Case
- **Method:** `GET`
- **Endpoint:** `/ipd/ot-cases/{ot_case_id}/consents`
- **Service:** IPD Service
- **Permission:** `ot:view`

---

### 4.4 Create Consents for OT Case
- **Method:** `POST`
- **Endpoint:** `/ipd/ot-cases/{ot_case_id}/consents`
- **Service:** IPD Service
- **Permission:** `ot:create`

---

### 4.5 Sign Consent
- **Method:** `POST`
- **Endpoint:** `/ipd/consents/{consent_id}/sign`
- **Service:** IPD Service
- **Permission:** `ot:edit`

**Body:** `{ "signature_type": "DIGITAL", "signed_by": "user-uuid", "doctor_id": "uuid", "witness_id": "uuid", "remarks": "Signed in person by patient" }`

---

### 4.6 PAC Clearance
- **Method:** `POST`
- **Endpoint:** `/ipd/ot-sessions/{id}/pac`
- **Service:** IPD Service
- **Permission:** `ot:edit`

**Body:** `{ "pac_cleared": true, "cleared_by": "anaesthetist-uuid", "notes": "Patient fit for GA" }`

---

### 4.7 Complete Pre-Op Checklist
- **Method:** `POST`
- **Endpoint:** `/ipd/ot-sessions/{id}/pre-op`
- **Service:** IPD Service
- **Permission:** `ot:edit`

**Body:**
```json
{
  "checklist": { "fasting_confirmed": true, "consent_signed": true, "site_marked": true, "jewellery_removed": true, "iv_access": true, "pre_med_given": true },
  "pre_op_notes": "Patient ready for incision"
}
```

---

### 4.8 Start OT Session
- **Method:** `POST`
- **Endpoint:** `/ipd/ot-sessions/{id}/start`
- **Service:** IPD Service
- **Permission:** `ot:edit`

**Body:** `{ "started_by": "surgeon-uuid", "actual_start": "2026-06-25T09:15:00Z" }`

---

### 4.9 Complete OT Session and Transfer to Recovery
- **Method:** `POST`
- **Endpoint:** `/ipd/ot-sessions/{id}/complete`
- **Service:** IPD Service
- **Permission:** `ot:edit`

**Body:**
| Field | Required | Description |
|-------|----------|-------------|
| `transferred_to_ward_id` | Yes | Recovery ward UUID |
| `transferred_to_bed_id` | Yes | Recovery bed UUID |
| `recovery_notes` | No | Post-operative observations |

---

## SECTION 5 — PERMISSIONS REFERENCE

| Permission | Who Holds It | Endpoints |
|-----------|-------------|-----------|
| `patients:create` | Receptionist, Admin | /patients/register, /patients/register-temp |
| `patients:edit` | Receptionist, Admin, Nurse | Update patient, Promote temp, Attendant, Insurance |
| `patients:view` | All clinical roles | /patients/{id} |
| `patients:search` | All clinical roles | /patients/search |
| `appointments:create` | Receptionist | Book appointment, Walk-in |
| `appointments:edit` | Receptionist, Admin | Confirm, Update status |
| `appointments:view` | All clinical roles | List, Today, Queue, Slots |
| `dashboard:view` | Receptionist, Admin | OPD and IPD dashboards |
| `emergency:create` | Receptionist, Admin, Nurse | Create emergency visit |
| `emergency:update` | Receptionist, Admin, Doctor | Update visit status |
| `emergency:view` | All clinical roles | Emergency dashboard |
| `admissions:create` | Receptionist, Admin | Admission request + admission |
| `admissions:edit` | Receptionist, Admin | Request status update, Transfer |
| `admissions:view` | All clinical roles | Wards, beds, admission list |
| `beds:edit` | Receptionist, Admin | Manual bed status, Emergency bed assign |
| `discharge:initiate` | Receptionist, Admin, Doctor | Initiate discharge |
| `discharge:clearance` | Receptionist (BILLING gate), Dept heads (clinical gates) | Update clearance gates |
| `discharge:complete` | Receptionist, Admin | Complete discharge |
| `billing:payment:collect` | Receptionist, Accountant | Collect payment |
| `billing:clearance:view` | Receptionist, Accountant | Check billing clearance |
| `billing:clearance:manage` | Admin, Accountant | Force billing clearance override |
| `ot:create` | Receptionist, Admin | Create OT session, Consents |
| `ot:edit` | Receptionist, Admin, Nurse | PAC, Pre-op, Start, Complete |
| `ot:view` | All clinical roles | OT session detail, Consents list |

---

## SECTION 6 — WORKFLOW SEQUENCES

### OPD Walk-In Flow
```
GET /patients/search?q=...         (search existing)
    if not found:
POST /patients/register            (PAT-2026-XXXX issued)
    |
POST /opd/walkin                   (status: PENDING_PAYMENT)
    |
POST /billing/collect              (receipt issued)
    |
POST /opd/appointments/{id}/confirm (payment_status=PAID)
    |
Token generated — patient joins queue
GET /opd/queue/{doctor_id}         (monitor queue)
```

### IPD Admission Flow
```
GET /ipd/admission-requests?status=REQUESTED   (inbox)
GET /ipd/beds?status=AVAILABLE&ward_id=...     (find bed)
PATCH /ipd/admission-requests/{id}/status      (READY)
POST /ipd/admissions                           (bed: OCCUPIED, admission_number issued)
POST /patients/{id}/attendant
POST /patients/{id}/insurance
--- [during stay] ---
POST /ipd/admissions/{id}/transfer             (bed swap)
--- [discharge] ---
POST /ipd/admissions/{id}/discharge/initiate   (4 gates: PENDING)
GET /billing/clearance/{admission_id}          (check balance)
POST /billing/collect                          (settle dues)
POST /ipd/admissions/{id}/discharge/clearance  (BILLING: CLEARED)
POST /ipd/admissions/{id}/discharge/complete   (bed: AVAILABLE)
```

### Emergency Flow
```
POST /patients/register-temp                   (TEMP-EMG-20260625-001)
POST /opd/emergency                            (triage_level=RED)
PATCH /ipd/emergency/{id}/bed                  (bed_id=<trauma-bay> -> OCCUPIED)
PATCH /opd/emergency/{id}/status               (UNDER_TREATMENT)
POST /patients/{id}/promote                    (PAT-2026-XXXX)
    if admitting to IPD:
POST /ipd/admissions                           (emergency_visit_id linked)
PATCH /ipd/emergency/{id}/bed                  (bed_id=null -> AVAILABLE)
    if discharging:
PATCH /opd/emergency/{id}/status               (DISCHARGED)
PATCH /ipd/emergency/{id}/bed                  (bed_id=null -> AVAILABLE)
```

### OT Scheduling Flow
```
POST /ipd/ot-sessions              (room + surgeon + time scheduled)
POST /ipd/ot-cases/{id}/consents   (consent docs generated)
POST /ipd/consents/{id}/sign       (patient/guardian signs)
POST /ipd/ot-sessions/{id}/pac     (anaesthetist clearance)
POST /ipd/ot-sessions/{id}/pre-op  (all pre-op checks done)
POST /ipd/ot-sessions/{id}/start   (surgery begins)
POST /ipd/ot-sessions/{id}/complete (recovery ward + bed assigned)
```

---

*HMS Receptionist Dashboard API Reference | Services: Patient · OPD · IPD · Billing | Last Updated: June 2026*
