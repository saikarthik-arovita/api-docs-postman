# Arovita HMS — Surgery & Procedure Documentation Flow Spec

**Version:** 1.0 (Production Reference Guide)  
**Target Audience:** Frontend Engineers, API Consumers, QA Engineers, Core Backend Developers  
**Core Objective:** Map the state transitions, database schema tables, and exact API request/response payloads for major Operating Theatre (OT) Surgeries and Minor Bedside Procedures.

---

## 🗺️ 1. Global Flow Overview

The system classifies surgical and clinical documentation into two main workflows:

1. **Bedside / Minor Clinical Procedures**: Executed in patient wards or consultation rooms, bypassing the Operating Theatre (OT) scheduling, anesthesia checkups, and recovery phases.
2. **Operating Theatre (OT) Major Surgeries**: Follows a comprehensive clinical lifecycle: Order Escalation ➔ Consent Verification ➔ Pre-Anesthetic Checkup (PAC) Clearance ➔ Pre-Op Nursing Checklist ➔ Surgical Intra-Op Notes ➔ Recovery Ward Monitoring ➔ Ward Transfer.

```mermaid
graph TD
    %% Bedside Procedure Flow
    subgraph Bedside Procedure
        P1[Create Procedure Request] -->|POST /ipd/procedures| P2[Status: REQUESTED]
        P2 -->|POST /ipd/procedures/{id}/start| P3[Status: IN_PROGRESS]
        P3 -->|POST /ipd/procedures/{id}/complete| P4[Status: COMPLETED]
    end

    %% Major Surgery Flow
    subgraph Major Surgical OT Session
        S1[Create Surgery Request] -->|POST /ipd/surgeries| S2[Status: REQUESTED]
        S2 -->|POST /ipd/surgeries/{id}/schedule| S3[Status: OT_SCHEDULED]
        S3 -->|POST /ipd/ot-sessions| S4[OT Session: SCHEDULED]
        S4 -->|Sign Consents| S5[OT Consent Clear]
        S5 -->|POST /ipd/ot-sessions/{id}/pac| S6[PAC Cleared]
        S6 -->|POST /ipd/ot-sessions/{id}/pre-op| S7[Status: PRE_OP]
        S7 -->|POST /ipd/ot-sessions/{id}/start| S8[Status: IN_PROGRESS]
        S8 -->|POST /ipd/ot-sessions/{id}/complete| S9[Status: POST_OP]
        S9 -->|POST /ipd/ot-sessions/{id}/transfer-out| S10[Status: COMPLETED]
    end
```

---

## 🗄️ 2. Core Database Schema Context

These workflows map to three core tables in the `ipd` schema. Below are their column configurations and data types:

### Table 1: `ipd.procedure_requests` (Bedside Procedures)
Stores lightweight clinical procedures.
* `id` (UUID, Primary Key, default `gen_random_uuid()`)
* `branch_id` (UUID, Tenant/Branch ID)
* `clinical_escalation_id` (UUID, Foreign Key to `ipd.clinical_escalations`, nullable)
* `admission_id` (UUID, Foreign Key to `ipd.ipd_admissions`)
* `patient_id` (UUID, Foreign Key to `patient.patients`)
* `doctor_id` (UUID, Foreign Key to `identity.users`)
* `procedure_name` (VARCHAR(255))
* `status` (VARCHAR/Enum: `'REQUESTED'`, `'IN_PROGRESS'`, `'COMPLETED'`)
* `outcome` (TEXT, outcome logs)
* `notes` (TEXT, general notes)
* `scheduled_at` / `started_at` / `completed_at` / `created_at` / `updated_at` (TIMESTAMPTZ)

### Table 2: `ipd.surgery_requests` (Surgical Orders)
Maintains high-level order states for surgery requests.
* `id` (UUID, Primary Key, default `gen_random_uuid()`)
* `branch_id` (UUID, Tenant/Branch ID)
* `clinical_escalation_id` (UUID, Foreign Key to `ipd.clinical_escalations`, nullable)
* `admission_id` (UUID, Foreign Key to `ipd.ipd_admissions`)
* `patient_id` (UUID, Foreign Key to `patient.patients`)
* `doctor_id` (UUID, Foreign Key to `identity.users`)
* `procedure_name` (VARCHAR(255))
* `urgency` (VARCHAR/Enum: `'ELECTIVE'`, `'EMERGENCY'`)
* `status` (VARCHAR/Enum: `'REQUESTED'`, `'OT_SCHEDULED'`, `'IN_OT'`, `'COMPLETED'`)
* `ot_room_id` (UUID, target OT room reference)
* `scheduled_at` / `started_at` / `completed_at` / `created_at` / `updated_at` (TIMESTAMPTZ)
* `notes` (TEXT)

