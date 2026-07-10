# Hospital Management System (HMS): End-to-End API Integration & Workflow Guide

This document is a comprehensive, step-by-step API integration guide for the Hospital Management System (HMS) patient journeys. It provides exact HTTP contracts, request/response payload examples, field validation rules, database mappings, and transitional logic for three core clinical pathways:

1. **Scenario 1 (Elective Flow):** Patient Registration → OPD Appointment → OPD Consultation → IPD Admission → OT Surgery → Discharge & Billing.
2. **Scenario 2 (Referral Flow):** External Referral Registration → Direct IPD Admission → Surgery Advice → OT Surgery → Discharge & Billing.
3. **Scenario 3 (Emergency Flow):** Patient Registration → ER Visit & Bed Assignment → Emergency IPD Admission → Emergency PAC Override → OT Surgery → Post-Op ICU Transfer → Discharge & Billing.

---

## 🔑 1. Global API Conventions

### Required Headers
All API calls must include the following headers for authentication, authorization, and multi-tenant isolation:
```http
Authorization: Bearer <access_token>
Content-Type: application/json
X-Tenant-Id: <tenant-uuid>
```
* **Authentication & Authorization:** Decoded JWT bearer token must contain the user's Cognito `sub` (mapping to `public.sys_users.cognito_sub`) and user must be active (`is_active = true`). Endpoint access is role-gated.
* **Tenant Isolation:** The `X-Tenant-Id` header is used to enforce Multi-Tenant Row-Level Security (RLS) across all database schemas.

### Standard Response Envelope

#### Success Response
```json
{
  "success": true,
  "code": 200, // or 201
  "message": "Operation completed successfully",
  "data": {}
}
```

#### Error Response
```json
{
  "success": false,
  "code": 400, // 400, 401, 403, 404, 409, 422, 500
  "error": "Detailed validation or business error message here"
}
```

---

## 📊 2. Global Workflow States & Status Enums

The clinical lifecycle progresses through the following workflow states and database statuses:

| Workflow State | Description | DB Table & Status Column |
| :--- | :--- | :--- |
| `REGISTERED` | Patient profile created | `patient.patients.is_active = true` |
| `APPOINTMENT_CREATED` | OPD consultation booked | `clinical.appointments.status = 'SCHEDULED'` |
| `WAITING` | Consultation fee paid; waiting in queue | `clinical.appointments.status = 'CONFIRMED'`, `clinical.opd_visits.status = 'ARRIVED'` |
| `IN_CONSULTATION` | Consultation session active | `clinical.opd_visits.status = 'IN_PROGRESS'` |
| `OPD_COMPLETED` | Consultation checked out | `clinical.opd_visits.status = 'COMPLETED'` |
| `ADMISSION_REQUESTED` | Doctor recommended inpatient admission | `ipd.admission_requests.status = 'REQUESTED'` |
| `ADMISSION_APPROVED` | Inpatient bed & insurance verified | `ipd.admission_requests.status = 'READY'` |
| `ADMITTED` | Patient checked into ward bed | `ipd.ipd_admissions.status = 'ADMITTED'`, `ipd_status = 'ADMITTED'` |
| `BED_ASSIGNED` | Assigned ward bed occupied | `ipd.beds.status = 'OCCUPIED'` |
| `SURGERY_ORDERED` | Surgery advice recorded | `clinical.service_orders.status = 'ORDERED'` |
| `OT_SCHEDULED` | OT Session created & team assigned | `ipd.ot_sessions.status = 'SCHEDULED'`, `clinical.service_orders.status = 'SCHEDULED'` |
| `PRE_OP` | PAC cleared & checklist completed | `ipd.ot_sessions.status = 'PRE_OP'` |
| `IN_OT` | Patient on OT table; surgery active | `ipd.ot_sessions.status = 'IN_PROGRESS'`, `clinical.service_orders.status = 'IN_PROGRESS'` |
| `SURGERY_COMPLETED` | Incision closed; patient in recovery | `ipd.ot_sessions.status = 'POST_OP'` |
| `RECOVERY` | Post-operative recovery monitoring | `ipd.ot_sessions.status = 'POST_OP'` |
| `POST_OP` | Transferred back to inpatient ward bed | `ipd.ot_sessions.status = 'COMPLETED'`, `clinical.service_orders.status = 'COMPLETED'`, `ipd.ipd_admissions.ipd_status = 'UNDER_TREATMENT'` |
| `READY_FOR_DISCHARGE` | Discharge summary written; billing cleared | `ipd.ipd_admissions.ipd_status = 'READY_FOR_DISCHARGE'` |
| `DISCHARGED` | Patient checked out; bed released | `ipd.ipd_admissions.status = 'DISCHARGED'`, `ipd.ipd_admissions.ipd_status = 'DISCHARGED'`, `ipd.beds.status = 'AVAILABLE'` |

---

# 🛣️ Flow 1: Patient Registration → OPD → IPD Admission → Surgery → Discharge

This represents the standard elective surgical pathway. An outpatient registers, undergoes consultation, is recommended for surgery, gets admitted, undergoes the procedure, settles billing, and is discharged.

---

## Step 1: Register Patient
* **Purpose:** Create a new patient profile and assign a Unique Health Identification (UHID) / Medical Record Number (MRN).
* **HTTP Method:** `POST`
* **Endpoint:** `/patients/register`
* **Required Permission:** `patients:create`
* **Preconditions:** None.
* **Postconditions:** A new patient record is created in the database.
* **Affected DB Entities:** `patient.patients` (inserts patient profile), `patient.addresses` (optional, inserts address).

### Request
* **Body Schema:**
  * `full_name` (String, Required): Patient's full name. Length: 2 to 100 characters.
  * `phone` (String, Required): Contact number. Must be a valid format (e.g., minimum 10 digits).
  * `dob` (String - Date, Optional): Date of birth (YYYY-MM-DD format).
  * `age` (Integer, Required): Age in years. Valid range: 0 to 150.
  * `gender` (String, Optional): Valid values: `MALE`, `FEMALE`, `OTHER`, `UNKNOWN` (default: `UNKNOWN`).
  * `blood_group` (String, Optional): Valid values: `A+`, `A-`, `B+`, `B-`, `AB+`, `AB-`, `O+`, `O-`, `UNKNOWN` (default: `UNKNOWN`).
  * `address` (Object, Optional):
    * `line1` (String, Optional): Street address (max 255 chars).
    * `city` (String, Optional): City name (max 100 chars).
    * `state` (String, Optional): State name (max 100 chars).
    * `pincode` (String, Optional): Postal code (max 20 chars).
  * `abha_number` (String, Optional): Government ABHA card ID (max 100 chars).

#### Request Example
```json
{
  "full_name": "Ramesh Sharma",
  "phone": "+919876543201",
  "dob": "1978-08-12",
  "age": 47,
  "gender": "MALE",
  "blood_group": "A+",
  "address": {
    "line1": "Flat 202, Sunshine Apts",
    "city": "Mumbai",
    "state": "Maharashtra",
    "pincode": "400001"
  }
}
```

### Response
* **Success (201 Created):**
```json
{
  "success": true,
  "code": 201,
  "message": "Patient registered successfully",
  "data": {
    "id": "e2c07a01-2092-4876-8889-112233445566",
    "uhid": "PAT-2026-0089",
    "full_name": "Ramesh Sharma",
    "phone": "+919876543201",
    "dob": "1978-08-12",
    "age": 47,
    "gender": "MALE",
    "blood_group": "A+",
    "is_active": true,
    "created_at": "2026-07-10T11:48:00Z"
  }
}
```
* **Error Responses:**
  * **400 Bad Request:** Validation error (e.g., age negative or name too short).
  * **409 Conflict:** Patient with this phone number already exists.

---

### ➔ Transition: Step 1 to Step 2
* **Why Next API is Required:** To allocate a doctor slot and time for the patient's initial consultation.
* **IDs Carried Forward:** `patient_id` (`data.id` from Step 1).
* **Workflow State Change:** `REGISTERED`
* **Required Permissions:** `appointments:create`
* **Validations:** Verify `patient_id` is a valid, active UUID in the database.

---

## Step 2: Book OPD Appointment
* **Purpose:** Schedule a diagnostic or consultation visit with a consulting physician/surgeon.
* **HTTP Method:** `POST`
* **Endpoint:** `/opd/appointments`
* **Required Permission:** `appointments:create`
* **Preconditions:** Patient must be registered.
* **Postconditions:** An appointment slot is reserved with status `SCHEDULED`.
* **Affected DB Entities:** `clinical.appointments` (inserts appointment details).

### Request
* **Body Schema:**
  * `patient_id` (UUID, Required): ID generated in Step 1.
  * `doctor_id` (UUID, Required): Consultant/Surgeon ID.
  * `department_id` (UUID, Optional): Specialty department ID.
  * `appointment_time` (String - ISO 8601, Required): Format: `YYYY-MM-DDTHH:MM:SSZ`.
  * `appointment_type` (String, Optional): Valid values: `NEW`, `FOLLOW_UP` (default: `NEW`).
  * `priority` (Integer, Optional): Valid values: `1` (Routine), `2` (Urgent), `3` (Emergency) (default: `1`).
  * `notes` (String, Optional): Clinical check-in note.

#### Request Example
```json
{
  "patient_id": "e2c07a01-2092-4876-8889-112233445566",
  "doctor_id": "97d2a10d-1b8a-40b4-a37a-5daf4e35f630",
  "department_id": "d1234567-abcd-efgh-ijkl-123456789012",
  "appointment_time": "2026-07-11T10:00:00Z",
  "appointment_type": "NEW",
  "priority": 1,
  "notes": "Complains of persistent abdominal distress"
}
```

### Response
* **Success (201 Created):**
```json
{
  "success": true,
  "code": 201,
  "message": "OPD Appointment booked",
  "data": {
    "id": "a9a9b8b8-c7c7-6d6d-5e5e-4f4f3f3f2e2e",
    "patient_id": "e2c07a01-2092-4876-8889-112233445566",
    "doctor_id": "97d2a10d-1b8a-40b4-a37a-5daf4e35f630",
    "department_id": "d1234567-abcd-efgh-ijkl-123456789012",
    "appointment_time": "2026-07-11T10:00:00Z",
    "token_number": "T-102",
    "status": "SCHEDULED",
    "created_at": "2026-07-10T11:50:00Z"
  }
}
```
* **Error Responses:**
  * **400 Bad Request:** Missing fields or slot is in the past.
  * **409 Conflict:** Doctor schedule conflict.

---

### ➔ Transition: Step 2 to Step 3
* **Why Next API is Required:** To process the payment of the OPD consultation fee and confirm the patient's arrival at the clinic.
* **IDs Carried Forward:** `appointment_id` (`data.id` from Step 2).
* **Workflow State Change:** `APPOINTMENT_CREATED`
* **Required Permissions:** `appointments:confirm`
* **Validations:** The appointment must be in `SCHEDULED` status and not yet cancelled or confirmed.

---

## Step 3: Confirm Appointment
* **Purpose:** Mark the appointment as paid and active, which automatically triggers the creation of an OPD visit log.
* **HTTP Method:** `POST`
* **Endpoint:** `/opd/appointments/{id}/confirm`
* **Required Permission:** `appointments:confirm`
* **Preconditions:** Appointment must be scheduled.
* **Postconditions:** Appointment moves to `CONFIRMED` and an `opd_visits` record is initialized with status `ARRIVED`.
* **Affected DB Entities:** `clinical.appointments` (status updated to `CONFIRMED`), `clinical.opd_visits` (new record created), `billing.payments` (logs transaction detail).

### Request
* **Path Parameters:**
  * `id` (UUID, Required): Appointment ID from Step 2.
* **Body Schema:**
  * `payment_status` (String, Required): Must be `PAID`.
  * `payment_mode` (String, Required): Valid values: `CASH`, `CARD`, `UPI`, `BANK_TRANSFER`.
  * `amount_paid` (Number, Required): Total consultation fee paid (must be > 0).
  * `payment_ref` (String, Optional): Txn ID or receipt reference.

#### Request Example
```json
{
  "payment_status": "PAID",
  "payment_mode": "UPI",
  "amount_paid": 500.00,
  "payment_ref": "UPI-TXN-9988771122"
}
```

