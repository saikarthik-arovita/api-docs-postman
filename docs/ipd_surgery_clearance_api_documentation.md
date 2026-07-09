# Inpatient Department (IPD), OT Surgeries, & Discharge Clearance API Specification

> [!IMPORTANT]
> The OT / Surgery flow has been simplified and consolidated. Please refer to [unified_ot_surgery_flow.md](file:///c:/Users/saika/OneDrive/Desktop/Arovita/ops-hms-ljb/docs/unified_ot_surgery_flow.md) for the latest unified sequence and API quick reference.

This reference document outlines the complete REST API specification for Inpatient Department (IPD) clinical lifecycles, Operating Theatre (OT) surgeries, clinical consent signatures, and multi-department discharge gate clearance workflows.


---

## 📊 Unified Clinical Requests & Recommendations API

To list all clinical recommendations and orders suggested by doctors (Admission requests, Surgery requests, and Bedside procedures) in a single unified view, you can call this API with optional advanced filtering.

* **Endpoint:** `GET /ipd/clinical-requests`
* **Headers:** `X-Tenant-Id: <tenant-uuid>`
* **Query Parameters:**
  * `request_type`: Optional string to filter by type. Options: `ADMISSION`, `SURGERY`, `PROCEDURE` (If omitted, returns all types merged).
  * `status`: Optional filter by request status (e.g. `REQUESTED`).
  * `patient_id`: Optional patient UUID filter.
  * `doctor_id`: Optional doctor UUID filter.
  * `page`: Page index (Default: `1`)
  * `page_size`: Page size limit (Default: `20`)
* **Success Response (200 OK):**
  ```json
  {
    "success": true,
    "code": 200,
    "data": {
      "requests": [
        {
          "request_type": "PROCEDURE",
          "id": "2d1263a8-bbbf-43bb-8de1-b513828cf275",
          "patient_id": "948d1a30-2b20-45be-ac42-005a61c1610e",
          "patient_name": "Admit Test Patient",
          "patient_uhid": "PAT-2026-0038",
          "doctor_id": "97d2a10d-1b8a-40b4-a37a-5daf4e35f630",
          "doctor_name": "Anita Rao",
          "title": "Colonoscopy",
          "status": "REQUESTED",
          "urgency": "ROUTINE",
          "details": null,
          "created_at": "2026-07-01T10:51:22Z"
        },
        {
          "request_type": "SURGERY",
          "id": "7848855e-d0fc-4710-bc94-76afc790cdc7",
          "patient_id": "c2cf0ed3-e8bf-4efe-a8a7-f5815c269b2b",
          "patient_name": "Consent Test Patient",
          "patient_uhid": "PAT-2026-0065",
          "doctor_id": "97d2a10d-1b8a-40b4-a37a-5daf4e35f630",
          "doctor_name": "Anita Rao",
          "title": "Appendectomy",
          "status": "REQUESTED",
          "urgency": "ELECTIVE",
          "details": null,
          "created_at": "2026-07-01T09:58:25Z"
        }
      ],
      "total": 2,
      "page": 1,
      "page_size": 20,
      "total_pages": 1
    }
  }
  ```

---

## 🗺️ 1. Complete Workflow Lifecycle Map

```mermaid
graph TD
    %% Admission Phase
    subgraph "1. Admission Request & Check-In"
        A1[Doctor: Decision to Admit] -->|POST /ipd/admission-requests| A2[Status: REQUESTED]
        A2 -->|PATCH /ipd/admission-requests/{id}/status| A3[Status: READY]
        A3 -->|POST /ipd/admissions| A4[Status: ADMITTED / Inpatient Bed Assigned]
    end

    %% Workflows Phase
    subgraph "2. Inpatient Clinical Care & Workflows"
        A4 -->|Bedside Care| B1[Bedside / Minor Procedures]
        A4 -->|Surgical Care| C1[OT Major Surgeries]
        
        %% Bedside Flow
        B1 -->|POST /ipd/procedures| B2[Status: REQUESTED]
        B2 -->|POST /ipd/procedures/{id}/start| B3[Status: IN_PROGRESS]
        B3 -->|POST /ipd/procedures/{id}/complete| B4[Status: COMPLETED]

        %% Surgery Flow
        C1 -->|POST /ipd/surgeries| C2[Status: REQUESTED]
        C2 -->|POST /ipd/surgeries/{id}/schedule| C3[Status: OT_SCHEDULED]
        C3 -->|POST /ipd/ot-sessions| C4[OT Session: SCHEDULED]
        C4 -->|POST /ipd/ot-cases/{id}/consents| C5[Consents Created]
        C5 -->|POST /ipd/consents/{id}/sign| C6[Patient/Doctor Signed]
        C6 -->|POST /ipd/ot-sessions/{id}/pac| C7[PAC Cleared]
        C7 -->|POST /ipd/ot-sessions/{id}/pre-op| C8[Pre-Op Checklist Completed]
        C8 -->|POST /ipd/ot-sessions/{id}/start| C9[Status: IN_PROGRESS]
        C9 -->|POST /ipd/ot-sessions/{id}/complete| C10[Status: POST_OP]
        C10 -->|POST /ipd/ot-sessions/{id}/transfer-out| C11[Transferred to Inpatient Bed]
    end

    %% Discharge Phase
    subgraph "3. Discharge Gate Clearance"
        C11 -->|POST /ipd/admissions/{id}/discharge/initiate| D1[IPD Status: READY_FOR_DISCHARGE]
        D1 -->|POST /ipd/admissions/{id}/discharge/clearance| D2[Billing Gate: CLEARED]
        D1 -->|POST /ipd/admissions/{id}/discharge/clearance| D3[Nursing Gate: CLEARED]
        D1 -->|POST /ipd/admissions/{id}/discharge/clearance| D4[Pharmacy Gate: CLEARED]
        D1 -->|POST /ipd/admissions/{id}/discharge/clearance| D5[Insurance Gate: CLEARED]
        
        D2 & D3 & D4 & D5 -->|All Gates CLEARED| D6[Gates Status: ALL_CLEARED]
        D6 -->|POST /ipd/admissions/{id}/discharge/complete| D7[IPD Status: DISCHARGED / Bed Released]
    end
```

---

## 🚪 2. Admission Request & Inpatient Check-In APIs

### A. Create Admission Request
When a doctor decides to admit an outpatient from OPD or Emergency, they create a pre-admission request.
* **Endpoint:** `POST /ipd/admission-requests`
* **Headers:** `X-Tenant-Id: <tenant-uuid>`
* **Request Body:**
  ```json
  {
    "patient_id": "c2cf0ed3-e8bf-4efe-a8a7-f5815c269b2b",
    "source_type": "OPD", // Options: "OPD", "EMERGENCY", "REFERRAL" (Default: "OPD")
    "source_reference": "71343aef-5521-4def-87b9-12e2b5c1a111", // Optional (OPD visit / Emergency visit ID)
    "request_reason": "Severe abdominal pain with suspected subacute intestinal obstruction.",
    "admission_type": "ELECTIVE", // Options: "ELECTIVE", "EMERGENCY", "TRANSFER" (Default: "ELECTIVE")
    "priority": "ROUTINE", // Options: "ROUTINE", "URGENT", "EMERGENCY" (Default: "ROUTINE")
    "preferred_ward_id": "4c5d12d2-79dd-4650-867e-079ac3b85a5a", // Optional
    "preferred_bed_type": "STANDARD", // Optional (Default: None)
    "notes": "Monitor vitals every 4 hours" // Optional
  }
  ```
* **Success Response (201 Created):**
  ```json
  {
    "success": true,
    "code": 201,
    "data": {
      "id": "7848855e-d0fc-4710-bc94-76afc790cdc7",
      "tenant_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
      "patient_id": "c2cf0ed3-e8bf-4efe-a8a7-f5815c269b2b",
      "source_type": "OPD",
      "source_reference": "71343aef-5521-4def-87b9-12e2b5c1a111",
      "request_reason": "Severe abdominal pain with suspected subacute intestinal obstruction.",
      "admission_type": "ELECTIVE",
      "priority": "ROUTINE",
      "status": "REQUESTED",
      "preferred_ward_id": "4c5d12d2-79dd-4650-867e-079ac3b85a5a",
      "preferred_bed_type": "STANDARD",
      "notes": "Monitor vitals every 4 hours",
      "created_at": "2026-07-01T05:00:00Z",
      "updated_at": "2026-07-01T05:00:00Z"
    }
  }
  ```

### B. Update Admission Request Status
Updates the review status of the admission request.
* **Endpoint:** `PATCH /ipd/admission-requests/{request_id}/status`
* **Headers:** `X-Tenant-Id: <tenant-uuid>`
* **Request Body:**
  ```json
  {
    "status": "READY", // Options: "UNDER_REVIEW", "READY", "CANCELLED", "EXPIRED"
    "reason": "Bed availability confirmed in Male Medical General Ward." // Optional
  }
  ```
* **Success Response (200 OK):**
  ```json
  {
    "success": true,
    "code": 200,
    "data": {
      "id": "7848855e-d0fc-4710-bc94-76afc790cdc7",
      "status": "READY",
      "notes": "Bed availability confirmed in Male Medical General Ward."
    }
  }
  ```

### C. Create Inpatient Admission (Check-In)
Converts a ready admission request or registers a standalone patient directly into a ward and bed.
* **Endpoint:** `POST /ipd/admissions`
* **Headers:** `X-Tenant-Id: <tenant-uuid>`
* **Request Body:**
  ```json
  {
    "patient_id": "c2cf0ed3-e8bf-4efe-a8a7-f5815c269b2b",
    "admission_request_id": "7848855e-d0fc-4710-bc94-76afc790cdc7", // Optional (links to request)
    "opd_visit_id": "71343aef-5521-4def-87b9-12e2b5c1a111", // Optional
    "emergency_visit_id": null, // Optional
    "ward_id": "4c5d12d2-79dd-4650-867e-079ac3b85a5a", // General Medical Ward
    "bed_id": "84820843-9876-4321-9abc-1234567890ab", // Bed number B-GEN-01
    "attending_doctor_id": "97d2a10d-1b8a-40b4-a37a-5daf4e35f630", // Anita Rao
    "admission_type": "ELECTIVE", // Options: "ELECTIVE", "EMERGENCY", "TRANSFER"
    "admission_reason": "Suspected subacute intestinal obstruction, scheduled for diagnostic evaluation.",
    "expected_discharge": "2026-07-06", // Optional (Date format: YYYY-MM-DD)
    "notes": "NPO (nil per os) after midnight for observation" // Optional
  }
  ```
* **Success Response (201 Created):**
  ```json
  {
    "success": true,
    "code": 201,
    "data": {
      "id": "81463aef-7253-4bef-87b9-31e2b5c1a928",
      "admission_number": "IPD-2026-00042",
      "patient_id": "c2cf0ed3-e8bf-4efe-a8a7-f5815c269b2b",
      "ward_id": "4c5d12d2-79dd-4650-867e-079ac3b85a5a",
      "bed_id": "84820843-9876-4321-9abc-1234567890ab",
      "attending_doctor_id": "97d2a10d-1b8a-40b4-a37a-5daf4e35f630",
      "admission_date": "2026-07-01T06:15:00Z",
      "status": "ADMITTED",
      "ipd_status": "ADMITTED"
    }
  }
  ```

---

## 🩺 3. Bedside / Minor Procedures APIs

### A. List Procedure Requests
Retrieves bedside procedure requests for the branch/tenant with support for pagination and filtering.
* **Endpoint:** `GET /ipd/procedures`
* **Headers:** `X-Tenant-Id: <tenant-uuid>`
* **Query Parameters:**
  * `status`: Optional filter by status (e.g. `REQUESTED`, `IN_PROGRESS`, `COMPLETED`, `CANCELLED`)
  * `patient_id`: Optional filter by patient ID
  * `doctor_id`: Optional filter by doctor ID
  * `page`: Page index (Default: `1`)
  * `page_size`: Page size limit (Default: `20`)
* **Success Response (200 OK):**
  ```json
  {
    "success": true,
    "code": 200,
    "data": {
      "requests": [
        {
          "id": "2d1263a8-bbbf-43bb-8de1-b513828cf275",
          "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
          "clinical_escalation_id": null,
          "admission_id": "5a58fc86-4e36-4f6b-a69a-c06cedbe8f46",
          "patient_id": "948d1a30-2b20-45be-ac42-005a61c1610e",
          "doctor_id": "97d2a10d-1b8a-40b4-a37a-5daf4e35f630",
          "procedure_name": "Colonoscopy",
          "status": "REQUESTED",
          "outcome": null,
          "notes": null,
          "scheduled_at": null,
          "started_at": null,
          "completed_at": null,
          "created_at": "2026-07-01T10:51:22Z",
          "updated_at": "2026-07-01T10:51:22Z",
          "patient_name": "Admit Test Patient",
          "patient_uhid": "PAT-2026-0038",
          "doctor_name": "Anita Rao"
        }
      ],
      "total": 1,
      "page": 1,
      "page_size": 20,
      "total_pages": 1
    }
  }
  ```

### B. Create Procedure Request
Lightweight bedside clinical procedure requests mapped to a patient admission.
* **Endpoint:** `POST /ipd/procedures`
* **Request Body:**
  ```json
  {
    "admission_id": "81463aef-7253-4bef-87b9-31e2b5c1a928",
    "patient_id": "c2cf0ed3-e8bf-4efe-a8a7-f5815c269b2b",
    "doctor_id": "97d2a10d-1b8a-40b4-a37a-5daf4e35f630",
    "procedure_name": "Lumbar Puncture", // Can pass procedure_id (catalogue ID) instead
    "notes": "Diagnostic tap for CSF analysis." // Optional
  }
  ```
* **Success Response (201 Created):**
  ```json
  {
    "success": true,
    "code": 201,
    "data": {
      "id": "e44d32a1-b8cd-47ef-a152-78ab9c0d12e4",
      "procedure_name": "Lumbar Puncture",
      "status": "REQUESTED",
      "created_at": "2026-07-01T06:30:00Z"
    }
  }
  ```

### C. Start Bedside Procedure
* **Endpoint:** `POST /ipd/procedures/{id}/start`
* **Success Response (200 OK):**
  ```json
  {
    "success": true,
    "code": 200,
    "data": {
      "id": "e44d32a1-b8cd-47ef-a152-78ab9c0d12e4",
      "status": "IN_PROGRESS",
      "started_at": "2026-07-01T06:45:00Z"
    }
  }
  ```

### D. Complete Bedside Procedure
* **Endpoint:** `POST /ipd/procedures/{id}/complete`
* **Request Body:**
  ```json
  {
    "outcome": "Successful diagnostic tap. Obtained 5ml clear CSF fluid, sent to pathology lab.",
    "notes": "Patient tolerated the procedure well, no immediate post-procedure complaints." // Optional
  }
  ```
* **Success Response (200 OK):**
  ```json
  {
    "success": true,
    "code": 200,
    "data": {
      "id": "e44d32a1-b8cd-47ef-a152-78ab9c0d12e4",
      "status": "COMPLETED",
      "outcome": "Successful diagnostic tap. Obtained 5ml clear CSF fluid, sent to pathology lab.",
      "completed_at": "2026-07-01T07:00:00Z"
    }
  }
  ```

---

## 🏛️ 4. Major OT Surgeries & OT Session Lifecycle APIs

### A. List Surgery Requests
Retrieves surgery requests/orders for the branch/tenant with support for pagination and filtering.
* **Endpoint:** `GET /ipd/surgeries`
* **Headers:** `X-Tenant-Id: <tenant-uuid>`
* **Query Parameters:**
  * `status`: Optional filter by status (e.g. `REQUESTED`, `OT_SCHEDULED`, `COMPLETED`, `CANCELLED`)
  * `urgency`: Optional filter by urgency (e.g. `ELECTIVE`, `URGENT`, `EMERGENCY`)
  * `patient_id`: Optional filter by patient ID
  * `doctor_id`: Optional filter by doctor ID
  * `page`: Page index (Default: `1`)
  * `page_size`: Page size limit (Default: `20`)
* **Success Response (200 OK):**
  ```json
  {
    "success": true,
    "code": 200,
    "data": {
      "requests": [
        {
          "id": "7848855e-d0fc-4710-bc94-76afc790cdc7",
          "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
          "clinical_escalation_id": null,
          "admission_id": "81463aef-7253-4bef-87b9-31e2b5c1a928",
          "patient_id": "c2cf0ed3-e8bf-4efe-a8a7-f5815c269b2b",
          "doctor_id": "97d2a10d-1b8a-40b4-a37a-5daf4e35f630",
          "procedure_name": "Appendectomy",
          "urgency": "ELECTIVE",
          "status": "REQUESTED",
          "ot_room_id": null,
          "scheduled_at": null,
          "started_at": null,
          "completed_at": null,
          "notes": null,
          "created_at": "2026-07-01T09:58:25Z",
          "updated_at": "2026-07-01T09:58:25Z",
          "patient_name": "Consent Test Patient",
          "patient_uhid": "PAT-2026-0065",
          "doctor_name": "Anita Rao"
        }
      ],
      "total": 1,
      "page": 1,
      "page_size": 20,
      "total_pages": 1
    }
  }
  ```

### B. Create Surgery Request
Creates a surgical log entry under the patient's active inpatient admission.
* **Endpoint:** `POST /ipd/surgeries`
* **Request Body:**
  ```json
  {
    "admission_id": "81463aef-7253-4bef-87b9-31e2b5c1a928",
    "patient_id": "c2cf0ed3-e8bf-4efe-a8a7-f5815c269b2b",
    "doctor_id": "97d2a10d-1b8a-40b4-a37a-5daf4e35f630",
    "procedure_name": "Appendectomy", // Can pass procedure_id (catalogue ID) instead
    "urgency": "ELECTIVE", // Options: "ELECTIVE", "URGENT", "EMERGENCY" (Default: "ELECTIVE")
    "notes": "Laparoscopic appendectomy planned." // Optional
  }
  ```
* **Success Response (201 Created):**
  ```json
  {
    "success": true,
    "code": 201,
    "data": {
      "id": "7848855e-d0fc-4710-bc94-76afc790cdc7",
      "procedure_name": "Appendectomy",
      "urgency": "ELECTIVE",
      "status": "REQUESTED",
      "created_at": "2026-07-01T07:15:00Z"
    }
  }
  ```

### C. Schedule Surgery Request
Sets the Operation Theatre room and schedule time.
* **Endpoint:** `POST /ipd/surgeries/{id}/schedule`
* **Request Body:**
  ```json
  {
    "ot_room_id": "OT-3", // Handled as string identifier (VARCHAR(50))
    "scheduled_at": "2026-07-02T09:00:00.000Z"
  }
  ```
* **Success Response (200 OK):**
  ```json
  {
    "success": true,
    "code": 200,
    "data": {
      "id": "7848855e-d0fc-4710-bc94-76afc790cdc7",
      "ot_room_id": "OT-3",
      "scheduled_at": "2026-07-02T09:00:00Z",
      "status": "OT_SCHEDULED"
    }
  }
  ```

### D. Create Operating Theatre (OT) Session
Once scheduled, staff create an active OT session mapping the clinical crew and room.
* **Endpoint:** `POST /ipd/ot-sessions`
* **Request Body:**
  ```json
  {
    "service_order_id": "7848855e-d0fc-4710-bc94-76afc790cdc7", // Target Surgery Request ID
    "admission_id": "81463aef-7253-4bef-87b9-31e2b5c1a928",
    "patient_id": "c2cf0ed3-e8bf-4efe-a8a7-f5815c269b2b",
    "surgeon_id": "97d2a10d-1b8a-40b4-a37a-5daf4e35f630",
    "anaesthetist_id": "f824ed84-6dfd-421e-9cde-24b29d47ebb6", // Optional
    "scrub_nurse_id": "ffeee23d-9a29-454f-9f46-192f1aaab285", // Optional
    "ot_number": "OT-101", // Optional
    "ot_room_id": "OT-3", // Optional
    "scheduled_start": "2026-07-02T09:00:00.000Z", // Optional
    "scheduled_end": "2026-07-02T11:00:00.000Z" // Optional
  }
  ```
* **Success Response (201 Created):**
  ```json
  {
    "success": true,
    "code": 201,
    "data": {
      "id": "e58d2044-e0eb-4dcf-90c9-fb2d663a427b",
      "service_order_id": "7848855e-d0fc-4710-bc94-76afc790cdc7",
      "status": "SCHEDULED",
      "pac_cleared": false,
      "pre_op_checklist": {}
    }
  }
  ```

### E. OT Consent Management
A patient cannot undergo surgery without active signed consents, unless an emergency override is recorded.

#### 1. Generate/Create OT Consents
* **Endpoint:** `POST /ipd/ot-cases/{ot_session_id}/consents`
* **Request Body:**
  ```json
  {
    "consent_codes": ["SURGICAL_CONSENT", "ANESTHESIA_CONSENT"], // Optional array
    "consents": [
      {
        "consent_code": "SURGICAL_CONSENT",
        "mandatory": true
      }
    ] // Optional array of custom definitions
  }
  ```
* **Success Response (201 Created):**
  ```json
  {
    "success": true,
    "code": 201,
    "data": [
      {
        "id": "c0b7ae69-d4f1-48e8-b527-284c60be2826",
        "consent_code": "SURGICAL_CONSENT",
        "status": "PENDING",
        "mandatory": true
      }
    ]
  }
  ```

#### 2. Sign Consent Form
* **Endpoint:** `POST /ipd/consents/{consent_id}/sign`
* **Request Body:**
  ```json
  {
    "signature_type": "DIGITAL", // "DIGITAL" or "PHYSICAL"
    "signed_by": "PATIENT",
    "guardian_name": null, // Optional
    "guardian_relation": null, // Optional
    "doctor_id": "97d2a10d-1b8a-40b4-a37a-5daf4e35f630", // Anita Rao (Witnessing)
    "witness_id": null, // Optional
    "remarks": "Signed with understanding of all risks and benefits." // Optional
  }
  ```
* **Success Response (200 OK):**
  ```json
  {
    "success": true,
    "data": {
      "id": "c0b7ae69-d4f1-48e8-b527-284c60be2826",
      "status": "SIGNED",
      "signed_at": "2026-07-01T07:30:00Z"
    }
  }
  ```

### F. Pre-Anesthetic Checkup (PAC) Clearance
* **Endpoint:** `POST /ipd/ot-sessions/{ot_session_id}/pac`
* **Request Body (Standard Clearance):**
  ```json
  {
    "notes": "PAC cleared. Airway Mallampati Class I, cleared for General Anesthesia.",
    "emergency_override": false
  }
  ```
* **Request Body (Emergency Override):**
  ```json
  {
    "notes": "Patient in acute shock, bypass standard PAC checklist.",
    "emergency_override": true,
    "override_reason": "Acute appendicitis with septic shock.",
    "primary_doctor_id": "97d2a10d-1b8a-40b4-a37a-5daf4e35f630",
    "second_doctor_id": "f824ed84-6dfd-421e-9cde-24b29d47ebb6" // Double sign-off required
  }
  ```
* **Success Response (200 OK):**
  ```json
  {
    "success": true,
    "data": {
      "id": "e58d2044-e0eb-4dcf-90c9-fb2d663a427b",
      "pac_cleared": true,
      "pac_cleared_by": "f824ed84-6dfd-421e-9cde-24b29d47ebb6",
      "pac_cleared_at": "2026-07-01T07:45:00Z"
    }
  }
  ```

### G. Complete Pre-Op Checklist
* **Endpoint:** `POST /ipd/ot-sessions/{ot_session_id}/pre-op`
* **Request Body:**
  ```json
  {
    "pre_op_checklist": {
      "npo_verified": true,
      "surgical_site_marked": true,
      "consents_verified": true,
      "jewelry_removed": true,
      "dentures_removed": true,
      "vitals_stable": true
    }
  }
  ```
* **Success Response (200 OK):**
  ```json
  {
    "success": true,
    "data": {
      "id": "e58d2044-e0eb-4dcf-90c9-fb2d663a427b",
      "status": "PRE_OP",
      "pre_op_checklist": {
        "npo_verified": true,
        "surgical_site_marked": true,
        "consents_verified": true,
        "jewelry_removed": true,
        "dentures_removed": true,
        "vitals_stable": true
      },
      "pre_op_completed_at": "2026-07-01T08:00:00Z"
    }
  }
  ```

### H. Start OT Session
* **Endpoint:** `POST /ipd/ot-sessions/{ot_session_id}/start`
* **Success Response (200 OK):**
  ```json
  {
    "success": true,
    "data": {
      "id": "e58d2044-e0eb-4dcf-90c9-fb2d663a427b",
      "status": "IN_PROGRESS",
      "surgery_start_at": "2026-07-01T08:15:00Z"
    }
  }
  ```

### I. Complete OT Session
Logs anesthesia parameters and surgical notes.
* **Endpoint:** `POST /ipd/ot-sessions/{ot_session_id}/complete`
* **Request Body:**
  ```json
  {
    "intra_op_notes": "Laparoscopic appendectomy executed. Appendix was inflamed and non-perforated. Successfully excised.",
    "anaesthesia_type": "GENERAL" // Options: "GENERAL", "SPINAL", "EPIDURAL", "LOCAL", "REGIONAL", "SEDATION"
  }
  ```
* **Success Response (200 OK):**
  ```json
  {
    "success": true,
    "data": {
      "id": "e58d2044-e0eb-4dcf-90c9-fb2d663a427b",
      "status": "POST_OP",
      "surgery_end_at": "2026-07-01T09:30:00Z",
      "anaesthesia_type": "GENERAL",
      "intra_op_notes": "Laparoscopic appendectomy executed. Appendix was inflamed and non-perforated. Successfully excised."
    }
  }
  ```

### J. Transfer Out from OT (Recovery to Bed Transfer)
Releases the OT room and moves the patient back to a recovery or ward bed.
* **Endpoint:** `POST /ipd/ot-sessions/{ot_session_id}/transfer-out`
* **Request Body:**
  ```json
  {
    "transferred_to_ward_id": "4100a692-6605-452d-baca-314892ba002d", // Intensive Care Unit (ICU)
    "transferred_to_bed_id": "fb30a5d4-65d6-42c2-9717-e82629dd7dde", // ICU-Bed-01
    "recovery_notes": "Patient stable, consciousness regained, shifting to ICU bed for close vitals monitoring." // Optional
  }
  ```
* **Success Response (200 OK):**
  ```json
  {
    "success": true,
    "data": {
      "id": "e58d2044-e0eb-4dcf-90c9-fb2d663a427b",
      "status": "COMPLETED",
      "recovery_end_at": "2026-07-01T10:00:00Z",
      "transferred_to_ward_id": "4100a692-6605-452d-baca-314892ba002d",
      "transferred_to_bed_id": "fb30a5d4-65d6-42c2-9717-e82629dd7dde",
      "transfer_at": "2026-07-01T10:00:00Z"
    }
  }
  ```

---

## 🏁 5. Discharge Gates Clearance Workflow APIs

Once the clinical recovery is complete, the patient cannot leave the hospital premises until all clearance gates (Billing, Nursing, Pharmacy, Insurance) are cleared.

### A. Initiate Discharge
The attending doctor registers the decision to discharge the patient.
* **Endpoint:** `POST /ipd/admissions/{admission_id}/discharge/initiate`
* **Request Body:**
  ```json
  {
    "discharge_summary": "Patient recovered well from laparoscopic appendectomy. Wound healing is healthy without discharge.",
    "follow_up_date": "2026-07-08", // Optional (Date format: YYYY-MM-DD)
    "follow_up_instructions": "Review in surgical OPD next Wednesday. Liquid/soft diet for 3 days, avoid heavy lifting.", // Optional
    "notes": "Start standard discharge medication packing." // Optional
  }
  ```
* **Success Response (200 OK):**
  ```json
  {
    "success": true,
    "code": 200,
    "data": {
      "admission_id": "81463aef-7253-4bef-87b9-31e2b5c1a928",
      "admission_number": "IPD-2026-00042",
      "patient_id": "c2cf0ed3-e8bf-4efe-a8a7-f5815c269b2b",
      "status": "DISCHARGING",
      "ipd_status": "READY_FOR_DISCHARGE"
    }
  }
  ```

### B. Fetch Discharge Clearance Status (Check Gates status)
Provides the status of each check gate before departure.
* **Endpoint:** `GET /ipd/admissions/{admission_id}/dama` or `GET /ipd/admissions/{admission_id}`
* **Headers:** `X-Tenant-Id: <tenant-uuid>`
* **Success Response (200 OK):**
  ```json
  {
    "success": true,
    "data": {
      "admission_id": "81463aef-7253-4bef-87b9-31e2b5c1a928",
      "ipd_status": "READY_FOR_DISCHARGE",
      "clearance_gates": [
        { "gate_name": "BILLING", "status": "PENDING", "cleared_by": null, "cleared_at": null, "blocking_reason": null },
        { "gate_name": "NURSING", "status": "PENDING", "cleared_by": null, "cleared_at": null, "blocking_reason": null },
        { "gate_name": "PHARMACY", "status": "PENDING", "cleared_by": null, "cleared_at": null, "blocking_reason": null },
        { "gate_name": "INSURANCE", "status": "PENDING", "cleared_by": null, "cleared_at": null, "blocking_reason": null }
      ]
    }
  }
  ```

### C. Update Department Gate Clearance Status
Each department (Billing, Nursing, Pharmacy, Insurance) submits their sign-off. If blocked, a reason must be provided.
* **Endpoint:** `POST /ipd/admissions/{admission_id}/discharge/clearance`
* **Request Body:**
  ```json
  {
    "gate_name": "BILLING", // Options: "BILLING", "NURSING", "PHARMACY", "INSURANCE"
    "gate_status": "CLEARED", // Options: "PENDING", "CLEARED", "BLOCKED"
    "blocking_reason": null // Required if gate_status is "BLOCKED"
  }
  ```
* **Success Response (200 OK):**
  ```json
  {
    "success": true,
    "data": {
      "admission_id": "81463aef-7253-4bef-87b9-31e2b5c1a928",
      "gate_name": "BILLING",
      "status": "CLEARED",
      "cleared_by": "f824ed84-6dfd-421e-9cde-24b29d47ebb6",
      "cleared_at": "2026-07-01T10:15:00Z"
    }
  }
  ```

### D. Complete Discharge (Gates Cleared)
Releases the bed and completes the admission closure. This call is only allowed when **all 4 gates** are marked as `CLEARED`.
* **Endpoint:** `POST /ipd/admissions/{admission_id}/discharge/complete`
* **Request Body:**
  ```json
  {
    "discharge_summary": "Patient recovered well from laparoscopic appendectomy. Discharge instructions provided.",
    "follow_up_date": "2026-07-08", // Optional
    "follow_up_instructions": "Review in surgical OPD in 1 week.", // Optional
    "medications_on_discharge": "Tab Paracetamol 650mg TDS x 3 days, Tab Cefixime 200mg BD x 5 days", // Optional
    "diet_instructions": "Light soft food, avoid oily/spicy diet." // Optional
  }
  ```
* **Success Response (200 OK):**
  ```json
  {
    "success": true,
    "data": {
      "admission_id": "81463aef-7253-4bef-87b9-31e2b5c1a928",
      "admission_number": "IPD-2026-00042",
      "patient_id": "c2cf0ed3-e8bf-4efe-a8a7-f5815c269b2b",
      "discharge_type": "NORMAL",
      "status": "DISCHARGED",
      "ipd_status": "DISCHARGED",
      "actual_discharge_at": "2026-07-01T10:20:00Z",
      "length_of_stay_days": 1
    }
  }
  ```