### Table 3: `ipd.ot_sessions` (Operating Theatre Live Coordination)
Coordinates live surgery steps inside the Operating Theatre.
* `id` (UUID, Primary Key)
* `branch_id` (UUID, Tenant/Branch ID)
* `service_order_id` (UUID, Foreign Key to `clinical.service_orders`)
* `admission_id` (UUID, Foreign Key to `ipd.ipd_admissions`, nullable)
* `patient_id` (UUID, Foreign Key to `patient.patients`)
* `surgeon_id` (UUID, Foreign Key to `public.sys_users`)
* `anaesthetist_id` (UUID, Foreign Key to `public.sys_users`, nullable)
* `scrub_nurse_id` (UUID, Foreign Key to `public.sys_users`, nullable)
* `pac_cleared` (BOOLEAN, default `FALSE`)
* `pac_cleared_by` (UUID) / `pac_cleared_at` (TIMESTAMPTZ)
* `pre_op_checklist` (JSONB, default `'{}'::jsonb`)
* `pre_op_completed_at` (TIMESTAMPTZ) / `pre_op_completed_by` (UUID)
* `ot_number` (VARCHAR(20)) / `ot_room_id` (VARCHAR(50))
* `scheduled_start` / `scheduled_end` / `surgery_start_at` / `surgery_end_at` (TIMESTAMPTZ)
* `anaesthesia_type` (Enum: `'GENERAL'`, `'SPINAL'`, `'EPIDURAL'`, `'LOCAL'`, `'REGIONAL'`, `'SEDATION'`)
* `intra_op_notes` (TEXT)
* `recovery_start_at` / `recovery_end_at` / `transfer_at` (TIMESTAMPTZ)
* `recovery_notes` (TEXT)
* `transferred_to_ward_id` (UUID, FK to `ipd.wards`)
* `transferred_to_bed_id` (UUID, FK to `ipd.beds`)
* `status` (Enum: `'SCHEDULED'`, `'PRE_OP'`, `'IN_PROGRESS'`, `'POST_OP'`, `'COMPLETED'`)
* `emergency_override` (BOOLEAN, default `FALSE`)
* `override_reason` (TEXT)
* `override_primary_doctor_id` (UUID) / `override_second_doctor_id` (UUID)

---

## 🚀 3. Bedside / Minor Procedures API Contracts

### A. Create Procedure Request
* **Endpoint:** `POST /ipd/procedures`
* **Headers:** `X-Tenant-Id: <tenant-uuid>`
* **Request Body:**
```json
{
  "admission_id": "8b52f36d-92a0-47bf-8f55-1f9e1e123456",
  "patient_id": "f512723d-0d0d-491d-83d6-37f3bcde518d",
  "doctor_id": "ffeee23d-9a29-454f-9f46-192f1aaab285",
  "procedure_id": "010034a7-2bc7-4bd2-9221-75fb580289d0", // Optional (resolves service_name from catalogue)
  "procedure_name": "Laser Sclerotherapy",                   // Required if procedure_id is omitted
  "clinical_escalation_id": "e44d32a1-b8cd-47ef-a152-78ab9c0d12e4", // Optional
  "notes": "Patient requires localized laser therapy for varicose vein management."
}
```
* **Success Response (201 Created):**
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "c7162fa9-e93d-4c3e-8c5e-88c9bb710111",
    "branch_id": "9a12b243-ea77-44df-9e22-83b38c291244",
    "clinical_escalation_id": "e44d32a1-b8cd-47ef-a152-78ab9c0d12e4",
    "admission_id": "8b52f36d-92a0-47bf-8f55-1f9e1e123456",
    "patient_id": "f512723d-0d0d-491d-83d6-37f3bcde518d",
    "doctor_id": "ffeee23d-9a29-454f-9f46-192f1aaab285",
    "procedure_name": "Laser Sclerotherapy",
    "status": "REQUESTED",
    "outcome": null,
    "notes": "Patient requires localized laser therapy for varicose vein management.",
    "scheduled_at": null,
    "started_at": null,
    "completed_at": null,
    "created_at": "2026-06-30T09:30:00.123Z",
    "updated_at": "2026-06-30T09:30:00.123Z"
  }
}
```

### B. Start Procedure Request
* **Endpoint:** `POST /ipd/procedures/{id}/start`
* **Request Body:** `{}`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "c7162fa9-e93d-4c3e-8c5e-88c9bb710111",
    "status": "IN_PROGRESS",
    "started_at": "2026-06-30T10:15:00.000Z"
  }
}
```