### Response
* **Success (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "message": "Appointment confirmed and visit generated",
  "data": {
    "appointment_id": "a9a9b8b8-c7c7-6d6d-5e5e-4f4f3f3f2e2e",
    "opd_visit_id": "f5f5e4e4-d3d3-2c2c-1b1b-0a0a9a9a8a8a",
    "status": "CONFIRMED"
  }
}
```
* **Error Responses:**
  * **404 Not Found:** Appointment ID not found.
  * **409 Conflict:** Appointment is already confirmed or cancelled.

---

### ➔ Transition: Step 3 to Step 4
* **Why Next API is Required:** To log that the doctor has admitted the patient into the consultation chamber and started writing clinical findings.
* **IDs Carried Forward:** `opd_visit_id` (`data.opd_visit_id` from Step 3).
* **Workflow State Change:** `WAITING`
* **Required Permissions:** `consultations:create`
* **Validations:** The visit status must be `ARRIVED`.

---

## Step 4: Start OPD Consultation
* **Purpose:** Doctor starts the consultation session. This opens the clinical EMR workspace for the doctor.
* **HTTP Method:** `POST`
* **Endpoint:** `/opd/visits/{id}/start`
* **Required Permission:** `consultations:create`
* **Preconditions:** Patient check-in payment completed.
* **Postconditions:** EMR status updates to `IN_PROGRESS` and starts consultation timer.
* **Affected DB Entities:** `clinical.opd_visits` (status updated to `IN_PROGRESS`, `consultation_started_at` set to current timestamp).

### Request
* **Path Parameters:**
  * `id` (UUID, Required): OPD visit ID from Step 3.
* **Body Schema:** None (Empty JSON `{}`).

#### Request Example
```json
{}
```

### Response
* **Success (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "f5f5e4e4-d3d3-2c2c-1b1b-0a0a9a9a8a8a",
    "patient_id": "e2c07a01-2092-4876-8889-112233445566",
    "doctor_id": "97d2a10d-1b8a-40b4-a37a-5daf4e35f630",
    "status": "IN_PROGRESS",
    "consultation_started_at": "2026-07-11T10:15:00Z"
  }
}
```
* **Error Responses:**
  * **400 Bad Request:** Visit cannot be started (e.g., wrong status, already started/completed).
  * **404 Not Found:** Visit ID does not exist.

---

### ➔ Transition: Step 4 to Step 5
* **Why Next API is Required:** During the consultation, the doctor determines that the patient requires surgery. The doctor creates a formal clinical surgery advice order.
* **IDs Carried Forward:** `patient_id` and `opd_visit_id` (`data.id` from Step 4).
* **Workflow State Change:** `IN_CONSULTATION`
* **Required Permissions:** `surgery:create`
* **Validations:** Consultation must be currently active (`IN_PROGRESS`).

---

## Step 5: Create Surgery / Procedure Order
* **Purpose:** Doctor documents the clinical decision to perform a surgery. This acts as the official medical advice.
* **HTTP Method:** `POST`
* **Endpoint:** `/opd/surgery`
* **Required Permission:** `surgery:create`
* **Preconditions:** Clinical evaluation performed during consultation.
* **Postconditions:** A record in `service_orders` is created with status `ORDERED`.
* **Affected DB Entities:** `clinical.service_orders` (inserts surgery order details).

### Request
* **Body Schema:**
  * `patient_id` (UUID, Required): Patient ID from Step 1.
  * `opd_visit_id` (UUID, Required): Active OPD Visit ID from Step 4.
  * `order_type` (String, Required): Must be `SURGERY` or `PROCEDURE`.
  * `service_name` (String, Required): Clinical name of the surgery (e.g., "Laparoscopic Appendectomy"). Length: 2 to 300 chars.
  * `service_code` (String, Optional): Internal catalog billing code (e.g., "SURG-104").
  * `assigned_doctor_id` (UUID, Required): Operating Surgeon.
  * `priority` (String, Required): Valid values: `ROUTINE`, `URGENT`, `EMERGENCY`.
  * `notes` (String, Optional): Diagnostic indicator comments (max 2000 chars).
  * `pre_op_instructions` (String, Optional): Diet/medical preparation notes (max 2000 chars).
  * `estimated_cost` (Number, Optional): Cost quote for financial counselling (must be >= 0).

#### Request Example
```json
{
  "patient_id": "e2c07a01-2092-4876-8889-112233445566",
  "opd_visit_id": "f5f5e4e4-d3d3-2c2c-1b1b-0a0a9a9a8a8a",
  "order_type": "SURGERY",
  "service_name": "Laparoscopic Appendectomy",
  "service_code": "SURG-104",
  "assigned_doctor_id": "97d2a10d-1b8a-40b4-a37a-5daf4e35f630",
  "priority": "URGENT",
  "notes": "USG reveals inflamed appendiceal wall. Indicated for elective surgery.",
  "pre_op_instructions": "Fasting from 12 midnight, start IV fluids upon admission",
  "estimated_cost": 85000.00
}
```

### Response
* **Success (201 Created):**
```json
{
  "success": true,
  "code": 201,
  "message": "Surgery order created successfully",
  "data": {
    "id": "c3d4e5f6-g7h8-9012-cdef-123456789012",
    "patient_id": "e2c07a01-2092-4876-8889-112233445566",
    "opd_visit_id": "f5f5e4e4-d3d3-2c2c-1b1b-0a0a9a9a8a8a",
    "order_type": "SURGERY",
    "service_name": "Laparoscopic Appendectomy",
    "assigned_doctor_id": "97d2a10d-1b8a-40b4-a37a-5daf4e35f630",
    "status": "ORDERED",
    "notes": "USG reveals inflamed appendiceal wall...",
    "estimated_cost": 85000.00,
    "created_at": "2026-07-11T10:20:00Z"
  }
}
```
* **Error Responses:**
  * **422 Unprocessable Entity:** Surgeon UUID invalid or wrong order type code.

---

### ➔ Transition: Step 5 to Step 6
* **Why Next API is Required:** To alert the IPD admission desk that this patient requires inpatient bed booking for their recommended surgery.
* **IDs Carried Forward:** `patient_id` and `service_order_id` (`data.id` from Step 5) as the `source_reference`.
* **Workflow State Change:** `SURGERY_ORDERED`
* **Required Permissions:** `admissions:create`
* **Validations:** Verify that the service order has status `ORDERED` and belongs to the patient.

---

## Step 6: Create Admission Request
* **Purpose:** Initiate the hospital admission workflow. This triggers bed planning and insurance approval verification.
* **HTTP Method:** `POST`
* **Endpoint:** `/ipd/admission-requests`
* **Required Permission:** `admissions:create`
* **Preconditions:** Clinical surgery advice must have been recorded.
* **Postconditions:** Admission request logged with status `REQUESTED`.
* **Affected DB Entities:** `ipd.admission_requests` (inserts pre-admission record).

### Request
* **Body Schema:**
  * `patient_id` (UUID, Required): Patient ID from Step 1.
  * `source_type` (String, Required): Must be `OPD` for this scenario.
  * `source_reference` (UUID, Required): OPD Surgery Service Order ID from Step 5.
  * `request_reason` (String, Required): Diagnostic justification (min 5, max 2000 chars).
  * `admission_type` (String, Optional): Valid values: `ELECTIVE`, `EMERGENCY`, `TRANSFER` (default: `ELECTIVE`).
  * `priority` (String, Optional): Valid values: `ROUTINE`, `URGENT`, `EMERGENCY` (default: `ROUTINE`).
  * `preferred_ward_id` (UUID, Optional): Target ward type selection.
  * `preferred_bed_type` (String, Optional): Preferred bed category.
  * `notes` (String, Optional): Special requirements notes.

#### Request Example
```json
{
  "patient_id": "e2c07a01-2092-4876-8889-112233445566",
  "source_type": "OPD",
  "source_reference": "c3d4e5f6-g7h8-9012-cdef-123456789012",
  "request_reason": "Pre-scheduled Laparoscopic Appendectomy for chronic appendicitis",
  "admission_type": "ELECTIVE",
  "priority": "ROUTINE",
  "preferred_ward_id": "4c5d12d2-79dd-4650-867e-079ac3b85a5a",
  "preferred_bed_type": "SEMI_PRIVATE"
}
```

### Response
* **Success (201 Created):**
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "7848855e-d0fc-4710-bc94-76afc790cdc7",
    "patient_id": "e2c07a01-2092-4876-8889-112233445566",
    "source_type": "OPD",
    "source_reference": "c3d4e5f6-g7h8-9012-cdef-123456789012",
    "request_reason": "Pre-scheduled Laparoscopic Appendectomy...",
    "admission_type": "ELECTIVE",
    "status": "REQUESTED",
    "created_at": "2026-07-11T10:30:00Z"
  }
}
```
* **Error Responses:**
  * **409 Conflict:** Patient already has an active pending admission request.

---

### ➔ Transition: Step 6 to Step 7
* **Why Next API is Required:** The admissions desk reviews the request, verifies insurance clearance, coordinates bed assignment availability, and approves the request.
* **IDs Carried Forward:** `request_id` (`data.id` from Step 6).
* **Workflow State Change:** `ADMISSION_REQUESTED`
* **Required Permissions:** `admissions:edit`
* **Validations:** Request status must be `REQUESTED` or `UNDER_REVIEW`.

---

## Step 7: Approve Admission Request
* **Purpose:** Transition the request status to `READY`, signaling that administrative clearance and a target ward/bed have been secured.
* **HTTP Method:** `PATCH`
* **Endpoint:** `/ipd/admission-requests/{request_id}/status`
* **Required Permission:** `admissions:edit`
* **Preconditions:** Request must be active.
* **Postconditions:** Status updates to `READY`.
* **Affected DB Entities:** `ipd.admission_requests` (status set to `READY`, `notes` appended).

### Request
* **Path Parameters:**
  * `request_id` (UUID, Required): Admission Request ID from Step 6.
* **Body Schema:**
  * `status` (String, Required): Must be `READY`.
  * `reason` (String, Optional): Approval and bed check details.

#### Request Example
```json
{
  "status": "READY",
  "reason": "Semi-private bed reserved, insurance pre-auth approved for ₹85000"
}
```

### Response
* **Success (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "7848855e-d0fc-4710-bc94-76afc790cdc7",
    "status": "READY",
    "notes": "Semi-private bed reserved, insurance pre-auth approved for ₹85000"
  }
}
```

---

### ➔ Transition: Step 7 to Step 8
* **Why Next API is Required:** To formally admit the patient into the hospital and register them to an active ward bed, making them a formal inpatient.
* **IDs Carried Forward:** `patient_id`, `request_id` (as `admission_request_id`), and the selected `ward_id` and `bed_id` identified by the admissions desk.
* **Workflow State Change:** `ADMISSION_APPROVED`
* **Required Permissions:** `admissions:create`
* **Validations:** Target bed must be currently `AVAILABLE` in the master bed registry.

---

## Step 8: Create IPD Admission (Check-In)
* **Purpose:** Formally admit the patient into the hospital, allocate the bed resource, and generate the IPD admission number.
* **HTTP Method:** `POST`
* **Endpoint:** `/ipd/admissions`
* **Required Permission:** `admissions:create`
* **Preconditions:** Bed must be available and admission request must be `READY`.
* **Postconditions:** Patient is registered as an active inpatient. The allocated bed is marked `OCCUPIED`. The source admission request status is set to `CONVERTED`.
* **Affected DB Entities:** `ipd.ipd_admissions` (inserts active admission record), `ipd.beds` (status set to `OCCUPIED`), `ipd.admission_requests` (status updated to `CONVERTED`).

### Request
* **Body Schema:**
  * `patient_id` (UUID, Required): Patient ID from Step 1.
  * `admission_request_id` (UUID, Required): Request ID from Step 6.
  * `ward_id` (UUID, Required): Target inpatient ward ID.
  * `bed_id` (UUID, Required): Target inpatient bed ID. Must be currently `AVAILABLE`.
  * `attending_doctor_id` (UUID, Required): Attending Doctor ID.
  * `admission_type` (String, Required): Valid values: `ELECTIVE`, `EMERGENCY`, `TRANSFER`.
  * `admission_reason` (String, Required): Clinical description.
  * `expected_discharge` (String - Date, Optional): Estimated discharge date (YYYY-MM-DD).
  * `notes` (String, Optional): Nursing or dietary instructions.

#### Request Example
```json
{
  "patient_id": "e2c07a01-2092-4876-8889-112233445566",
  "admission_request_id": "7848855e-d0fc-4710-bc94-76afc790cdc7",
  "ward_id": "4c5d12d2-79dd-4650-867e-079ac3b85a5a",
  "bed_id": "84820843-9876-4321-9abc-1234567890ab",
  "attending_doctor_id": "97d2a10d-1b8a-40b4-a37a-5daf4e35f630",
  "admission_type": "ELECTIVE",
  "admission_reason": "Laparoscopic Appendectomy",
  "expected_discharge": "2026-07-14"
}
```

