# IPD Registration & Lodging Master Data API Specification

**Document Version:** 1.3.0  
**Target Audience:** Frontend Engineers, Integration Teams, Backend Engineers, QA Engineers  
**Service Scope:** Inpatient Department (IPD) Microservice (`services/ipd`)  
**Base URL:** `https://4t1eo222cl.execute-api.ap-south-1.amazonaws.com/dev/ipd`  
**Authentication:** Required (`Authorization: Bearer <Cognito Access Token>`)  
**Tenant Context Header:** `x-tenant-id: 46fc39d8-7c4e-4704-9430-f82d6dcfa34c` (or extracted from JWT principal)

---

## Global Request Headers

Every API request in this specification requires the following HTTP headers:

| Header Name | Type | Required? | Example Value | Description |
| :--- | :--- | :--- | :--- | :--- |
| `Authorization` | `String` | **Mandatory** | `Bearer eyJhbGciOiJSUzI1Ni...` | Valid Cognito Access JWT Token |
| `x-tenant-id` | `UUID` | **Mandatory** | `46fc39d8-7c4e-4704-9430-f82d6dcfa34c` | Active Branch / Tenant UUID |
| `Content-Type` | `String` | **Mandatory** | `application/json` | Request Payload MIME type |

---

## Table of Contents

