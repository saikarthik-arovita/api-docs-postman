# HMS — Pharmacy Service API Documentation

This document covers all prescription management, medicine catalogue lookups, cancel orders, history audits, and versioning tracking for the HMS **Pharmacy Service**.

---

## 1. Global Conventions

### Base URL
All requests are sent to the Pharmacy Service API Gateway stage:
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

## 2. Prescriptions Management

Prescriptions record medication directives given by doctors for patients. They can be created under an OPD Visit or an IPD Admission context.

### 2.1 Create Prescription
* **Endpoint:** `POST /pharmacy`
* **Required Permission:** (Inherits token context for doctors / clinical staff)
* **Request Body:**
  | Field | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `patient_id` | UUID | **Mandatory** | Patient identifier | |
  | `doctor_id` | UUID | **Mandatory** | Attending doctor identifier | |
  | `context_type` | String | **Mandatory** | Encounter category | e.g. `OPD_VISIT`, `IPD_ADMISSION` |
  | `context_id` | UUID | **Mandatory** | Context UUID | |
  | `source_module` | String | **Mandatory** | Creating module | e.g. `OPD`, `IPD` |
  | `encounter_type`| String | **Mandatory** | Encounter subclass | e.g. `WARD_ROUND`, `CONSULTATION` |
  | `items` | Array | **Mandatory** | Medicine line items | 1 to 30 items |
  | `allergy_checked`| Boolean | Optional | Allergy checks run toggle | Defaults to `false` |
  | `interaction_checked`| Boolean | Optional | Contra-indication toggle | Defaults to `false` |
  | `generic_suggested`| Boolean | Optional | Alternate generic toggle | Defaults to `false` |
  | `valid_till` | Date | Optional | Expiration limit | `YYYY-MM-DD` |
  | `notes` | String | Optional | Prescription notes | |

* **PrescriptionItemCreate Sub-Object Structure:**
  * `medicine_id` (UUID, Optional): ID from medicines catalog. (Mandatory if `medicine_name` not supplied).
  * `medicine_name` (String, Optional): Drug name (min 1, max 200 chars).
  * `generic_name` (String, Optional): Generic identifier.
  * `route` (String, Mandatory): `ORAL`, `IV`, `IM`, `SC`, `TOPICAL`, `INHALATION`, etc.
  * `dosage` (String, Mandatory): dosage (e.g. `1 tablet`, `5ml`).
  * `frequency` (String, Mandatory): Prescription frequency. Accepts standard database codes (`OD`, `BD`, `TDS`, `QDS`, `SOS`, `STAT`, `HS`, `AC`, `PC`) as well as human-readable strings (e.g. `"Twice daily"`, `"Once daily"`, `"1-0-1"`) which are automatically normalized into valid database codes by the backend.
  * `duration_days` (Integer, Optional): duration between 1 and 365.
  * `duration_label` (String, Optional): max 40 chars.
  * `morning_dose` (Boolean, Optional): defaults to `false`.
  * `afternoon_dose` (Boolean, Optional): defaults to `false`.
  * `evening_dose` (Boolean, Optional): defaults to `false`.
  * `night_dose` (Boolean, Optional): defaults to `false`.
  * `with_food` (Boolean, Optional): defaults to `false`.
  * `instructions` (String, Optional): max 500 chars.

* **Example Request:**
```json
{
  "patient_id": "patient-uuid-1111",
  "doctor_id": "doc-uuid-2222",
  "context_type": "OPD_VISIT",
  "context_id": "visit-uuid-3333",
  "source_module": "OPD",
  "encounter_type": "CONSULTATION",
  "allergy_checked": true,
  "interaction_checked": true,
  "items": [
    {
      "medicine_name": "Amoxicillin 500mg",
      "route": "ORAL",
      "dosage": "1 capsule",
      "frequency": "Three times daily",
      "duration_days": 5,
      "with_food": true
    }
  ]
}
```
* **Example Response (201 Created):**
```json
{
  "success": true,
  "message": "Prescription created successfully",
  "data": {
    "id": "rx-uuid-4444",
    "branch_id": "branch-uuid-5555",
    "patient_id": "patient-uuid-1111",
    "doctor_id": "doc-uuid-2222",
    "status": "SIGNED",
    "version_no": 1,
    "items": [
      {
        "id": "rx-item-uuid-6666",
        "medicine_name": "Amoxicillin 500mg",
        "route": "ORAL",
        "dosage": "1 capsule",
        "frequency": "Three times daily",
        "is_discontinued": false
      }
    ],
    "created_at": "2026-06-23T17:55:00Z"
  }
}
```

---

### 2.2 Retrieve Prescription details
* **Endpoint:** `GET /pharmacy/{prescription_id}`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "rx-uuid-4444",
    "status": "SIGNED",
    "version_no": 1,
    "items": [ ... ]
  }
}
```

---

### 2.3 Update Prescription (Prescribing New Version)
Replaces active prescription items or updates headers. Keeps previous version in transaction log.

* **Endpoint:** `PUT /pharmacy/{prescription_id}`
* **Request Body:**
  | Field | Type | Required? | Description |
  | :--- | :--- | :--- | :--- |
  | `change_reason` | String | **Mandatory** | Rationale for update (min 3, max 500 chars) |
  | `allergy_checked` | Boolean | Optional | Update allergy check status |
  | `items` | Array | Optional | New complete list of prescription items |

* **Example Request:**
```json
{
  "change_reason": "Adjusting dosage due to recovery progress",
  "items": [
    {
      "medicine_name": "Amoxicillin 500mg",
      "route": "ORAL",
      "dosage": "1 capsule",
      "frequency": "Twice daily",
      "duration_days": 3
    }
  ]
}
```
* **Example Response (200 OK):**
```json
{
  "success": true,
  "message": "Prescription updated successfully",
  "data": {
    "id": "rx-uuid-4444",
    "status": "SIGNED",
    "version_no": 2,
    "items": [
      {
        "id": "rx-item-uuid-7777",
        "medicine_name": "Amoxicillin 500mg",
        "frequency": "Twice daily"
      }
    ]
  }
}
```

---

### 2.4 Cancel Prescription
Discontinues/invalidates the entire prescription.

* **Endpoint:** `POST /pharmacy/{prescription_id}/cancel`
* **Request Body:**
```json
{
  "reason": "Treatment regime completely discontinued"
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Prescription cancelled successfully",
  "data": {
    "id": "rx-uuid-4444",
    "status": "CANCELLED"
  }
}
```

---

## 3. Medicine Catalogue Lookups

### 3.1 Search Medicines
Retrieves registered medicine items from catalog.

* **Endpoint:** `GET /pharmacy/medicines`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": "med-uuid-001",
      "name": "Paracetamol 650mg",
      "brand": "Calpol",
      "category": "TABLET",
      "price": 15.50,
      "stock_quantity": 500
    }
  ]
}
```

---

## 4. Audits & History Versioning

### 4.1 Get Prescription History
* **Endpoint:** `GET /pharmacy/{prescription_id}/history`

---

### 4.2 Get All Prescription Versions
* **Endpoint:** `GET /pharmacy/{prescription_id}/versions`