### Response
* **Success (201 Created):**
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "e5f6g7h8-i9j0-1234-efgh-345678901234",
    "admission_number": "ADM-2026-0091",
    "patient_id": "e2c07a01-2092-4876-8889-112233445566",
    "status": "ADMITTED",
    "ipd_status": "ADMITTED",
    "ward_id": "4c5d12d2-79dd-4650-867e-079ac3b85a5a",
    "bed_id": "84820843-9876-4321-9abc-1234567890ab",
    "admitted_at": "2026-07-11T11:00:00Z"
  }
}
```
* **Error Responses:**
  * **409 Conflict:** The chosen bed is no longer available (status: `OCCUPIED`).

---

### ➔ Transition: Step 8 to Step 9
* **Why Next API is Required:** To create the Operating Theatre (OT) session, assign the surgical team, reserve the OT room slot, and link the session back to the clinical surgery order.
* **IDs Carried Forward:** `patient_id`, `admission_id` (`data.id` from Step 8), and `service_order_id` (from Step 5).
* **Workflow State Change:** `ADMITTED` / `BED_ASSIGNED`
* **Required Permissions:** `ot:schedule`
* **Validations:** Patient must currently be admitted (`ipd_status` = `ADMITTED` or `UNDER_TREATMENT`).

---

## Step 9: Create OT Session (Schedule Surgery)
* **Purpose:** Reserve an OT room, schedule the operation slot, and register the surgery team (surgeon, anaesthetist, scrub nurse).
* **HTTP Method:** `POST`
* **Endpoint:** `/ipd/ot-sessions`
* **Required Permission:** `ot:schedule`
* **Preconditions:** Patient must be admitted and have a valid surgery service order.
* **Postconditions:** OT session is initialized with status `SCHEDULED`. The linked service order status is updated to `SCHEDULED`.
* **Affected DB Entities:** `ipd.ot_sessions` (inserts session schedule), `clinical.service_orders` (updates status to `SCHEDULED`).

### Request
* **Body Schema:**
  * `service_order_id` (UUID, Required): Clinical Surgery Order ID from Step 5.
  * `patient_id` (UUID, Required): Patient ID from Step 1.
  * `admission_id` (UUID, Required): IPD Admission ID from Step 8.
  * `surgeon_id` (UUID, Required): Primary surgeon user ID.
  * `anaesthetist_id` (UUID, Required): Attending anaesthetist user ID.
  * `scrub_nurse_id` (UUID, Optional): Scrub nurse user ID.
  * `ot_room_id` (String, Required): Room identifier code (e.g., "OT-ROOM-1").
  * `scheduled_start` (String - ISO 8601, Required): Format: `YYYY-MM-DDTHH:MM:SSZ`.
  * `scheduled_end` (String - ISO 8601, Required): Format: `YYYY-MM-DDTHH:MM:SSZ`.

#### Request Example
```json
{
  "service_order_id": "c3d4e5f6-g7h8-9012-cdef-123456789012",
  "patient_id": "e2c07a01-2092-4876-8889-112233445566",
  "admission_id": "e5f6g7h8-i9j0-1234-efgh-345678901234",
  "surgeon_id": "97d2a10d-1b8a-40b4-a37a-5daf4e35f630",
  "anaesthetist_id": "87d2a10d-1b8a-40b4-a37a-5daf4e35f690",
  "scrub_nurse_id": "77d2a10d-1b8a-40b4-a37a-5daf4e35f680",
  "ot_room_id": "OT-ROOM-1",
  "scheduled_start": "2026-07-12T08:00:00Z",
  "scheduled_end": "2026-07-12T10:00:00Z"
}
```

### Response
* **Success (201 Created):**
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "g7h8i9j0-k1l2-3456-ghij-567890123456",
    "service_order_id": "c3d4e5f6-g7h8-9012-cdef-123456789012",
    "admission_id": "e5f6g7h8-i9j0-1234-efgh-345678901234",
    "status": "SCHEDULED",
    "pac_cleared": false,
    "created_at": "2026-07-11T12:00:00Z"
  }
}
```

---

### ➔ Transition: Step 9 to Step 10
* **Why Next API is Required:** Before any surgical procedure begins, mandatory legal consents must be registered in the patient's record.
* **IDs Carried Forward:** `ot_session_id` (`data.id` from Step 9, internally used as `ot_case_id`).
* **Workflow State Change:** `OT_SCHEDULED`
* **Required Permissions:** `consent:create`
* **Validations:** OT session status must be `SCHEDULED`.

---

## Step 10: Create OT Case Consents
* **Purpose:** Generate pending consent templates (`SURGICAL`, `ANAESTHESIA`, `PROCEDURE`) for the patient to sign.
* **HTTP Method:** `POST`
* **Endpoint:** `/ipd/ot-cases/{ot_case_id}/consents`
* **Required Permission:** `consent:create`
* **Preconditions:** OT session must be scheduled.
* **Postconditions:** Unsigned consent records are created in `clinical.consents` with status `PENDING`.
* **Affected DB Entities:** `clinical.consents` (inserts multiple pending consent rows).

### Request
* **Path Parameters:**
  * `ot_case_id` (UUID, Required): OT session ID from Step 9.
* **Body Schema:**
  * `consent_codes` (Array of Strings, Required): Valid codes: `SURGICAL`, `ANAESTHESIA`, `PROCEDURE`, `BLOOD_TRANSFUSION`, `HIGH_RISK`, `IMPLANT_DEVICE`, `RESEARCH`, `PHOTOGRAPHY_VIDEO`, `BLOOD_TISSUE_SAMPLE`, `HIV_INFECTIOUS`, `PHYSICIAN_FITNESS`.

#### Request Example
```json
{
  "consent_codes": [
    "SURGICAL",
    "ANAESTHESIA",
    "PROCEDURE"
  ]
}
```

### Response
* **Success (201 Created):**
```json
{
  "success": true,
  "code": 201,
  "data": [
    {
      "id": "con-surg-0001-abcd-efgh-ijklmnopqrst",
      "ot_case_id": "g7h8i9j0-k1l2-3456-ghij-567890123456",
      "consent_code": "SURGICAL",
      "mandatory": true,
      "status": "PENDING"
    },
    {
      "id": "con-anes-0002-abcd-efgh-ijklmnopqrst",
      "ot_case_id": "g7h8i9j0-k1l2-3456-ghij-567890123456",
      "consent_code": "ANAESTHESIA",
      "mandatory": true,
      "status": "PENDING"
    },
    {
      "id": "con-proc-0003-abcd-efgh-ijklmnopqrst",
      "ot_case_id": "g7h8i9j0-k1l2-3456-ghij-567890123456",
      "consent_code": "PROCEDURE",
      "mandatory": true,
      "status": "PENDING"
    }
  ]
}
```

---

### ➔ Transition: Step 10 to Step 11
* **Why Next API is Required:** To transition each pending consent to `SIGNED` status by collecting the signature of the patient or their legal guardian.
* **IDs Carried Forward:** `consent_id` (each individual UUID in `data[].id` from Step 10).
* **Workflow State Change:** None.
* **Required Permissions:** `consent:sign`
* **Validations:** The consent record must be in `PENDING` status.

---

## Step 11: Sign Consent
* **Purpose:** Record the digital signature or acknowledgement of the consent forms.
* **HTTP Method:** `POST`
* **Endpoint:** `/ipd/consents/{consent_id}/sign`
* **Required Permission:** `consent:sign`
* **Preconditions:** Consent record must exist.
* **Postconditions:** Consent status changes from `PENDING` to `SIGNED`. A tamper-proof SHA-256 hash is generated.
* **Affected DB Entities:** `clinical.consents` (status updated to `SIGNED`, records timestamp and signee), `clinical.consent_audit_log` (logs signature audit trail).

### Request
* **Path Parameters:**
  * `consent_id` (UUID, Required): Target consent ID.
* **Body Schema:**
  * `signature_type` (String, Required): Must be `DIGITAL`.
  * `signed_by` (UUID, Required): User ID of the patient (or guardian).
  * `doctor_id` (UUID, Optional): Witnessing doctor.
  * `witness_id` (UUID, Optional): Witnessing nurse/staff.
  * `remarks` (String, Optional): Notes.

#### Request Example
```json
{
  "signature_type": "DIGITAL",
  "signed_by": "e2c07a01-2092-4876-8889-112233445566",
  "doctor_id": "97d2a10d-1b8a-40b4-a37a-5daf4e35f630",
  "witness_id": "77d2a10d-1b8a-40b4-a37a-5daf4e35f680",
  "remarks": "Patient explained risks and consented."
}
```

### Response
* **Success (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "con-surg-0001-abcd-efgh-ijklmnopqrst",
    "ot_case_id": "g7h8i9j0-k1l2-3456-ghij-567890123456",
    "consent_code": "SURGICAL",
    "status": "SIGNED",
    "signed_at": "2026-07-11T14:00:00Z"
  }
}
```
* **Error Responses:**
  * **409 Conflict:** Consent is already signed.

*(Note: Repeat Step 11 for all mandatory consents: `SURGICAL`, `ANAESTHESIA`, and `PROCEDURE`).*

---

### ➔ Transition: Step 11 to Step 12
* **Why Next API is Required:** To record that the pre-anaesthetic evaluation has been cleared by the anaesthetist. This is a critical clinical gate.
* **IDs Carried Forward:** `ot_session_id` (`ot_case_id` from Step 10).
* **Workflow State Change:** None.
* **Required Permissions:** `ot:pac`
* **Validations:** All mandatory consents (`SURGICAL`, `ANAESTHESIA`, `PROCEDURE`) must be in `SIGNED` status in the DB.

---

## Step 12: Clear Pre-Anaesthetic Checkup (PAC)
* **Purpose:** Log the official PAC clearance by the anaesthetist.
* **HTTP Method:** `POST`
* **Endpoint:** `/ipd/ot-sessions/{id}/pac`
* **Required Permission:** `ot:pac`
* **Preconditions:** All mandatory consents must be signed.
* **Postconditions:** The field `pac_cleared` is set to `true` on the OT session.
* **Affected DB Entities:** `ipd.ot_sessions` (sets `pac_cleared = true`, `pac_cleared_by`, `pac_cleared_at`).

### Request
* **Path Parameters:**
  * `id` (UUID, Required): OT session ID from Step 9.
* **Body Schema:**
  * `notes` (String, Optional): Clearance details (e.g., "ASA class II, airway normal").
  * `emergency_override` (Boolean, Optional): Default is `false`.

#### Request Example
```json
{
  "notes": "Patient cleared for surgery. NPO status confirmed since midnight."
}
```

### Response
* **Success (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "g7h8i9j0-k1l2-3456-ghij-567890123456",
    "status": "SCHEDULED",
    "pac_cleared": true,
    "pac_cleared_by": "87d2a10d-1b8a-40b4-a37a-5daf4e35f690",
    "pac_cleared_at": "2026-07-12T07:00:00Z"
  }
}
```
* **Error Responses:**
  * **409 Conflict:** Blocked because mandatory consents are still pending signature.

---

### ➔ Transition: Step 12 to Step 13
* **Why Next API is Required:** To complete the preoperative nursing checklist (e.g., vital signs, site marking, fasting verification) before entering the OR.
* **IDs Carried Forward:** `ot_session_id` (`id` from Step 12).
* **Workflow State Change:** None.
* **Required Permissions:** `ot:pre-op`
* **Validations:** `pac_cleared` must be `true` on the OT session.

---

## Step 13: Complete Pre-Op Checklist
* **Purpose:** Log preoperative checklist completion by the floor nurse.
* **HTTP Method:** `POST`
* **Endpoint:** `/ipd/ot-sessions/{id}/pre-op`
* **Required Permission:** `ot:pre-op`
* **Preconditions:** PAC must be cleared.
* **Postconditions:** OT session status advances to `PRE_OP`.
* **Affected DB Entities:** `ipd.ot_sessions` (updates status to `PRE_OP`, stores checklist JSON).

### Request
* **Path Parameters:**
  * `id` (UUID, Required): OT session ID.
* **Body Schema:**
  * `pre_op_checklist` (Object, Required): JSON checklist structure.

#### Request Example
```json
{
  "pre_op_checklist": {
    "patient_identity_verified": true,
    "consent_verified": true,
    "surgical_site_marked": true,
    "npo_status_verified": true,
    "allergies_reviewed": true,
    "vitals_stable": true
  }
}
```