### C. Complete Procedure Request
* **Endpoint:** `POST /ipd/procedures/{id}/complete`
* **Request Body:**
```json
{
  "outcome": "SUCCESS",
  "notes": "Laser session completed successfully. Localized swelling observed; ice pack applied."
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "c7162fa9-e93d-4c3e-8c5e-88c9bb710111",
    "status": "COMPLETED",
    "outcome": "SUCCESS",
    "notes": "Laser session completed successfully. Localized swelling observed; ice pack applied.",
    "completed_at": "2026-06-30T10:45:00.000Z"
  }
}
```

---

## 🔪 4. Major Surgery & Operation Theatre API Contracts

### Phase 4.1: Surgery Request Lifecycle

#### A. Create Surgery Request
* **Endpoint:** `POST /ipd/surgeries`
* **Request Body:**
```json
{
  "admission_id": "8b52f36d-92a0-47bf-8f55-1f9e1e123456",
  "patient_id": "f512723d-0d0d-491d-83d6-37f3bcde518d",
  "doctor_id": "ffeee23d-9a29-454f-9f46-192f1aaab285",
  "procedure_id": "020088b9-38cd-47ef-a152-78ab9c0d12e4", // Target surgery lookup from catalogue
  "urgency": "EMERGENCY",                                     // ELECTIVE or EMERGENCY
  "clinical_escalation_id": "e44d32a1-b8cd-47ef-a152-78ab9c0d12e4", // Optional
  "notes": "Acute appendicitis escalation. Needs urgent laparoscopic intervention."
}
```
* **Success Response (201 Created):**
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "a9a8b8c8-ffee-4a3b-9c9c-77d77fef8899",
    "branch_id": "9a12b243-ea77-44df-9e22-83b38c291244",
    "clinical_escalation_id": "e44d32a1-b8cd-47ef-a152-78ab9c0d12e4",
    "admission_id": "8b52f36d-92a0-47bf-8f55-1f9e1e123456",
    "patient_id": "f512723d-0d0d-491d-83d6-37f3bcde518d",
    "doctor_id": "ffeee23d-9a29-454f-9f46-192f1aaab285",
    "procedure_name": "Laparoscopic Appendectomy",
    "urgency": "EMERGENCY",
    "status": "REQUESTED",
    "created_at": "2026-06-30T09:35:00Z"
  }
}
```

#### B. Schedule Surgery Request
* **Endpoint:** `POST /ipd/surgeries/{id}/schedule`
* **Request Body:**
```json
{
  "ot_room_id": "d13543cc-89de-4b11-a83a-888bb1123456",
  "scheduled_at": "2026-06-30T14:30:00Z"
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "a9a8b8c8-ffee-4a3b-9c9c-77d77fef8899",
    "status": "OT_SCHEDULED",
    "ot_room_id": "d13543cc-89de-4b11-a83a-888bb1123456",
    "scheduled_at": "2026-06-30T14:30:00Z"
  }
}
```

---

### Phase 4.2: Operating Theatre (OT) Session Coordination

#### A. Create OT Session
* **Endpoint:** `POST /ipd/ot-sessions`
* **Request Body:**
```json
{
  "service_order_id": "9bc2c1b9-38cd-47ef-a152-78ab9c0d12e4", // Created when booking service orders
  "admission_id": "8b52f36d-92a0-47bf-8f55-1f9e1e123456",
  "patient_id": "f512723d-0d0d-491d-83d6-37f3bcde518d",
  "surgeon_id": "ffeee23d-9a29-454f-9f46-192f1aaab285",
  "anaesthetist_id": "c1f2e3d4-b9c1-4b1a-b36c-ffee00112233",
  "scrub_nurse_id": "99ee88dd-1122-3344-5566-778899aabbcc",
  "ot_number": "OT-ROOM-2",
  "ot_room_id": "OT-ROOM-2",
  "scheduled_start": "2026-06-30T14:30:00Z",
  "scheduled_end": "2026-06-30T16:30:00Z"
}
```
* **Success Response (201 Created):**
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "e7c2a9a9-38cd-47ef-a152-78ab9c0d5122",
    "status": "SCHEDULED",
    "pac_cleared": false,
    "pre_op_checklist": {}
  }
}
```

#### B. Pre-Anesthetic Checkup (PAC) Clearance
Under standard protocols, this endpoint requires all mandatory surgical consents to be signed in the backend. Attempts to call without signed consents return a `400 Bad Request` block.

##### Standard Clearance:
* **Endpoint:** `POST /ipd/ot-sessions/{id}/pac`
* **Request Body:**
```json
{
  "notes": "Lungs clear, cardiovascularly stable. Approved for GENERAL anesthesia."
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "e7c2a9a9-38cd-47ef-a152-78ab9c0d5122",
    "pac_cleared": true,
    "emergency_override": false
  }
}
```

