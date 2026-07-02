# HMS — Diagnostics Service API Documentation

This document covers all diagnostic order creation, laboratory/imaging test catalog listing, order cancellations, amendments, audit trails, and version histories for the HMS **Diagnostics Service**.

---

## 1. Global Conventions

### Base URL
All requests are sent to the Diagnostics Service API Gateway stage:
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

## 2. Diagnostic Orders Management

Diagnostics orders record requests for lab analysis (blood, urine, etc.) or radiology scans (X-ray, CT, MRI, etc.).

### 2.1 Create Diagnostic Order
* **Endpoint:** `POST /diagnostic-orders`
* **Request Body:**
  | Field | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `patient_id` | UUID | **Mandatory** | Patient identifier | |
  | `doctor_id` | UUID | **Mandatory** | Ordering doctor identifier | |
  | `order_type` | String | **Mandatory** | Diagnostic category | `LAB`, `RADIOLOGY`, `CARDIOLOGY`, `PATHOLOGY` |
  | `priority` | String | Optional | Urgency indicator | `ROUTINE`, `URGENT`, `STAT` (defaults to `ROUTINE`) |
  | `context_type` | String | **Mandatory** | Encounter class | e.g. `OPD_VISIT`, `IPD_ADMISSION` |
  | `context_id` | UUID | **Mandatory** | Context UUID | |
  | `source_module` | String | **Mandatory** | e.g. `OPD`, `IPD` |
  | `encounter_type`| String | **Mandatory** | e.g. `WARD_ROUND`, `CONSULTATION` |
  | `clinical_notes`| String | Optional | Special clinical instructions | |
  | `lab_items` | Array | Optional | List of ordered lab tests | Mandatory if `order_type` is `LAB` or `PATHOLOGY` |
  | `imaging_items` | Array | Optional | List of ordered scans | Mandatory if `order_type` is `RADIOLOGY` or `CARDIOLOGY` |

* **LabTestOrderItem structure:**
  * `test_id` (UUID, Optional): catalogue test identifier. (Mandatory if `test_name` not supplied).
  * `test_name` (String, Optional): scan/test name.
  * `notes` (String, Optional): specific instructions.

* **ImagingTestOrderItem structure:**
  * `scan_type` (String, Mandatory): scan modality (e.g. `XRAY`, `CT`, `MRI`).
  * `body_part` (String, Mandatory): scan site (e.g. `Chest`, `Abdomen`).
  * `laterality` (String, Optional): left/right indicator.
  * `contrast_required` (Boolean, Optional): defaults to `false`.
  * `notes` (String, Optional): prep notes.

* **Example Request:**
```json
{
  "patient_id": "patient-uuid-1111",
  "doctor_id": "doc-uuid-2222",
  "order_type": "LAB",
  "priority": "URGENT",
  "context_type": "OPD_VISIT",
  "context_id": "visit-uuid-3333",
  "source_module": "OPD",
  "encounter_type": "CONSULTATION",
  "clinical_notes": "Patient fasting since 10 PM yesterday",
  "lab_items": [
    {
      "test_name": "Fasting Blood Sugar (FBS)",
      "notes": "Urgent review required"
    }
  ]
}
```
* **Example Response (201 Created):**
```json
{
  "success": true,
  "message": "Diagnostic order created successfully",
  "data": {
    "id": "order-uuid-4444",
    "patient_id": "patient-uuid-1111",
    "doctor_id": "doc-uuid-2222",
    "order_type": "LAB",
    "status": "ORDERED",
    "version_no": 1,
    "lab_items": [
      {
        "id": "item-uuid-5555",
        "test_name": "Fasting Blood Sugar (FBS)",
        "status": "ORDERED"
      }
    ],
    "created_at": "2026-06-23T18:00:00Z"
  }
}
```

---

### 2.2 Retrieve Order details
* **Endpoint:** `GET /diagnostic-orders/{order_id}`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "order-uuid-4444",
    "status": "ORDERED",
    "version_no": 1,
    "lab_items": [ ... ],
    "imaging_items": []
  }
}
```

---

### 2.3 Update Order Details
Modifies priority, clinical notes, or updates items list on active orders.

* **Endpoint:** `PUT /diagnostic-orders/{order_id}`
* **Request Body:**
  | Field | Type | Required? | Description |
  | :--- | :--- | :--- | :--- |
  | `change_reason` | String | **Mandatory** | Audit change details (min 3, max 500 chars) |
  | `priority` | String | Optional | Update priority |
  | `lab_items` | Array | Optional | New test list |

* **Example Request:**
```json
{
  "change_reason": "Adding PPBS to check alongside FBS",
  "lab_items": [
    {
      "test_name": "Fasting Blood Sugar (FBS)"
    },
    {
      "test_name": "Post-Prandial Blood Sugar (PPBS)"
    }
  ]
}
```

---

### 2.4 Cancel Order
* **Endpoint:** `POST /diagnostic-orders/{order_id}/cancel`
* **Request Body:**
```json
{
  "reason": "Patient discharged before tests performed"
}
```

---

### 2.5 Amend Order Items
Logs a formal amendment request (e.g. adding contrast, adding complementary tests) auditing who and why requested it.

* **Endpoint:** `POST /diagnostic-orders/{order_id}/amend`
* **Request Body:**
```json
{
  "reason": "Radiologist recommended contrast enhancement",
  "imaging_items": [
    {
      "scan_type": "CT",
      "body_part": "Abdomen",
      "contrast_required": true
    }
  ]
}
```
* **Success Response (201 Created):**
```json
{
  "success": true,
  "message": "Amendment request submitted successfully"
}
```

---

## 3. Test Catalogue Lookups

### 3.1 List Diagnostic Tests
* **Endpoint:** `GET /diagnostic-orders/tests`
* **Query Parameters:**
  * `type` (optional): Filter tests catalog by category (`lab` or `radiology`)
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": "test-cat-101",
      "name": "Complete Blood Count",
      "type": "LAB",
      "code": "CBC"
    }
  ]
}
```

---

## 4. Audits & History Versioning

### 4.1 Get Order Event History
* **Endpoint:** `GET /diagnostic-orders/{order_id}/history`

---

### 4.2 Get Order Versions History
* **Endpoint:** `GET /diagnostic-orders/{order_id}/versions`
```json
{
  "success": true,
  "data": [
    {
      "version_no": 1,
      "updated_at": "2026-06-23T18:00:00Z"
    }
  ]
}
```