### Response
* **Success (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "g7h8i9j0-k1l2-3456-ghij-567890123456",
    "status": "PRE_OP",
    "pre_op_completed_at": "2026-07-12T07:45:00Z"
  }
}
```

---

### ➔ Transition: Step 13 to Step 14
* **Why Next API is Required:** To record the start of the incision and shift the patient status inside the Operating Room.
* **IDs Carried Forward:** `ot_session_id` (`id` from Step 13).
* **Workflow State Change:** `PRE_OP`
* **Required Permissions:** `ot:start`
* **Validations:** Session status must be `PRE_OP`.

---

## Step 14: Start OT Session (Surgery Begins)
* **Purpose:** Record the start timestamp of the surgical procedure.
* **HTTP Method:** `POST`
* **Endpoint:** `/ipd/ot-sessions/{id}/start`
* **Required Permission:** `ot:start`
* **Preconditions:** Pre-Op checklist must be completed.
* **Postconditions:** Session status changes to `IN_PROGRESS`. The linked service order status changes to `IN_PROGRESS`.
* **Affected DB Entities:** `ipd.ot_sessions` (status set to `IN_PROGRESS`, `surgery_start_at` set), `clinical.service_orders` (status set to `IN_PROGRESS`).

### Request
* **Path Parameters:**
  * `id` (UUID, Required): OT session ID.
* **Body Schema:** None (Empty JSON `{}`).

#### Request Example
```json
{}
```

### Response
* **Success (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "g7h8i9j0-k1l2-3456-ghij-567890123456",
    "status": "IN_PROGRESS",
    "surgery_start_at": "2026-07-12T08:05:00Z"
  }
}
```

---

### ➔ Transition: Step 14 to Step 15
* **Why Next API is Required:** To log materials, medications, and surgical implants used during the procedure for clinical tracking and inventory billing.
* **IDs Carried Forward:** `ot_session_id` (`id` from Step 14).
* **Workflow State Change:** `IN_OT`
* **Required Permissions:** `ot:consumables`
* **Validations:** OT session status must be active (`IN_PROGRESS`).

---

## Step 15: Log OT Consumables
* **Purpose:** Charge consumables, sutures, implants, and drugs directly to the patient's OT case.
* **HTTP Method:** `POST`
* **Endpoint:** `/ipd/ot-sessions/{id}/consumables`
* **Required Permission:** `ot:consumables`
* **Preconditions:** Surgery must be currently in progress.
* **Postconditions:** Charges are added to the billing register, and inventory stock level is decremented on the backend.
* **Affected DB Entities:** `ipd.ot_consumables` (inserts item consumption log), `billing.charge_items` (inserts chargeable items linked to the admission).

### Request
* **Path Parameters:**
  * `id` (UUID, Required): OT session ID.
* **Body Schema:**
  * `item_name` (String, Required): Name of the inventory item.
  * `quantity` (Number, Required): Quantity used (must be > 0).
  * `batch_no` (String, Optional): Material batch tracking code.
  * `implant_serial` (String, Optional): Implant serial number.

#### Request Example
```json
{
  "item_name": "Prolene Suture Thread 3-0",
  "quantity": 2.0,
  "batch_no": "B-PR-88122",
  "implant_serial": "SN-MOCK-998811"
}
```

### Response
* **Success (201 Created):**
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "consumable-uuid-771122",
    "ot_session_id": "g7h8i9j0-k1l2-3456-ghij-567890123456",
    "item_name": "Prolene Suture Thread 3-0",
    "quantity": 2.0,
    "batch_no": "B-PR-88122"
  }
}
```

---

### ➔ Transition: Step 15 to Step 16
* **Why Next API is Required:** To record the end of the procedure and capture intraoperative physician notes.
* **IDs Carried Forward:** `ot_session_id` (`id` from Step 15).
* **Workflow State Change:** None.
* **Required Permissions:** `ot:complete`
* **Validations:** Session status must be `IN_PROGRESS`.

---

## Step 16: Complete OT Session
* **Purpose:** Log the end of the surgery and start the recovery room phase.
* **HTTP Method:** `POST`
* **Endpoint:** `/ipd/ot-sessions/{id}/complete`
* **Required Permission:** `ot:complete`
* **Preconditions:** Incision closure completed.
* **Postconditions:** OT session status changes to `POST_OP`.
* **Affected DB Entities:** `ipd.ot_sessions` (status set to `POST_OP`, `surgery_end_at` set, `intra_op_notes` and `anaesthesia_type` recorded).

### Request
* **Path Parameters:**
  * `id` (UUID, Required): OT session ID.
* **Body Schema:**
  * `anaesthesia_type` (String, Required): Valid values: `GENERAL`, `SPINAL`, `EPIDURAL`, `LOCAL`, `REGIONAL`, `SEDATION`.
  * `intra_op_notes` (String, Optional): Surgical findings (max 4000 chars).

#### Request Example
```json
{
  "anaesthesia_type": "GENERAL",
  "intra_op_notes": "Appendix excised. Minimal blood loss. Suture closed in layers. Specimen sent to histopathology."
}
```

### Response
* **Success (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "g7h8i9j0-k1l2-3456-ghij-567890123456",
    "status": "POST_OP",
    "surgery_end_at": "2026-07-12T09:40:00Z"
  }
}
```

---

### ➔ Transition: Step 16 to Step 17
* **Why Next API is Required:** Once the patient is stabilized in recovery, they must be transferred back to a ward bed. The OT room must be cleared.
* **IDs Carried Forward:** `ot_session_id` (`id` from Step 16), and the target ward/bed details.
* **Workflow State Change:** `SURGERY_COMPLETED` / `RECOVERY`
* **Required Permissions:** `ot:transfer-out`
* **Validations:** The target ward bed must be `AVAILABLE`.

---

## Step 17: Transfer Out from OT (Recovery → Ward Bed)
* **Purpose:** Formally release the OT room resource, complete the OT lifecycle, and transfer the patient back to an inpatient ward bed.
* **HTTP Method:** `POST`
* **Endpoint:** `/ipd/ot-sessions/{id}/transfer-out`
* **Required Permission:** `ot:transfer-out`
* **Preconditions:** Recovery phase complete; patient stable.
* **Postconditions:**
  1. The OT bed/room is released.
  2. The target inpatient bed is marked `OCCUPIED`.
  3. The admission's active `ward_id` and `bed_id` are updated.
  4. The clinical service order status is updated to `COMPLETED`.
  5. The OT session status is set to `COMPLETED`.
  6. The admission's `ipd_status` changes to `UNDER_TREATMENT`.
* **Affected DB Entities:** `ipd.ot_sessions` (status set to `COMPLETED`), `clinical.service_orders` (status set to `COMPLETED`), `ipd.ipd_admissions` (updates `ward_id`, `bed_id`, `ipd_status = 'UNDER_TREATMENT'`), `ipd.beds` (releases old recovery bed to `AVAILABLE`; sets new ward bed status to `OCCUPIED`).

### Request
* **Path Parameters:**
  * `id` (UUID, Required): OT session ID.
* **Body Schema:**
  * `transferred_to_ward_id` (UUID, Required): Target inpatient ward ID.
  * `transferred_to_bed_id` (UUID, Required): Target inpatient bed ID. Must be `AVAILABLE`.
  * `recovery_notes` (String, Optional): Clinical transfer observations.

#### Request Example
```json
{
  "transferred_to_ward_id": "4c5d12d2-79dd-4650-867e-079ac3b85a5a",
  "transferred_to_bed_id": "84820843-9876-4321-9abc-1234567890ab",
  "recovery_notes": "Patient fully awake, vitals normal. Pain controlled."
}
```

### Response
* **Success (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "g7h8i9j0-k1l2-3456-ghij-567890123456",
    "status": "COMPLETED",
    "transfer_at": "2026-07-12T11:00:00Z"
  }
}
```

---

### ➔ Transition: Step 17 to Step 18
* **Why Next API is Required:** To compile and generate the financial invoice for the inpatient stay, aggregating charges for bed stay, surgery, consumables, and professional fees.
* **IDs Carried Forward:** `patient_id` and `admission_id` (as `visit_id`).
* **Workflow State Change:** `POST_OP`
* **Required Permissions:** `billing:create`
* **Validations:** Patient must be under inpatient treatment (`ipd_status` = `UNDER_TREATMENT`).

---

## Step 18: Generate Invoice
* **Purpose:** Retrieve all billable charges logged for the admission and compile them into a unified checkout invoice.
* **HTTP Method:** `POST`
* **Endpoint:** `/billing/invoices`
* **Required Permission:** `billing:create`
* **Preconditions:** Inpatient treatment must be active.
* **Postconditions:** An invoice is created with status `OPEN` and outstanding amount calculated.
* **Affected DB Entities:** `billing.invoices` (creates invoice header record), `billing.invoice_items` (aggregates charges from `billing.charge_items`).

### Request
* **Body Schema:**
  * `patient_id` (UUID, Required): Patient ID from Step 1.
  * `visit_type` (String, Required): Must be `IPD`.
  * `visit_id` (UUID, Required): IPD Admission ID from Step 8.

#### Request Example
```json
{
  "patient_id": "e2c07a01-2092-4876-8889-112233445566",
  "visit_type": "IPD",
  "visit_id": "e5f6g7h8-i9j0-1234-efgh-345678901234"
}
```

### Response
* **Success (201 Created):**
```json
{
  "success": true,
  "code": 201,
  "data": {
    "invoice_id": "inv-0001-abcd-efgh-ijkl-567890123456",
    "invoice_number": "INV-2026-00911",
    "status": "OPEN",
    "totals": {
      "total_amount": 95000.00,
      "paid_amount": 0.00,
      "outstanding": 95000.00
    }
  }
}
```

---

### ➔ Transition: Step 18 to Step 19
* **Why Next API is Required:** To process the payment from the patient or insurance provider to clear the outstanding balance.
* **IDs Carried Forward:** `invoice_id` (`data.invoice_id` from Step 18).
* **Workflow State Change:** None.
* **Required Permissions:** `billing:payment:collect`
* **Validations:** Invoice status must be `OPEN` or `PARTIALLY_PAID`.

---

## Step 19: Collect Payment
* **Purpose:** Log a transaction receipt (cash, card, UPI, or insurance guarantee).
* **HTTP Method:** `POST`
* **Endpoint:** `/billing/collect`
* **Required Permission:** `billing:payment:collect`
* **Preconditions:** Invoice must exist.
* **Postconditions:** If the outstanding amount reaches `0.00`, the invoice status transitions to `PAID`.
* **Affected DB Entities:** `billing.payments` (inserts payment transaction), `billing.invoices` (updates status and `outstanding` amount).

### Request
* **Body Schema:**
  * `invoice_id` (UUID, Required): Target Invoice ID from Step 18.
  * `amount` (Number, Required): Amount paid (must be > 0).
  * `payment_mode` (String, Required): Valid values: `CASH`, `CARD`, `UPI`, `BANK_TRANSFER`, `CHEQUE`, `INSURANCE`.
  * `reference_no` (String, Optional): Transaction reference ID.

#### Request Example
```json
{
  "invoice_id": "inv-0001-abcd-efgh-ijkl-567890123456",
  "amount": 95000.00,
  "payment_mode": "CARD",
  "reference_no": "TXN-20260712-887711"
}
```

### Response
* **Success (201 Created):**
```json
{
  "success": true,
  "code": 201,
  "data": {
    "invoice_id": "inv-0001-abcd-efgh-ijkl-567890123456",
    "transaction_id": "txn-0001-abcd-efgh-ijkl-567890123456",
    "invoice_status": "PAID",
    "outstanding": 0.00,
    "paid_at": "2026-07-12T14:30:00Z"
  }
}
```

---

### ➔ Transition: Step 19 to Step 20
* **Why Next API is Required:** To request a formal No Objection Certificate (NOC) from the billing department before the patient can leave the ward.
* **IDs Carried Forward:** `admission_id` (`visit_id` from Step 18).
* **Workflow State Change:** None.
* **Required Permissions:** `billing:clearance:view`
* **Validations:** Verified against active invoice table records.

---

## Step 20: Get Billing Clearance (NOC Verification)
* **Purpose:** Verify if the patient has any outstanding billing balance.
* **HTTP Method:** `GET`
* **Endpoint:** `/billing/clearance/{admission_id}`
* **Required Permission:** `billing:clearance:view`
* **Preconditions:** Invoice checkout must have been run.
* **Postconditions:** Returns `cleared: true` if outstanding is `0.00`.
* **Affected DB Entities:** None (Read-only query).

### Request
* **Path Parameters:**
  * `admission_id` (UUID, Required): IPD Admission ID.

#### Request Example
None (Query is a GET request).

### Response
* **Success - Cleared (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "admission_id": "e5f6g7h8-i9j0-1234-efgh-345678901234",
    "cleared": true,
    "noc_message": "Billing clearance confirmed. No outstanding balance."
  }
}
```

---