##### Emergency Override (Bypassing Unsigned Consents):
* **Endpoint:** `POST /ipd/ot-sessions/{id}/pac`
* **Request Body:**
```json
{
  "notes": "Consents unavailable due to patient unconsciousness and no immediate kin. Critical emergency.",
  "emergency_override": true,
  "override_reason": "Acute intra-abdominal bleeding requiring emergency life-saving Laparotomy",
  "primary_doctor_id": "ffeee23d-9a29-454f-9f46-192f1aaab285", // Surgeon UUID
  "second_doctor_id": "e4d2a1b9-38cd-47ef-a152-78ab9c0d12e4"     // Independent Attending Doctor UUID
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "e7c2a9a9-38cd-47ef-a152-78ab9c0d5122",
    "pac_cleared": true,
    "emergency_override": true,
    "override_reason": "Acute intra-abdominal bleeding requiring emergency life-saving Laparotomy",
    "override_primary_doctor_id": "ffeee23d-9a29-454f-9f46-192f1aaab285",
    "override_second_doctor_id": "e4d2a1b9-38cd-47ef-a152-78ab9c0d12e4"
  }
}
```

#### C. Complete Pre-Op Checklist
* **Endpoint:** `POST /ipd/ot-sessions/{id}/pre-op`
* **Request Body:**
```json
{
  "pre_op_checklist": {
    "consent_form_verified": true,
    "npo_status_verified": true,
    "surgical_site_marked": true,
    "prosthesis_removed": true,
    "blood_units_reserved": 2
  }
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "e7c2a9a9-38cd-47ef-a152-78ab9c0d5122",
    "status": "PRE_OP"
  }
}
```

#### D. Start OT Session (Starts Surgery / Time-in)
* **Endpoint:** `POST /ipd/ot-sessions/{id}/start`
* **Request Body:** `{}`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "e7c2a9a9-38cd-47ef-a152-78ab9c0d5122",
    "status": "IN_PROGRESS"
  }
}
```

#### E. Complete OT Session (Post-Op Stage / Time-out)
* **Endpoint:** `POST /ipd/ot-sessions/{id}/complete`
* **Request Body:**
```json
{
  "intra_op_notes": "Successful appendectomy. Normal anatomy restored. Fluid drainage: 20ml.",
  "anaesthesia_type": "GENERAL"
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "e7c2a9a9-38cd-47ef-a152-78ab9c0d5122",
    "status": "POST_OP"
  }
}
```

#### F. Transfer Out (Session Completion & Bed Allocation)
Marks recovery phase complete and handles ward/bed transfer. This automatically releases the old bed, schedules it for `MAINTENANCE` cleaning, and assigns the patient to the new ward/bed destination.
* **Endpoint:** `POST /ipd/ot-sessions/{id}/transfer-out`
* **Request Body:**
```json
{
  "transferred_to_ward_id": "c1a2f3f4-b223-4c91-9e22-83b38c29b111", // Target Recovery Ward / General Ward
  "transferred_to_bed_id": "78aa23e4-bb11-477e-a89a-0011bb22cc33",  // Target Bed
  "recovery_notes": "Patient conscious, oriented, vitals stable. Transferred back to bed."
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "e7c2a9a9-38cd-47ef-a152-78ab9c0d5122",
    "status": "COMPLETED"
  }
}
```

#### G. Fetch OT Session Details
* **Endpoint:** `GET /ipd/ot-sessions/{id}`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "e7c2a9a9-38cd-47ef-a152-78ab9c0d5122",
    "status": "COMPLETED",
    "pac_cleared": true,
    "anaesthesia_type": "GENERAL",
    "intra_op_notes": "Successful appendectomy. Normal anatomy restored. Fluid drainage: 20ml.",
    "recovery_notes": "Patient conscious, oriented, vitals stable. Transferred back to bed.",
    "transferred_to_ward_id": "c1a2f3f4-b223-4c91-9e22-83b38c29b111",
    "transferred_to_bed_id": "78aa23e4-bb11-477e-a89a-0011bb22cc33"
  }
}
```

---

## 📑 5. Audit Trail & RLS (Row-Level Security)
Every status change in both Bedside Procedures and Major Surgeries triggers an automatic audit trail logged via `repo.log_audit` capturing:
- `user_id` / `role_id` of the performing clinician.
- `action` (`CREATE`, `START`, `SCHEDULE`, `COMPLETE`, `OT_PAC_CLEARED`).
- `metadata` stores the `old_data` snapshot and `new_data` result.

Additionally, standard PostgreSQL Tenant Isolation (Row Level Security) is active:
- Row access is restricted based on matching `branch_id = current_setting('app.branch_id')`.