1. [Master Data Hierarchy Overview](#1-master-data-hierarchy-overview)
2. [Floors API Specification](#2-floors-api-specification)
   - [GET /ipd/floors](#1-get-ipdfloors)
3. [Blocks API Specification](#3-blocks-api-specification)
   - [GET /ipd/blocks](#1-get-ipdblocks)
   - [POST /ipd/blocks](#2-post-ipdblocks)
   - [GET /ipd/blocks/{block_id}](#3-get-ipdblocksblock_id)
   - [PUT /ipd/blocks/{block_id}](#4-put-ipdblocksblock_id)
   - [DELETE /ipd/blocks/{block_id}](#5-delete-ipdblocksblock_id)
4. [Wards API Specification](#4-wards-api-specification)
   - [GET /ipd/wards](#1-get-ipdwards)
   - [GET /ipd/wards/{ward_id}](#2-get-ipdwardsward_id)
   - [PATCH /ipd/wards/{ward_id}](#3-patch-ipdwardsward_id)
   - [POST /ipd/wards/{ward_id}/block](#4-post-ipdwardsward_idblock)
   - [POST /ipd/wards/{ward_id}/maintenance](#5-post-ipdwardsward_idmaintenance)
5. [Beds API Specification](#5-beds-api-specification)
   - [GET /ipd/beds](#1-get-ipdbeds)
   - [GET /ipd/beds/{bed_id}](#2-get-ipdbedsbed_id)
   - [POST /ipd/beds](#3-post-ipdbeds)
   - [PATCH /ipd/beds/{bed_id}/status](#4-patch-ipdbedsbed_idstatus)
   - [POST /ipd/beds/{bed_id}/release](#5-post-ipdbedsbed_idrelease)
   - [POST /ipd/beds/transfer](#6-post-ipdbedstransfer)
   - [GET /ipd/beds/availability](#7-get-ipdbedsavailability)
6. [IPD Split-Stage Registration & Admission API Specification](#6-ipd-split-stage-registration--admission-api-specification)
   - [POST /ipd/admissions (Stage 1 - Draft Admission Creation)](#1-post-ipdadmissions-stage-1---draft-admission-creation)
   - [POST /ipd/admissions/{id}/confirm (Stage 2 - Admission Confirmation)](#2-post-ipdadmissionsidconfirm-stage-2---admission-confirmation)
     - [Option A: Immediate Payment](#option-a-immediate-payment)
     - [Option B: Pay Later Credit](#option-b-pay-later-credit)
     - [Option C: Insurance Pre-Authorization](#option-c-insurance-pre-authorization)
   - [POST /ipd/admissions/{id}/cancel (Stage 3 - Draft Cancellation)](#3-post-ipdadmissionsidcancel-stage-3---draft-cancellation)
   - [GET /ipd/admissions (List Admissions)](#4-get-ipdadmissions-list-admissions)
   - [GET /ipd/admissions/{id} (Get Admission Detail)](#5-get-ipdadmissionsid-get-admission-detail)

---

## 1. Master Data Hierarchy Overview

```
+-------------------------------------------------------------+
|                          Floor                              |
+------------------------------+------------------------------+
                               |
                               v
+-------------------------------------------------------------+
|                          Block                              |
+------------------------------+------------------------------+
                               |
                               v
+-------------------------------------------------------------+
|                          Ward                               |
+------------------------------+------------------------------+
                               |
                               v
+-------------------------------------------------------------+
|                          Bed                                |
|  (Status: AVAILABLE -> RESERVED -> OCCUPIED -> MAINTENANCE) |
+-------------------------------------------------------------+
```

---

## 2. Floors API Specification

### 1. GET `/ipd/floors`

Retrieves all hospital floors configured for the active tenant branch.

* **Method:** `GET`
* **Path:** `/ipd/floors`
* **Query Parameters:** *None*

#### Response Envelope (`200 OK`)
```json
{
  "success": true,
  "data": [
    {
      "id": "39ff007e-1daf-4c5f-87a7-be8f712a2e46",
      "floor_name": "First Floor",
      "floor_number": 1,
      "code": "FL-01",
      "description": "General & Private Surgical Wards",
      "is_active": true,
      "created_at": "2026-06-01T08:00:00Z"
    },
    {
      "id": "77777777-1daf-4c5f-87a7-be8f712a2e47",
      "floor_name": "Second Floor",
      "floor_number": 2,
      "code": "FL-02",
      "description": "ICU and Critical Care Units",
      "is_active": true,
      "created_at": "2026-06-01T08:00:00Z"
    }
  ]
}
```

---

## 3. Blocks API Specification

### 1. GET `/ipd/blocks`

Lists all blocks with optional filtering by parent floor.

* **Method:** `GET`
* **Path:** `/ipd/blocks`
* **Query Parameters:**

| Parameter | Type | Required? | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| `floor_id` | `UUID` | Optional | `null` | Filter blocks belonging to a specific floor ID. |

#### Response Envelope (`200 OK`)
```json
{
  "success": true,
  "data": [
    {
      "id": "4caf45de-b989-4c1c-b82d-b7be15e99ebd",
      "floor_id": "39ff007e-1daf-4c5f-87a7-be8f712a2e46",
      "floor_name": "First Floor",
      "name": "Block A - East Wing",
      "code": "BLK-A",
      "description": "Cardiology & Cardio-Thoracic Surgery Block",
      "is_active": true,
      "created_at": "2026-06-01T08:00:00Z"
    }
  ]
}
```

---

### 2. POST `/ipd/blocks`

Creates a new hospital block mapped to a floor.

* **Method:** `POST`
* **Path:** `/ipd/blocks`

#### Fields Breakdown

| Field | Type | Required? | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| `floor_id` | `UUID` | **Mandatory** | — | Parent floor ID. |
| `name` | `String` | **Mandatory** | — | Name of the block (1–100 chars). |
| `code` | `String` | **Mandatory** | — | Unique short identifier code (1–20 chars). |
| `description` | `String` | Optional | `null` | Detailed block description. |
| `is_active` | `Boolean` | Optional | `true` | Defaults to `true`. |

#### Sample Request Body
```json
{
  "floor_id": "39ff007e-1daf-4c5f-87a7-be8f712a2e46",
  "name": "Block B - West Wing",
  "code": "BLK-B",
  "description": "Orthopedic & Neurological Specialty Block",
  "is_active": true
}
```

#### Response Envelope (`201 Created`)
```json
{
  "success": true,
  "data": {
    "id": "4caf45de-b989-4c1c-b82d-b7be15e99ebd",
    "floor_id": "39ff007e-1daf-4c5f-87a7-be8f712a2e46",
    "name": "Block B - West Wing",
    "code": "BLK-B",
    "description": "Orthopedic & Neurological Specialty Block",
    "is_active": true,
    "created_at": "2026-07-31T11:00:00Z"
  }
}
```

---

### 3. GET `/ipd/blocks/{block_id}`

Retrieves detailed master information for a specific block.

* **Method:** `GET`
* **Path Parameters:** `block_id` (`UUID`, **Mandatory**) — e.g., `4caf45de-b989-4c1c-b82d-b7be15e99ebd`

---

### 4. PUT `/ipd/blocks/{block_id}`

Updates an existing block record.

* **Method:** `PUT`
* **Path Parameters:** `block_id` (`UUID`, **Mandatory**)

---

### 5. DELETE `/ipd/blocks/{block_id}`

Deactivates a block (soft delete).

* **Method:** `DELETE`
* **Path Parameters:** `block_id` (`UUID`, **Mandatory**)

---

## 4. Wards API Specification

### 1. GET `/ipd/wards`

Retrieves a paginated list of hospital wards with optional filters.

* **Method:** `GET`
* **Path:** `/ipd/wards`
* **Query Parameters:**

| Parameter | Type | Required? | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| `floor_id` | `UUID` | Optional | `null` | Filter by floor ID. |
| `block_id` | `UUID` | Optional | `null` | Filter by block ID. |
| `ward_type` | `String` | Optional | `null` | `GENERAL`, `SEMI_PRIVATE`, `PRIVATE`, `ICU`, `NICU`, `PICU`, `OBSERVATION`, `DAY_CARE`, `ISOLATION`, `HDU`, `OT`, `OTHER`. |
| `is_active` | `Boolean` | Optional | `true` | Filter by active status. |
| `page` | `Integer` | Optional | `1` | Page index (1-indexed). |
| `page_size` | `Integer` | Optional | `50` | Items per page (max 100). |

#### Response Envelope (`200 OK`)
```json
{
  "success": true,
  "data": {
    "wards": [
      {
        "id": "f1c00f90-ec55-497b-b9ef-e07b3c417214",
        "ward_code": "WARD-ICU-01",
        "ward_name": "Cardiac Intensive Care Unit",
        "ward_type": "ICU",
        "gender_restriction": "MIXED",
        "floor_id": "39ff007e-1daf-4c5f-87a7-be8f712a2e46",
        "block_id": "4caf45de-b989-4c1c-b82d-b7be15e99ebd",
        "capacity": 10,
        "total_beds": 10,
        "occupied_beds": 4,
        "reserved_beds": 2,
        "available_beds": 4,
        "is_active": true
      }
    ],
    "total": 1,
    "page": 1,
    "page_size": 50,
    "total_pages": 1
  }
}
```

---

### 2. GET `/ipd/wards/{ward_id}`

Gets detailed status and bed count metrics for a single ward.

* **Method:** `GET`
* **Path Parameters:** `ward_id` (`UUID`, **Mandatory**)

---

### 3. PATCH `/ipd/wards/{ward_id}`

Updates ward attributes.

* **Method:** `PATCH`
* **Path Parameters:** `ward_id` (`UUID`, **Mandatory**)

---

### 4. POST `/ipd/wards/{ward_id}/block`

Toggles active status of a ward (Block / Unblock).

* **Method:** `POST`
* **Path Parameters:** `ward_id` (`UUID`, **Mandatory**)

---

### 5. POST `/ipd/wards/{ward_id}/maintenance`

Marks all beds in a ward under maintenance status.

* **Method:** `POST`
* **Path Parameters:** `ward_id` (`UUID`, **Mandatory**)

---

## 5. Beds API Specification

### 1. GET `/ipd/beds`

Lists beds with state and filtering options.

* **Method:** `GET`
* **Path:** `/ipd/beds`
* **Query Parameters:**

| Parameter | Type | Required? | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| `ward_id` | `UUID` | Optional | `null` | Filter beds in a specific ward. |
| `status` | `String` | Optional | `null` | `AVAILABLE`, `RESERVED`, `OCCUPIED`, `MAINTENANCE`, `CLEANING`. |
| `bed_type` | `String` | Optional | `null` | `STANDARD`, `ICU`, `VENTILATOR`, `ELECTRIC`, `ISOLATION`. |

#### Response Envelope (`200 OK`)
```json
{
  "success": true,
  "data": {
    "beds": [
      {
        "id": "859c4316-ee95-4674-b555-5da98729b637",
        "ward_id": "f1c00f90-ec55-497b-b9ef-e07b3c417214",
        "ward_name": "Cardiac Intensive Care Unit",
        "bed_number": "ICU-BED-01",
        "bed_type": "ICU",
        "status": "AVAILABLE",
        "reserved_until": null,
        "is_active": true
      }
    ],
    "total": 1,
    "page": 1,
    "page_size": 50,
    "total_pages": 1
  }
}
```

---

### 2. GET `/ipd/beds/{bed_id}`

Retrieves single bed details.

---

### 3. POST `/ipd/beds`

Creates a new bed inside a ward.

---

### 4. PATCH `/ipd/beds/{bed_id}/status`

Updates bed status manually (`AVAILABLE`, `MAINTENANCE`, `CLEANING`).

---

### 5. POST `/ipd/beds/{bed_id}/release`

Releases an occupied or reserved bed back to `AVAILABLE`.

* **Method:** `POST`
* **Path Parameters:** `bed_id` (`UUID`, **Mandatory**) — e.g. `f2209727-4924-4258-b2a6-75c57fc2a053`

---

### 6. POST `/ipd/beds/transfer`

Transfers a bed from one ward to another.

---

### 7. GET `/ipd/beds/availability`

Gets aggregated bed availability summary.

---

## 6. IPD Split-Stage Registration & Admission API Specification

---

### 1. POST `/ipd/admissions` (Stage 1 - Draft Admission Creation)

Pre-registers a patient into IPD, validates resource availability, reserves the selected bed (`RESERVED` status for a 2-hour hold window), and returns the **FULL ENRICHED ADMISSION OBJECT** with `workflow_state: "DRAFT"` and `admission_status: "PENDING_PAYMENT"`.

* **Method:** `POST`
* **Path:** `/ipd/admissions`

#### Request Body Parameters Breakdown

| Field | Type | Required? | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| `patient_id` | `UUID` | **Mandatory** | — | Real patient UUID in database. |
| `attending_doctor_id` | `UUID` | **Mandatory** | — | Attending doctor UUID in database. |
| `ward_id` | `UUID` | **Mandatory** | — | Target ward UUID. |
| `bed_id` | `UUID` | **Mandatory** | — | Target bed UUID (must belong to ward and be `AVAILABLE`). |
| `admission_reason` | `String` | **Mandatory** | — | Clinical reason for admission (min 3, max 2000 chars). |
| `visit_type` | `String` | Optional | `"IPD"` | Must be `"IPD"`. |
| `department_id` | `UUID` | Optional | `null` | Department UUID. |
| `floor_id` | `UUID` | Optional | `null` | Floor UUID. |
| `block_id` | `UUID` | Optional | `null` | Block UUID. |
| `admission_type` | `String` | Optional | `"ELECTIVE"` | `ELECTIVE`, `EMERGENCY`, `TRANSFER`. |
| `was_referred` | `Boolean` | Optional | `false` | Set `true` if referred by external entity. |
| `referral_details` | `Object` | Optional | `null` | Mandatory if `was_referred` is `true`. |
| `referral_details.referred_by_doctor` | `String` | Optional | `null` | Name of referring doctor. |
| `referral_details.referral_source` | `String` | Optional | `null` | Name of referring hospital / clinic. |
| `referral_details.notes` | `String` | Optional | `null` | Referral notes. |
| `notes` | `String` | Optional | `null` | Receptionist notes. |

#### Real Sample Request Body
```json
{
  "patient_id": "9a2e070c-1966-46a6-9011-80d1afbe88c8",
  "visit_type": "IPD",
  "attending_doctor_id": "ba865601-e3b7-4bfc-9b0e-0706ce894dd5",
  "floor_id": "39ff007e-1daf-4c5f-87a7-be8f712a2e46",
  "block_id": "4caf45de-b989-4c1c-b82d-b7be15e99ebd",
  "ward_id": "ef9978fe-248d-4769-895d-03942bddcb3e",
  "bed_id": "f2209727-4924-4258-b2a6-75c57fc2a053",
  "admission_reason": "Acute Coronary Syndrome - Inpatient Observation & Monitoring",
  "admission_type": "ELECTIVE",
  "was_referred": true,
  "referral_details": {
    "referred_by_doctor": "Dr. Sarah Connor",
    "referral_source": "City Clinic & Diagnostics",
    "notes": "Patient referred for specialized inpatient cardiac management"
  },
  "notes": "Attendant requested window bed"
}
```

#### Enriched Response Envelope (`201 Created`)
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "52b909a4-226a-40c9-92b0-92add8885b3e",
    "admission_id": "52b909a4-226a-40c9-92b0-92add8885b3e",
    "tenant_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "admission_number": null,
    "patient_id": "9a2e070c-1966-46a6-9011-80d1afbe88c8",
    "patient_name": "Arpita Arovita",
    "patient_uhid": "PAT-2026-0106",
    "patient_phone": "9867866997",
    "admission_request_id": null,
    "opd_visit_id": null,
    "emergency_visit_id": null,
    "ward_id": "ef9978fe-248d-4769-895d-03942bddcb3e",
    "ward_name": "Premium Private Ward",
    "ward_type": "GENERAL",
    "bed_id": "f2209727-4924-4258-b2a6-75c57fc2a053",
    "bed_number": "B-PVT-05",
    "bed_type": "STANDARD",
    "attending_doctor_id": "ba865601-e3b7-4bfc-9b0e-0706ce894dd5",
    "attending_doctor_name": "Kiran Patil",
    "department_id": null,
    "department_name": null,
    "admission_type": "ELECTIVE",
    "admission_reason": "Acute Coronary Syndrome - Inpatient Observation & Monitoring",
    "expected_discharge": null,
    "notes": "Attendant requested window bed",
    "status": "PENDING",
    "ipd_status": "ADMITTED",
    "workflow_state": "DRAFT",
    "admission_status": "PENDING_PAYMENT",
    "bed_status": "RESERVED",
    "reserved_until": "2026-07-31T18:54:48.885479+05:30",
    "discharge_type": null,
    "admitted_at": null,
    "actual_discharge_at": null,
    "case_closed_at": null,
    "registered_by": "b1da0859-f222-4a4a-83d6-785b779295fb",
    "created_at": "2026-07-31T16:54:48.885479+05:30",
    "updated_at": "2026-07-31T16:54:48.885479+05:30"
  }
}
```

---

### 2. POST `/ipd/admissions/{id}/confirm` (Stage 2 - Admission Confirmation)

Finalizes a draft admission upon payment verification, credit approval, or insurance pre-authorization. Generates an official admission number (`IP-2607-XXXX`), transitions bed status to `OCCUPIED`, and sets workflow state to `ACTIVE` / `ADMITTED`. Returns the **FULL ENRICHED ADMISSION OBJECT**.

* **Method:** `POST`
* **Path Parameters:** `id` (`UUID`, **Mandatory** — Draft admission ID)

#### Sample Request Body (Immediate Payment):
```json
{
  "payment_workflow": "IMMEDIATE",
  "payment_details": {
    "method": "UPI",
    "amount": "15000.00",
    "transaction_reference": "TXN987654321",
    "receipt_notes": "Advance deposit paid via GPay"
  }
}
```

#### Enriched Response Envelope (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "52b909a4-226a-40c9-92b0-92add8885b3e",
    "admission_id": "52b909a4-226a-40c9-92b0-92add8885b3e",
    "tenant_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "admission_number": "IP-2607-0001",
    "patient_id": "9a2e070c-1966-46a6-9011-80d1afbe88c8",
    "patient_name": "Arpita Arovita",
    "patient_uhid": "PAT-2026-0106",
    "patient_phone": "9867866997",
    "ward_id": "ef9978fe-248d-4769-895d-03942bddcb3e",
    "ward_name": "Premium Private Ward",
    "ward_type": "GENERAL",
    "bed_id": "f2209727-4924-4258-b2a6-75c57fc2a053",
    "bed_number": "B-PVT-05",
    "bed_type": "STANDARD",
    "attending_doctor_id": "ba865601-e3b7-4bfc-9b0e-0706ce894dd5",
    "attending_doctor_name": "Kiran Patil",
    "admission_type": "ELECTIVE",
    "admission_reason": "Acute Coronary Syndrome - Inpatient Observation & Monitoring",
    "status": "ADMITTED",
    "ipd_status": "UNDER_TREATMENT",
    "workflow_state": "ACTIVE",
    "admission_status": "ADMITTED",
    "bed_status": "OCCUPIED",
    "confirmed_at": "2026-07-31T17:00:00Z",
    "receipt_id": "RCPT-2026-00001",
    "approving_staff_id": null
  }
}
```

---

### 3. POST `/ipd/admissions/{id}/cancel` (Stage 3 - Draft Cancellation)

Cancels a draft admission and releases bed lock back to `AVAILABLE`.

---

### 4. GET `/ipd/admissions` (List Admissions)

Retrieves a paginated list of IPD admissions.

---

### 5. GET `/ipd/admissions/{id}` (Get Admission Detail)

Retrieves full details of a specific IPD admission.