### ➔ Transition: Step 20 to Step 21
* **Why Next API is Required:** The attending physician initiates the clinical discharge sequence, writing the summary and medications.
* **IDs Carried Forward:** `admission_id` (`admission_id` from Step 20).
* **Workflow State Change:** None.
* **Required Permissions:** `admissions:edit`
* **Validations:** Inpatient clinical status must be `UNDER_TREATMENT`.

---

## Step 21: Initiate Discharge
* **Purpose:** Attending doctor signs the clinical discharge plan and publishes the post-op follow-up protocol.
* **HTTP Method:** `POST`
* **Endpoint:** `/ipd/admissions/{id}/discharge/initiate`
* **Required Permission:** `admissions:edit`
* **Preconditions:** Physician must authorize checkout.
* **Postconditions:** Inpatient status advances to `READY_FOR_DISCHARGE`.
* **Affected DB Entities:** `ipd.ipd_admissions` (updates `ipd_status` to `READY_FOR_DISCHARGE`, stores discharge summary).

### Request
* **Path Parameters:**
  * `id` (UUID, Required): IPD Admission ID.
* **Body Schema:**
  * `discharge_summary` (String, Required): Clinical summary of hospitalization.
  * `follow_up_date` (String - Date, Optional): Recommended checkup date.
  * `follow_up_instructions` (String, Optional): Care instructions.

#### Request Example
```json
{
  "discharge_summary": "Patient Ramesh Sharma (47M) underwent an uneventful Laparoscopic Appendectomy. Post-op recovery was stable.",
  "follow_up_date": "2026-07-19",
  "follow_up_instructions": "Keep dressing dry. Suture removal in 7 days."
}
```

### Response
* **Success (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "e5f6g7h8-i9j0-1234-efgh-345678901234",
    "ipd_status": "READY_FOR_DISCHARGE",
    "updated_at": "2026-07-13T09:00:00Z"
  }
}
```

---

### ➔ Transition: Step 21 to Step 22
* **Why Next API is Required:** Before a patient can physically leave the hospital premises, key departments (Billing, Pharmacy, Nursing, and the Attending Doctor) must sign off on their respective clearance gates.
* **IDs Carried Forward:** `admission_id` (`id` from Step 21) and the specific clearance gate details.
* **Workflow State Change:** `READY_FOR_DISCHARGE`
* **Required Permissions:** `admissions:edit`
* **Validations:** Discharge must have been initiated (status `READY_FOR_DISCHARGE`).

---

## Step 22: Update Discharge Clearance Gates
* **Purpose:** Log clearance sign-offs for the billing, pharmacy, nursing, and physician gates.
* **HTTP Method:** `POST`
* **Endpoint:** `/ipd/admissions/{id}/discharge/clearance`
* **Required Permission:** `admissions:edit`
* **Preconditions:** Discharge must be initiated.
* **Postconditions:** Specific gate updated to `CLEARED`. Once all gates are `CLEARED`, the checkout can be completed.
* **Affected DB Entities:** `ipd.discharge_clearances` (inserts or updates gate status row).

### Request
* **Path Parameters:**
  * `id` (UUID, Required): IPD Admission ID.
* **Body Schema:**
  * `gate_name` (String, Required): Valid values: `BILLING`, `PHARMACY`, `NURSING`, `DOCTOR`.
  * `gate_status` (String, Required): Valid values: `PENDING`, `CLEARED`, `BLOCKED`.
  * `blocking_reason` (String, Optional): Notes if blocked.

#### Request Example
```json
{
  "gate_name": "BILLING",
  "gate_status": "CLEARED"
}
```

### Response
* **Success (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "e5f6g7h8-i9j0-1234-efgh-345678901234",
    "gates": {
      "BILLING": "CLEARED",
      "PHARMACY": "PENDING",
      "NURSING": "PENDING",
      "DOCTOR": "PENDING"
    }
  }
}
```

*(Note: Repeat Step 22 for gates `PHARMACY`, `NURSING`, and `DOCTOR` to clear all departments).*

---

### ➔ Transition: Step 22 to Step 23
* **Why Next API is Required:** To complete the patient's checkout, release their assigned bed, and finalize their inpatient record.
* **IDs Carried Forward:** `admission_id` (`id` from Step 22).
* **Workflow State Change:** None.
* **Required Permissions:** `admissions:edit`
* **Validations:** All clearance gates must be in `CLEARED` status.

---

## Step 23: Complete Discharge
* **Purpose:** Final administrative check-out. Releases the bed resource back to available status.
* **HTTP Method:** `POST`
* **Endpoint:** `/ipd/admissions/{id}/discharge/complete`
* **Required Permission:** `admissions:edit`
* **Preconditions:** All clearance gates must be cleared.
* **Postconditions:** Admission status becomes `DISCHARGED`. The ward bed is released back to `AVAILABLE`.
* **Affected DB Entities:** `ipd.ipd_admissions` (status set to `DISCHARGED`, `ipd_status` set to `DISCHARGED`, `discharged_at` timestamp recorded), `ipd.beds` (status set to `AVAILABLE`).

### Request
* **Path Parameters:**
  * `id` (UUID, Required): IPD Admission ID.
* **Body Schema:**
  * `discharge_summary` (String, Required): Final clinical summary.
  * `medications_on_discharge` (Object, Optional): Prescribed discharge drugs.
  * `diet_instructions` (String, Optional): Post-discharge diet notes.

#### Request Example
```json
{
  "discharge_summary": "Patient Ramesh Sharma (47M) underwent an uneventful Laparoscopic Appendectomy. Suture healing clean, no signs of infection.",
  "medications_on_discharge": {
    "items": [
      { "name": "Tab Paracetamol 650mg", "dosage": "1-0-1", "duration": "5 days" },
      { "name": "Tab Pantocid 40mg", "dosage": "1-0-0", "duration": "5 days" }
    ]
  },
  "diet_instructions": "Light soft diet for 3 days, avoid spicy food."
}
```

### Response
* **Success (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "e5f6g7h8-i9j0-1234-efgh-345678901234",
    "status": "DISCHARGED",
    "ipd_status": "DISCHARGED",
    "discharged_at": "2026-07-13T10:30:00Z"
  }
}
```

---

### ➔ Transition: Step 23 to Step 24
* **Why Next API is Required:** To complete the final administrative closure of the inpatient case, ensuring all clinical audits and financial checks are finalized.
* **IDs Carried Forward:** `admission_id` (`id` from Step 23).
* **Workflow State Change:** `DISCHARGED`
* **Required Permissions:** `admissions:close`
* **Validations:** Patient must be fully discharged (status `DISCHARGED`).

---

## Step 24: Close Case
* **Purpose:** Archive the IPD case history. This is the terminal step in the patient journey.
* **HTTP Method:** `POST`
* **Endpoint:** `/ipd/admissions/{id}/close`
* **Required Permission:** `admissions:close`
* **Preconditions:** Patient must be discharged.
* **Postconditions:** Inpatient admission case record is closed. No further edits are allowed.
* **Affected DB Entities:** `ipd.ipd_admissions` (sets `case_closed = true`, records `case_closed_at` timestamp).

### Request
* **Path Parameters:**
  * `id` (UUID, Required): IPD Admission ID.
* **Body Schema:**
  * `billing_cleared` (Boolean, Required): Must be `true`.
  * `closure_notes` (String, Optional): Administrative closure comments.

#### Request Example
```json
{
  "billing_cleared": true,
  "closure_notes": "All medical and billing records audited and verified. Case closed."
}
```

### Response
* **Success (201 Created):**
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "e5f6g7h8-i9j0-1234-efgh-345678901234",
    "case_closed": true,
    "case_closed_at": "2026-07-13T11:00:00Z"
  }
}
```

---

# 🛣️ Flow 2: Direct Admission / Referral Surgery → Surgery → Discharge

This workflow is for patients referred directly from external clinics/doctors for surgery, bypassing the hospital's standard outpatient (OPD) consultation.

### 🔗 Capturing and Tracking Referral Information
* **Intake:** The patient's incoming referral is registered using the `POST /doctors/referral` API. This records the referring physician, organization, and the core clinical indication in the `clinical.referrals` table.
* **Linkage:** The generated `referral_id` is passed as the `source_reference` in the IPD Admission Request, with `source_type` set to `REFERRAL`.
* **Direct Admission Check-In:** The patient is admitted directly to an inpatient ward bed without an active OPD visit.
* **Clinical Order:** The surgeon advises surgery (`POST /opd/surgery`) using the active `admission_id` instead of an `opd_visit_id`. This links the surgical order directly to the inpatient record and carries the referral context forward.

---

## Step 1: Register Patient
*(Identical to Flow 1, Step 1. Generates `patient_id` = `e2c07a01-2092-4876-8889-112233445566`).*

---

### ➔ Transition: Step 1 to Step 2
* **Why Next API is Required:** To document the external referral details and link the patient to the referring clinician in the clinical registry.
* **IDs Carried Forward:** `patient_id`.
* **Workflow State Change:** `REGISTERED`
* **Required Permissions:** `referrals:create`
* **Validations:** Patient must be registered.

---

## Step 2: Record External Referral
* **Purpose:** Log the incoming referral details from the external doctor.
* **HTTP Method:** `POST`
* **Endpoint:** `/doctors/referral`
* **Required Permission:** `referrals:create`
* **Preconditions:** Patient must be registered.
* **Postconditions:** A referral record is created with status `PENDING`.
* **Affected DB Entities:** `clinical.referrals` (inserts referral details).

### Request
* **Body Schema:**
  * `patient_id` (UUID, Required): Patient ID.
  * `referred_to_dept_id` (UUID, Required): Specialty department.
  * `referred_to_doctor_id` (UUID, Optional): Target surgeon ID.
  * `referral_reason` (String, Required): Reason for referral.
  * `clinical_summary` (String, Optional): EMR summary notes.
  * `urgency` (String, Optional): Valid values: `ROUTINE`, `URGENT`, `EMERGENCY` (default: `ROUTINE`).
  * `external_doctor_name` (String, Optional): Name of the external referring clinician.

#### Request Example
```json
{
  "patient_id": "e2c07a01-2092-4876-8889-112233445566",
  "referred_to_dept_id": "d1234567-abcd-efgh-ijkl-123456789012",
  "referred_to_doctor_id": "97d2a10d-1b8a-40b4-a37a-5daf4e35f630",
  "referral_reason": "Acute calculus cholecystitis - recommended for Laparoscopic Cholecystectomy",
  "clinical_summary": "Gallstones seen on ultrasound. Recommended for surgery.",
  "urgency": "URGENT",
  "external_doctor_name": "Dr. A. K. Sen (City Clinic)"
}
```

### Response
* **Success (201 Created):**
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "ref-778899-abcd-efgh-ijkl-567890123456",
    "patient_id": "e2c07a01-2092-4876-8889-112233445566",
    "status": "PENDING",
    "urgency": "URGENT",
    "external_doctor_name": "Dr. A. K. Sen (City Clinic)"
  }
}
```

---

### ➔ Transition: Step 2 to Step 3
* **Why Next API is Required:** To request inpatient bed allocation, referencing the external referral record.
* **IDs Carried Forward:** `patient_id` and `referral_id` (`data.id` from Step 2) as `source_reference`.
* **Workflow State Change:** None.
* **Required Permissions:** `admissions:create`
* **Validations:** Referral status must be active.

---

## Step 3: Create Admission Request (From Referral)
* **Purpose:** Initiate direct admission booking for the referred patient.
* **HTTP Method:** `POST`
* **Endpoint:** `/ipd/admission-requests`
* **Required Permission:** `admissions:create`
* **Preconditions:** Referral must be logged.
* **Postconditions:** Request logged with status `REQUESTED` and `source_type` = `REFERRAL`.
* **Affected DB Entities:** `ipd.admission_requests` (inserts request).

### Request
* **Body Schema:**
  * `patient_id` (UUID, Required): Patient ID.
  * `source_type` (String, Required): Must be `REFERRAL`.
  * `source_reference` (UUID, Required): Referral record ID from Step 2.
  * `request_reason` (String, Required): Reason details.
  * `admission_type` (String, Optional): Valid values: `ELECTIVE`, `EMERGENCY` (default: `ELECTIVE`).
  * `priority` (String, Optional): Valid values: `ROUTINE`, `URGENT` (default: `ROUTINE`).
  * `preferred_ward_id` (UUID, Optional): Ward ID.

#### Request Example
```json
{
  "patient_id": "e2c07a01-2092-4876-8889-112233445566",
  "source_type": "REFERRAL",
  "source_reference": "ref-778899-abcd-efgh-ijkl-567890123456",
  "request_reason": "Direct admission for Cholecystectomy as advised by referral doctor",
  "admission_type": "ELECTIVE",
  "priority": "URGENT",
  "preferred_ward_id": "4c5d12d2-79dd-4650-867e-079ac3b85a5a"
}
```

### Response
* **Success (201 Created):**
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "7848855e-d0fc-4710-bc94-76afc790cdc7",
    "patient_id": "e2c07a01-2092-4876-8889-112233445566",
    "source_type": "REFERRAL",
    "source_reference": "ref-778899-abcd-efgh-ijkl-567890123456",
    "status": "REQUESTED"
  }
}
```

---

### ➔ Transition: Step 3 to Step 4
* **Why Next API is Required:** Admission desk verifies the referral details, confirms room availability, and approves the request.
* **IDs Carried Forward:** `request_id` (`data.id` from Step 3).
* **Workflow State Change:** `ADMISSION_REQUESTED`
* **Required Permissions:** `admissions:edit`
* **Validations:** Request status must be `REQUESTED`.

---

## Step 4: Approve Admission Request
*(Identical to Flow 1, Step 7. Request ID is updated to `READY`).*

---

### ➔ Transition: Step 4 to Step 5
* **Why Next API is Required:** To formally check the patient into their assigned bed in the ward.
* **IDs Carried Forward:** `patient_id` and `request_id`.
* **Workflow State Change:** `ADMISSION_APPROVED`
* **Required Permissions:** `admissions:create`
* **Validations:** Target bed must be `AVAILABLE`.

---

## Step 5: Create IPD Admission (Check-In)
*(Identical to Flow 1, Step 8. Returns `admission_id` = `e5f6g7h8-i9j0-1234-efgh-345678901234`).*

---

### ➔ Transition: Step 5 to Step 6
* **Why Next API is Required:** Since the patient bypassed OPD consultation, the surgical order must be created directly under the active IPD admission.
* **IDs Carried Forward:** `patient_id` and `admission_id`.
* **Workflow State Change:** `ADMITTED` / `BED_ASSIGNED`
* **Required Permissions:** `surgery:create`
* **Validations:** Patient must currently be admitted (`status` = `ADMITTED`).

---

## Step 6: Advise Surgery / Create Surgery Order (IPD Context)
* **Purpose:** Log the clinical surgery order directly under the active IPD admission record.
* **HTTP Method:** `POST`
* **Endpoint:** `/opd/surgery`
* **Required Permission:** `surgery:create`
* **Preconditions:** Patient must be admitted.
* **Postconditions:** A surgery order is created with status `ORDERED` and linked to the active `admission_id`.
* **Affected DB Entities:** `clinical.service_orders` (inserts surgery order details, sets `ipd_admission_id` = `admission_id` and `opd_visit_id` = `null`).

### Request
* **Body Schema:**
  * `patient_id` (UUID, Required): Patient ID.
  * `admission_id` (UUID, Required): Active Inpatient Admission ID from Step 5.
  * `order_type` (String, Required): Must be `SURGERY`.
  * `service_name` (String, Required): Surgery name.
  * `assigned_doctor_id` (UUID, Required): Operating surgeon ID.
  * `priority` (String, Required): Valid values: `ROUTINE`, `URGENT`, `EMERGENCY`.
  * `notes` (String, Optional): Notes.

#### Request Example
```json
{
  "patient_id": "e2c07a01-2092-4876-8889-112233445566",
  "admission_id": "e5f6g7h8-i9j0-1234-efgh-345678901234",
  "order_type": "SURGERY",
  "service_name": "Laparoscopic Cholecystectomy",
  "assigned_doctor_id": "97d2a10d-1b8a-40b4-a37a-5daf4e35f630",
  "priority": "URGENT",
  "notes": "Direct admission based on referral diagnostic tests."
}
```

### Response
* **Success (201 Created):**
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "c3d4e5f6-g7h8-9012-cdef-123456789012",
    "patient_id": "e2c07a01-2092-4876-8889-112233445566",
    "admission_id": "e5f6g7h8-i9j0-1234-efgh-345678901234",
    "order_type": "SURGERY",
    "service_name": "Laparoscopic Cholecystectomy",
    "status": "ORDERED"
  }
}
```

---

### ➔ Transition: Step 6 to Step 7
* **Why Next API is Required:** To create the OT session and schedule the surgery slot.
* **IDs Carried Forward:** `patient_id`, `admission_id`, and `service_order_id`.
* **Workflow State Change:** `SURGERY_ORDERED`
* **Required Permissions:** `ot:schedule`
* **Validations:** Surgery order must be `ORDERED`.

---

## Step 7: Create OT Session
*(Identical to Flow 1, Step 9. Returns `ot_session_id`. The patient then proceeds through the identical OT, Consent, PAC, billing, clearance, and discharge steps defined in Flow 1, Steps 10-24).*

---

# 🛣️ Flow 3: Emergency → Surgery → Discharge

This workflow covers critical, life-threatening emergency admissions requiring immediate surgical intervention. It details the triage intake, immediate bed assignment, emergency admission, and the **Emergency PAC Override** mechanism which bypasses signed consents to proceed with surgery.

---

## Step 1: Quick Patient Registration
* **Purpose:** Fast-track registration for emergency triage. If the patient is unidentified or unconscious, minimal details are used.
* **HTTP Method:** `POST`
* **Endpoint:** `/patients/register`
* **Required Permission:** `patients:create`
* **Preconditions:** Patient arrives at ER.
* **Postconditions:** Patient record created.
* **Affected DB Entities:** `patient.patients`.

### Request
* **Body Schema:**
  * `full_name` (String, Required): Name or placeholder (e.g., "Unknown Male 01").
  * `phone` (String, Required): Placeholder or emergency contact number (must be > 10 chars).
  * `age` (Integer, Required): Estimated or actual age.
  * `gender` (String, Optional): Valid values: `MALE`, `FEMALE`, `UNKNOWN` (default: `UNKNOWN`).
  * `blood_group` (String, Optional): Default: `UNKNOWN`.

#### Request Example
```json
{
  "full_name": "Unknown Male 01 (Trauma)",
  "phone": "+910000000000",
  "age": 30,
  "gender": "MALE",
  "blood_group": "UNKNOWN"
}
```

### Response
* **Success (201 Created):**
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "e2c07a01-2092-4876-8889-112233445566",
    "uhid": "TEMP-EMG-2026-0001",
    "full_name": "Unknown Male 01 (Trauma)",
    "phone": "+910000000000",
    "age": 30,
    "gender": "MALE",
    "status": "REGISTERED"
  }
}
```

---

### ➔ Transition: Step 1 to Step 2
* **Why Next API is Required:** To create the clinical emergency encounter in the OPD service and set the triage priority level.
* **IDs Carried Forward:** `patient_id`.
* **Workflow State Change:** `REGISTERED`
* **Required Permissions:** `emergency:create`
* **Validations:** Patient ID must exist.

---

## Step 2: Create Emergency Visit
* **Purpose:** Log the emergency encounter details and assign triage level.
* **HTTP Method:** `POST`
* **Endpoint:** `/opd/emergency`
* **Required Permission:** `emergency:create`
* **Preconditions:** Quick registration complete.
* **Postconditions:** Emergency visit status is set to `ARRIVED`.
* **Affected DB Entities:** `clinical.encounters` (inserts encounter header), `clinical.emergency_visits` (inserts ER encounter specifics).

### Request
* **Body Schema:**
  * `patient_id` (UUID, Required): Patient ID.
  * `triage_level` (String, Required): Must be `RED` (immediate life threat), `ORANGE`, `YELLOW`, or `GREEN`.
  * `chief_complaint` (String, Required): Description of symptoms.
  * `arrival_mode` (String, Required): Valid values: `WALK_IN`, `AMBULANCE`, `REFERRAL`, `TRANSFER`.
  * `assigned_doctor_id` (UUID, Optional): Attending ER physician.

#### Request Example
```json
{
  "patient_id": "e2c07a01-2092-4876-8889-112233445566",
  "triage_level": "RED",
  "chief_complaint": "Poly-trauma following road traffic accident. Active abdominal bleeding.",
  "arrival_mode": "AMBULANCE",
  "assigned_doctor_id": "97d2a10d-1b8a-40b4-a37a-5daf4e35f630"
}
```

### Response
* **Success (201 Created):**
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "emg-visit-uuid-001",
    "patient_id": "e2c07a01-2092-4876-8889-112233445566",
    "triage_level": "RED",
    "status": "ARRIVED"
  }
}
```

---

### ➔ Transition: Step 2 to Step 3
* **Why Next API is Required:** To immediately assign the patient to a physical triage bay bed for stabilization.
* **IDs Carried Forward:** `visit_id` (`data.id` from Step 2) and target ER bed details.
* **Workflow State Change:** None.
* **Required Permissions:** `emergency:update`
* **Validations:** Target bed must be `AVAILABLE` and located in the Emergency ward.

---

## Step 3: Assign / Transfer Emergency Bed
* **Purpose:** Atomically assign a physical ER bed resource.
* **HTTP Method:** `PATCH`
* **Endpoint:** `/ipd/emergency/{visit_id}/bed`
* **Required Permission:** `emergency:update`
* **Preconditions:** ER visit must be registered.
* **Postconditions:** The selected ER bed status is updated to `OCCUPIED`.
* **Affected DB Entities:** `ipd.beds` (status set to `OCCUPIED`), `clinical.emergency_visits` (records `assigned_bed_id` = `bed_id`).

### Request
* **Path Parameters:**
  * `visit_id` (UUID, Required): Emergency Visit ID from Step 2.
* **Body Schema:**
  * `bed_id` (UUID, Required): Target ER bed ID.

#### Request Example
```json
{
  "bed_id": "bed-er-001122-abcd-efgh-ijkl-123456"
}
```

### Response
* **Success (200 OK):**
```json
{
  "success": true,
  "message": "Emergency bed assignment updated",
  "data": {
    "visit_id": "emg-visit-uuid-001",
    "previous_bed_id": null,
    "assigned_bed_id": "bed-er-001122-abcd-efgh-ijkl-123456"
  }
}
```

---

### ➔ Transition: Step 3 to Step 4
* **Why Next API is Required:** ER evaluation indicates a ruptured spleen requiring immediate surgery. The ER doctor orders immediate inpatient admission.
* **IDs Carried Forward:** `patient_id` and `visit_id` (as `source_reference`).
* **Workflow State Change:** None.
* **Required Permissions:** `admissions:create`
* **Validations:** Visit ID must exist.

---

## Step 4: Create Emergency Admission Request
* **Purpose:** Request immediate emergency inpatient admission.
* **HTTP Method:** `POST`
* **Endpoint:** `/ipd/admission-requests`
* **Required Permission:** `admissions:create`
* **Preconditions:** Immediate surgery indicated.
* **Postconditions:** Admission request logged with status `REQUESTED` and priority `EMERGENCY`.
* **Affected DB Entities:** `ipd.admission_requests`.

### Request
* **Body Schema:**
  * `patient_id` (UUID, Required): Patient ID.
  * `source_type` (String, Required): Must be `EMERGENCY`.
  * `source_reference` (UUID, Required): ER Visit ID from Step 2.
  * `request_reason` (String, Required): Emergency reason details.
  * `admission_type` (String, Required): Must be `EMERGENCY`.
  * `priority` (String, Required): Must be `EMERGENCY`.
  * `preferred_ward_id` (UUID, Optional): ICU or Emergency ward target.

#### Request Example
```json
{
  "patient_id": "e2c07a01-2092-4876-8889-112233445566",
  "source_type": "EMERGENCY",
  "source_reference": "emg-visit-uuid-001",
  "request_reason": "Hemoperitoneum, ruptured spleen - requires emergency exploratory laparotomy",
  "admission_type": "EMERGENCY",
  "priority": "EMERGENCY",
  "preferred_ward_id": "4c5d12d2-79dd-4650-867e-079ac3b85abc"
}
```

### Response
* **Success (201 Created):**
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "d4e5f6g7-h8i9-0123-defg-234567890123",
    "patient_id": "e2c07a01-2092-4876-8889-112233445566",
    "source_type": "EMERGENCY",
    "source_reference": "emg-visit-uuid-001",
    "status": "REQUESTED"
  }
}
```

---

### ➔ Transition: Step 4 to Step 5
* **Why Next API is Required:** Admission request is instantly approved by the admitting supervisor or clinical director.
* **IDs Carried Forward:** `request_id` (`data.id` from Step 4).
* **Workflow State Change:** `ADMISSION_REQUESTED`
* **Required Permissions:** `admissions:edit`
* **Validations:** Request ID must exist.

---

## Step 5: Approve Admission Request (Emergency Bypass)
*(Identical to Flow 1, Step 7. Request ID status transitions to `READY`).*

---

### ➔ Transition: Step 5 to Step 6
* **Why Next API is Required:** To create the IPD admission record and allocate an ICU or emergency ward bed resource.
* **IDs Carried Forward:** `patient_id`, `request_id` (as `admission_request_id`), and `visit_id` (as `emergency_visit_id`).
* **Workflow State Change:** `ADMISSION_APPROVED`
* **Required Permissions:** `admissions:create`
* **Validations:** Target bed must be `AVAILABLE`.

---

## Step 6: Create IPD Admission (Emergency Admission)
* **Purpose:** Formally admit the patient into an IPD ward bed, linking the emergency visit context.
* **HTTP Method:** `POST`
* **Endpoint:** `/ipd/admissions`
* **Required Permission:** `admissions:create`
* **Preconditions:** Admission request must be `READY`.
* **Postconditions:** Patient status updated to `ADMITTED`. The ER bed is released, and the new IPD bed is marked `OCCUPIED`.
* **Affected DB Entities:** `ipd.ipd_admissions` (inserts admission details), `ipd.beds` (releases previous ER bed; locks new IPD ward bed to `OCCUPIED`), `ipd.admission_requests` (status set to `CONVERTED`).

### Request
* **Body Schema:**
  * `patient_id` (UUID, Required): Patient ID.
  * `admission_request_id` (UUID, Required): Request ID.
  * `emergency_visit_id` (UUID, Required): ER Visit ID.
  * `ward_id` (UUID, Required): Target inpatient ward (e.g., ICU ward).
  * `bed_id` (UUID, Required): Target bed ID. Must be `AVAILABLE`.
  * `attending_doctor_id` (UUID, Required): Attending Surgeon ID.
  * `admission_type` (String, Required): Must be `EMERGENCY`.
  * `admission_reason` (String, Required): Clinical explanation.

#### Request Example
```json
{
  "patient_id": "e2c07a01-2092-4876-8889-112233445566",
  "admission_request_id": "d4e5f6g7-h8i9-0123-defg-234567890123",
  "emergency_visit_id": "emg-visit-uuid-001",
  "ward_id": "4c5d12d2-79dd-4650-867e-079ac3b85abc",
  "bed_id": "bed-icu-0055-abcd-efgh-ijkl-123456",
  "attending_doctor_id": "97d2a10d-1b8a-40b4-a37a-5daf4e35f630",
  "admission_type": "EMERGENCY",
  "admission_reason": "Acute internal bleeding, splenic laceration"
}
```

### Response
* **Success (201 Created):**
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "e5f6g7h8-i9j0-1234-efgh-345678901234",
    "admission_number": "ADM-2026-EMG01",
    "status": "ADMITTED",
    "ipd_status": "ADMITTED",
    "admitted_at": "2026-07-13T12:15:00Z"
  }
}
```

---

### ➔ Transition: Step 6 to Step 7
* **Why Next API is Required:** To transition the source emergency visit encounter status to `ADMITTED` in the OPD registry, linking it to the active IPD admission.
* **IDs Carried Forward:** `visit_id`.
* **Workflow State Change:** `ADMITTED`
* **Required Permissions:** `emergency:update`
* **Validations:** Inpatient admission must be successfully created.

---

## Step 7: Update Emergency Visit Status (Mark Admitted)
* **Purpose:** Mark the ER encounter as completed by admission.
* **HTTP Method:** `PATCH`
* **Endpoint:** `/opd/emergency/{visit_id}/status`
* **Required Permission:** `emergency:update`
* **Preconditions:** IPD admission created.
* **Postconditions:** Emergency visit status set to `ADMITTED`.
* **Affected DB Entities:** `clinical.emergency_visits` (status set to `ADMITTED`).

### Request
* **Path Parameters:**
  * `visit_id` (UUID, Required): ER Visit ID.
* **Body Schema:**
  * `status` (String, Required): Must be `ADMITTED`.
  * `notes` (String, Optional): Notes.

#### Request Example
```json
{
  "status": "ADMITTED",
  "notes": "Patient transferred to ICU Bed ICU-0055. Explored for emergency surgery."
}
```

### Response
* **Success (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "emg-visit-uuid-001",
    "status": "ADMITTED",
    "updated_at": "2026-07-13T12:20:00Z"
  }
}
```

---

### ➔ Transition: Step 7 to Step 8
* **Why Next API is Required:** Surgeon logs the emergency surgical advice under the active IPD admission.
* **IDs Carried Forward:** `patient_id` and `admission_id`.
* **Workflow State Change:** None.
* **Required Permissions:** `surgery:create`
* **Validations:** Admission must be active.

---

## Step 8: Advise Surgery / Create Surgery Order (Emergency Context)
* **Purpose:** Create the emergency surgery order.
* **HTTP Method:** `POST`
* **Endpoint:** `/opd/surgery`
* **Required Permission:** `surgery:create`
* **Preconditions:** Admission completed.
* **Postconditions:** Service order status set to `ORDERED` with priority `EMERGENCY`.
* **Affected DB Entities:** `clinical.service_orders`.

### Request
* **Body Schema:**
  * `patient_id` (UUID, Required): Patient ID.
  * `admission_id` (UUID, Required): IPD Admission ID.
  * `order_type` (String, Required): Must be `SURGERY`.
  * `service_name` (String, Required): Surgery name.
  * `assigned_doctor_id` (UUID, Required): Surgeon ID.
  * `priority` (String, Required): Must be `EMERGENCY`.
  * `notes` (String, Optional): Notes.

#### Request Example
```json
{
  "patient_id": "e2c07a01-2092-4876-8889-112233445566",
  "admission_id": "e5f6g7h8-i9j0-1234-efgh-345678901234",
  "order_type": "SURGERY",
  "service_name": "Emergency Splenectomy",
  "assigned_doctor_id": "97d2a10d-1b8a-40b4-a37a-5daf4e35f630",
  "priority": "EMERGENCY",
  "notes": "Ruptured spleen with severe internal hemoperitoneum."
}
```

### Response
* **Success (201 Created):**
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "c3d4e5f6-g7h8-9012-cdef-123456789012",
    "patient_id": "e2c07a01-2092-4876-8889-112233445566",
    "admission_id": "e5f6g7h8-i9j0-1234-efgh-345678901234",
    "order_type": "SURGERY",
    "status": "ORDERED",
    "priority": "EMERGENCY"
  }
}
```

---

### ➔ Transition: Step 8 to Step 9
* **Why Next API is Required:** To immediately schedule the emergency OT slot.
* **IDs Carried Forward:** `patient_id`, `admission_id`, and `service_order_id`.
* **Workflow State Change:** `SURGERY_ORDERED`
* **Required Permissions:** `ot:schedule`
* **Validations:** Surgery order must be active.

---

## Step 9: Schedule OT Session
*(Identical to Flow 1, Step 9, with immediate `scheduled_start` timestamp. Returns `ot_session_id` = `g7h8i9j0-k1l2-3456-ghij-567890123456`).*

---

### ➔ Transition: Step 9 to Step 10
* **Why Next API is Required:** To generate emergency consent templates.
* **IDs Carried Forward:** `ot_session_id` (as `ot_case_id`).
* **Workflow State Change:** `OT_SCHEDULED`
* **Required Permissions:** `consent:create`
* **Validations:** Session status must be `SCHEDULED`.

---

## Step 10: Create OT Case Consents
*(Identical to Flow 1, Step 10. Generates pending consents for `SURGICAL`, `ANAESTHESIA`, and `PROCEDURE`).*

---

### ➔ Transition: Step 10 to Step 11
* **Why Next API is Required:** If the patient is unconscious or incapacitated, legal consent must be signed by an authorized guardian.
* **IDs Carried Forward:** `consent_id` (for each pending consent form).
* **Workflow State Change:** None.
* **Required Permissions:** `consent:sign`
* **Validations:** Consent status must be `PENDING`.

---

## Step 11: Sign Consent (Guardian Signature)
* **Purpose:** Log consent signed by the patient's legal guardian due to emergency incapacitation.
* **HTTP Method:** `POST`
* **Endpoint:** `/ipd/consents/{consent_id}/sign`
* **Required Permission:** `consent:sign`
* **Preconditions:** Consent template exists.
* **Postconditions:** Consent status set to `SIGNED`.
* **Affected DB Entities:** `clinical.consents`.

### Request
* **Path Parameters:**
  * `consent_id` (UUID, Required): Target consent ID.
* **Body Schema:**
  * `signature_type` (String, Required): Must be `DIGITAL`.
  * `signed_by` (UUID, Optional): Signing staff user ID.
  * `guardian_name` (String, Required): Guardian's full name.
  * `guardian_relation` (String, Required): Relationship to patient (e.g., "SPOUSE", "FATHER", "GUARDIAN").
  * `doctor_id` (UUID, Required): Admitting surgeon witnessing.

#### Request Example
```json
{
  "signature_type": "DIGITAL",
  "signed_by": "77d2a10d-1b8a-40b4-a37a-5daf4e35f680",
  "guardian_name": "Sita Sharma",
  "guardian_relation": "SPOUSE",
  "doctor_id": "97d2a10d-1b8a-40b4-a37a-5daf4e35f630"
}
```

### Response
* **Success (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "con-surg-0001-abcd-efgh-ijklmnopqrst",
    "status": "SIGNED",
    "guardian_name": "Sita Sharma",
    "guardian_relation": "SPOUSE"
  }
}
```

---

### ➔ Transition: Step 11 to Step 12
* **Why Next API is Required:** If certain consents (e.g. anaesthesia or procedure) cannot be signed in time due to immediate threat to life, a two-doctor emergency override is executed to bypass the validation and unlock the PAC clearance.
* **IDs Carried Forward:** `ot_session_id` (`id` from Step 9).
* **Workflow State Change:** None.
* **Required Permissions:** `ot:pac`
* **Validations:** Bypasses standard signature checks when `emergency_override` = `true`. Requires two distinct doctor IDs.

---

## Step 12: Clear PAC (Pre-Anaesthetic Checkup) - Emergency Override
* **Purpose:** Clear the pre-anaesthetic gate using the emergency override mechanism, bypassing standard pending consent validation checks.
* **HTTP Method:** `POST`
* **Endpoint:** `/ipd/ot-sessions/{id}/pac`
* **Required Permission:** `ot:pac`
* **Preconditions:** Life-threatening emergency.
* **Postconditions:** `pac_cleared` is set to `true` on the session.
* **Affected DB Entities:** `ipd.ot_sessions` (sets `pac_cleared = true`, logs override details).

### Request
* **Path Parameters:**
  * `id` (UUID, Required): OT session ID.
* **Body Schema:**
  * `notes` (String, Required): Override details.
  * `emergency_override` (Boolean, Required): Must be `true`.
  * `override_reason` (String, Required): Clinical justification.
  * `primary_doctor_id` (UUID, Required): Operating Surgeon.
  * `second_doctor_id` (UUID, Required): Chief Medical Officer or distinct senior consultant (must be different from primary doctor).

#### Request Example
```json
{
  "notes": "Emergency bypass - patient in hypovolemic shock. Proceeding directly.",
  "emergency_override": true,
  "override_reason": "Immediate life-saving exploratory laparotomy for splenic rupture",
  "primary_doctor_id": "97d2a10d-1b8a-40b4-a37a-5daf4e35f630",
  "second_doctor_id": "doc2-5678-abcd-1234-efgh-567890123456"
}
```

### Response
* **Success (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "g7h8i9j0-k1l2-3456-ghij-567890123456",
    "status": "SCHEDULED",
    "pac_cleared": true,
    "emergency_override": true,
    "override_reason": "Immediate life-saving exploratory laparotomy for splenic rupture"
  }
}
```
* **Error Responses:**
  * **400 Bad Request:** Primary and second doctor IDs are not distinct.

---

### ➔ Transition: Step 12 to Step 13
* **Why Next API is Required:** To progress to surgical checklist verification and enter the OR.
* **IDs Carried Forward:** `ot_session_id`.
* **Workflow State Change:** None.
* **Required Permissions:** `ot:pre-op`
* **Validations:** `pac_cleared` must be `true`.

---

## Step 13: Complete Pre-Op Checklist (Emergency)
* **Purpose:** Log preoperative checklist completion by the floor nurse (rapid emergency format).
* **HTTP Method:** `POST`
* **Endpoint:** `/ipd/ot-sessions/{id}/pre-op`
* **Required Permission:** `ot:pre-op`
* **Preconditions:** PAC must be cleared.
* **Postconditions:** OT session status advances to `PRE_OP`.
* **Affected DB Entities:** `ipd.ot_sessions` (updates status to `PRE_OP`, stores checklist JSON).

### Request
* **Path Parameters:**
  * `id` (UUID, Required): OT session ID.
* **Body Schema:**
  * `pre_op_checklist` (Object, Required): JSON checklist structure.

#### Request Example
```json
{
  "pre_op_checklist": {
    "patient_identity_verified": true,
    "consent_verified": false,
    "emergency_override_active": true,
    "surgical_site_marked": true,
    "npo_status_verified": false,
    "vitals_stable": false
  }
}
```

### Response
* **Success (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "g7h8i9j0-k1l2-3456-ghij-567890123456",
    "status": "PRE_OP",
    "pre_op_completed_at": "2026-07-13T12:30:00Z"
  }
}
```

---

### ➔ Transition: Step 13 to Step 14
* **Why Next API is Required:** To record the start of incision.
* **IDs Carried Forward:** `ot_session_id` (`id` from Step 13).
* **Workflow State Change:** `PRE_OP`
* **Required Permissions:** `ot:start`
* **Validations:** Session status must be `PRE_OP`.

---

## Step 14: Start OT Session (Surgery Begins)
*(Identical to Flow 1, Step 14. Status set to `IN_PROGRESS`).*

---

### ➔ Transition: Step 14 to Step 15
* **Why Next API is Required:** To charge emergency supplies used.
* **IDs Carried Forward:** `ot_session_id` (`id` from Step 14).
* **Workflow State Change:** `IN_OT`
* **Required Permissions:** `ot:consumables`
* **Validations:** OT session status must be active (`IN_PROGRESS`).

---

## Step 15: Log OT Consumables (Emergency Log)
* **Purpose:** Charge emergency supplies to the patient's billing record.
* **HTTP Method:** `POST`
* **Endpoint:** `/ipd/ot-sessions/{id}/consumables`
* **Required Permission:** `ot:consumables`
* **Preconditions:** Surgery must be currently in progress.
* **Postconditions:** Charges are added to the billing register.
* **Affected DB Entities:** `ipd.ot_consumables` (inserts item consumption log), `billing.charge_items` (inserts chargeable items linked to the admission).

### Request
* **Path Parameters:**
  * `id` (UUID, Required): OT session ID.
* **Body Schema:**
  * `item_name` (String, Required): Name of the inventory item.
  * `quantity` (Number, Required): Quantity used (must be > 0).
  * `batch_no` (String, Optional): Material batch tracking code.

#### Request Example
```json
{
  "item_name": "Emergency Blood Unit O-Negative",
  "quantity": 3.0,
  "batch_no": "B-BL-00912"
}
```

### Response
* **Success (201 Created):**
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "consumable-uuid-771199",
    "ot_session_id": "g7h8i9j0-k1l2-3456-ghij-567890123456",
    "item_name": "Emergency Blood Unit O-Negative",
    "quantity": 3.0,
    "batch_no": "B-BL-00912"
  }
}
```

---

### ➔ Transition: Step 15 to Step 16
* **Why Next API is Required:** To record completion of incision closure.
* **IDs Carried Forward:** `ot_session_id` (`id` from Step 15).
* **Workflow State Change:** None.
* **Required Permissions:** `ot:complete`
* **Validations:** Session status must be `IN_PROGRESS`.

---

## Step 16: Complete OT Session
*(Identical to Flow 1, Step 16. Returns status `POST_OP`).*

---

### ➔ Transition: Step 16 to Step 17
* **Why Next API is Required:** To transfer the patient from the OR recovery room into a specialized ICU ward bed.
* **IDs Carried Forward:** `ot_session_id` (`id` from Step 16).
* **Workflow State Change:** `SURGERY_COMPLETED` / `RECOVERY`
* **Required Permissions:** `ot:transfer-out`
* **Validations:** ICU bed must be `AVAILABLE`.

---

## Step 17: Transfer Out from OT (Recovery → ICU Bed)
* **Purpose:** Releases the OT room, completes the OT session, and transfers the patient into the Intensive Care Unit (ICU) bed.
* **HTTP Method:** `POST`
* **Endpoint:** `/ipd/ot-sessions/{id}/transfer-out`
* **Required Permission:** `ot:transfer-out`
* **Preconditions:** Patient stable enough for transport.
* **Postconditions:** Target ICU bed is marked `OCCUPIED`. Old OT room released. Admission `ward_id` and `bed_id` updated to ICU.
* **Affected DB Entities:** `ipd.ot_sessions` (status set to `COMPLETED`), `clinical.service_orders` (status set to `COMPLETED`), `ipd.ipd_admissions` (updates `ward_id`, `bed_id`, `ipd_status = 'UNDER_TREATMENT'`), `ipd.beds` (sets new ICU bed status to `OCCUPIED`).

### Request
* **Path Parameters:**
  * `id` (UUID, Required): OT session ID.
* **Body Schema:**
  * `transferred_to_ward_id` (UUID, Required): ICU ward ID.
  * `transferred_to_bed_id` (UUID, Required): Target ICU bed ID. Must be `AVAILABLE`.
  * `recovery_notes` (String, Optional): Critical care transfer notes.

#### Request Example
```json
{
  "transferred_to_ward_id": "4c5d12d2-79dd-4650-867e-079ac3b85abc",
  "transferred_to_bed_id": "bed-icu-0055-abcd-efgh-ijkl-123456",
  "recovery_notes": "Ventilator support active. Transferred to ICU bed."
}
```

### Response
* **Success (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "g7h8i9j0-k1l2-3456-ghij-567890123456",
    "status": "COMPLETED",
    "transfer_at": "2026-07-13T14:15:00Z"
  }
}
```

---

### ➔ Transition: Step 17 to Step 18
* **Why Next API is Required:** To compile final charges.
* **IDs Carried Forward:** `patient_id` and `admission_id`.
* **Workflow State Change:** `POST_OP`
* **Required Permissions:** `billing:create`
* **Validations:** Inpatient status must be active.

---

## Steps 18 to 24 (Billing & Discharge)
*(The patient undergoes clinical recovery, settles billing, completes clearance gates, and is discharged, following the same sequence as Flow 1, Steps 18-24).*

---

# 🛑 3. Conditional Exception Branches & Edge Cases

The following diagrams and API contracts define the recovery/exception branches supported throughout the patient journey.

## 1. Surgery Rescheduling Flow
If the patient develops a contraindication (e.g., sudden clinical instability) before the incision, the scheduler updates the slot.

```mermaid
graph TD
    A[OT Session: SCHEDULED] -->|POST /ipd/ot-sessions/{id}/cancel| B[Cancel Session]
    B -->|Linked Service Order status reverts to ORDERED| C[Reschedule]
    C -->|PATCH /opd/surgery/{id}/schedule| D[OT Session re-scheduled]
```

### Cancel OT Session
* **HTTP Method:** `POST`
* **Endpoint:** `/ipd/ot-sessions/{id}/cancel`
* **Required Permission:** `ot:cancel`
* **Request Body:**
```json
{
  "reason": "Patient developed sudden tachycardia and high fever."
}
```
* **Response (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "g7h8i9j0-k1l2-3456-ghij-567890123456",
    "status": "CANCELLED",
    "cancellation_reason": "Patient developed sudden tachycardia and high fever."
  }
}
```
* **Impact:** The linked service order status in `clinical.service_orders` reverts to `ORDERED`.

### Reschedule Surgery Order
* **HTTP Method:** `PATCH`
* **Endpoint:** `/opd/surgery/{id}/schedule`
* **Required Permission:** `surgery:edit`
* **Request Body:**
```json
{
  "scheduled_at": "2026-07-15T08:00:00Z",
  "reason": "Postponed due to clinical stabilization requirements."
}
```
* **Response (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "c3d4e5f6-g7h8-9012-cdef-123456789012",
    "status": "SCHEDULED",
    "scheduled_at": "2026-07-15T08:00:00Z"
  }
}
```

---

## 2. Admission Request Rejection / Cancellation
If the patient cancels elective surgery before check-in, the request is cancelled.
* **HTTP Method:** `PATCH`
* **Endpoint:** `/ipd/admission-requests/{request_id}/status`
* **Required Permission:** `admissions:edit`
* **Request Body:**
```json
{
  "status": "CANCELLED",
  "reason": "Patient cancelled procedure due to personal reasons"
}
```
* **Response (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "7848855e-d0fc-4710-bc94-76afc790cdc7",
    "status": "CANCELLED",
    "notes": "Patient cancelled procedure due to personal reasons"
  }
}
```

---

## 3. Bed Unavailability Handling
If the preferred bed type is unavailable at check-in, the system returns a conflict.
* **Endpoint:** `POST /ipd/admissions`
* **Response (409 Conflict):**
```json
{
  "success": false,
  "code": 409,
  "error": "Conflict: Target bed ICU-0055 is currently occupied (status: OCCUPIED)."
}
```
* **Resolution:** Front desk queries bed availability registry `GET /ipd/beds?status=AVAILABLE&ward_id={uuid}` to select an alternate bed, then retries the `POST /ipd/admissions` request.

---

## 4. Billing Outstanding Block Override
If a patient must be discharged but has an outstanding balance (e.g., charity case or employee credit), the supervisor clears the block.
* **HTTP Method:** `POST`
* **Endpoint:** `/billing/clearance/{admission_id}/clear`
* **Required Permission:** `billing:clearance:edit`
* **Request Body:**
```json
{
  "clearance_type": "CHARITY",
  "approved_by": "admin-user-uuid-998811",
  "notes": "Approved for 100% waiver under trust fund."
}
```
* **Response (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "admission_id": "e5f6g7h8-i9j0-1234-efgh-345678901234",
    "cleared": true,
    "clearance_type": "CHARITY"
  }
}
```
* **Impact:** Bypasses outstanding balance verification, allowing the `BILLING` clearance gate to transition to `CLEARED`.

---

## 5. EMR Department Validation Failure
If clinical fields fail validation (e.g., diagnosis is too short), the API returns a validation error.
* **Response (400 Bad Request / 422 Unprocessable Entity):**
```json
{
  "success": false,
  "code": 400,
  "error": "Validation error: request_reason must contain at least 5 characters"
}
```

---

# 🔗 4. End-to-End Key Identifier Propagation Map

The following map shows how keys generated from initial actions are carried forward to maintain reference integrity throughout the clinical lifecycle:

```
[POST /patients/register]
       ↓
    patient_id (UUID)
       │
       ├─► [POST /opd/appointments]
       │         ↓
       │      appointment_id (UUID)
       │         ↓
       │      [POST /opd/appointments/{id}/confirm]
       │         ↓
       │      opd_visit_id (UUID)
       │         │
       │         ├─► [POST /opd/surgery] (links patient_id + opd_visit_id)
       │         │         ↓
       │         │      service_order_id (UUID)
       │         │         │
       │         │         ├─► [POST /ipd/admission-requests] (links patient_id + service_order_id)
       │         │         │         ↓
       │         │         │      request_id (UUID)
       │         │         │         ↓
       │         │         │      [POST /ipd/admissions] (links patient_id + request_id)
       │         │         │         ↓
       │         │         │      admission_id (UUID)
       │         │         │         │
       │         │         │         ├─► [POST /ipd/ot-sessions] (links patient_id + service_order_id + admission_id)
       │         │         │         │         ↓
       │         │         │         │      ot_session_id (UUID) (used as ot_case_id)
       │         │         │         │         │
       │         │         │         │         ├─► [POST /ipd/ot-cases/{ot_case_id}/consents]
       │         │         │         │         │         ↓
       │         │         │         │         │      consent_ids[] (UUID)
       │         │         │         │         │         ↓
       │         │         │         │         │      [POST /ipd/consents/{consent_id}/sign]
       │         │         │         │         │
       │         │         │         │         ├─► [POST /ipd/ot-sessions/{id}/pac]
       │         │         │         │         ├─► [POST /ipd/ot-sessions/{id}/pre-op]
       │         │         │         │         ├─► [POST /ipd/ot-sessions/{id}/start]
       │         │         │         │         ├─► [POST /ipd/ot-sessions/{id}/consumables]
       │         │         │         │         ├─► [POST /ipd/ot-sessions/{id}/complete]
       │         │         │         │         └─► [POST /ipd/ot-sessions/{id}/transfer-out]
       │         │         │         │
       │         │         │         ├─► [POST /billing/invoices] (links patient_id + admission_id)
       │         │         │         │         ↓
       │         │         │         │      invoice_id (UUID)
       │         │         │         │         ↓
       │         │         │         │      [POST /billing/collect]
       │         │         │         │
       │         │         │         ├─► [GET /billing/clearance/{admission_id}]
       │         │         │         ├─► [POST /ipd/admissions/{id}/discharge/initiate]
       │         │         │         ├─► [POST /ipd/admissions/{id}/discharge/clearance]
       │         │         │         ├─► [POST /ipd/admissions/{id}/discharge/complete]
       │         │         │         └─► [POST /ipd/admissions/{id}/close]
```
