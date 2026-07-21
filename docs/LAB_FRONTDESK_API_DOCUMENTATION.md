# Laboratory Front Desk — API Documentation

> **For**: Front Desk / Reception Staff  
> **Module**: Lab Registration & Patient Billing  
> **Base URL**: `https://api.arovita.com/diagnostic-orders`  
> **Last Updated**: 2026-07-16  
> **Permission Required**: `diagnostics:order:view`

---

## Table of Contents

1. [Authentication & Headers](#1-authentication--headers)
2. [Response Format](#2-response-format)
3. [Workflow Overview](#3-workflow-overview)
4. [API Endpoints](#4-api-endpoints)
   - [Search Patients](#41-search-patients)
   - [Get Registration Dropdowns](#42-get-registration-dropdowns)
   - [List Available Tests](#43-list-available-tests)
   - [Create Registration (New Order)](#44-create-registration-new-order)
   - [Get Registration Detail](#45-get-registration-detail)
   - [Get Registration Billing](#46-get-registration-billing)
   - [Process Payment](#47-process-payment)
   - [Download Lab Card (PDF)](#48-download-lab-card-pdf)
   - [List Orders (Worklist)](#49-list-orders-worklist)
   - [Get Order Detail](#410-get-order-detail)
   - [List Samples](#411-list-samples)
   - [List Billing Invoices](#412-list-billing-invoices)
5. [Lookup Endpoint Reference](#5-lookup-endpoint-reference)
6. [Error Responses](#6-error-responses)

---

## 1. Authentication & Headers

| Header | Required | Description |
| :--- | :---: | :--- |
| `Authorization` | ✅ | `Bearer <JWT_TOKEN>` |
| `X-Tenant-Id` | ✅ | Branch/Tenant UUID |
| `Content-Type` | ✅ | `application/json` |

---

## 2. Response Format

### Success
```json
{
  "success": true,
  "code": 200,
  "data": { },
  "message": "optional message"
}
```

### Error
```json
{
  "success": false,
  "code": 400,
  "message": "Error description",
  "errors": []
}
```

---

## 3. Workflow Overview

This is the typical front desk workflow for registering a patient for lab tests:

```
Step 1: Search Patient       →  GET  /lookup?type=dropdowns  (load form options)
        ↓                       GET  /orders?search=<name>   (check existing)
Step 2: Search / Select      →  GET  /orders?q=<name>        (find patient)
        Patient
        ↓
Step 3: Select Tests         →  GET  /lookup?type=tests      (browse test catalog)
        ↓
Step 4: Create Registration  →  POST /orders                 (register patient + tests)
        ↓
Step 5: Get Billing          →  GET  /orders/{id}            (get registration detail)
        ↓                       (billing is auto-generated with registration)
Step 6: Collect Payment      →  POST /orders/{id}/actions    (action: "payment")
        ↓
Step 7: Print Lab Card       →  GET  /orders/{id}            (get order card PDF)
```

---

## 4. API Endpoints

### 4.1 Search Patients

`GET /diagnostic-orders/orders`

Search existing patients by name, UHID, or phone number.

**Query Parameters:**

| Param | Type | Description |
| :--- | :--- | :--- |
| `search` | string | Patient name, UHID, or phone |
| `page` | integer | Page number (default: 1) |
| `limit` | integer | Items per page (default: 10) |

**Response:**
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "id": "8404356a-9d8d-494b-bd29-ddc76b0fb58b",
      "uhid": "PAT-2026-0045",
      "full_name": "Aditya",
      "phone": "+91951383663",
      "dob": null,
      "age": 29,
      "gender": "MALE",
      "blood_group": "O+"
    }
  ]
}
```

---

### 4.2 Get Registration Dropdowns

`GET /diagnostic-orders/lookup`

or

`GET /diagnostic-orders/lookup?type=dropdowns`

Load all dropdown options needed for the registration form.

**Response:**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "visit_types": [
      "Walk-in",
      "OPD Referral",
      "IPD Referral",
      "Emergency"
    ],
    "priorities": [
      "Routine",
      "Urgent",
      "Stat"
    ],
    "sample_types": [
      "Whole Blood",
      "Serum",
      "Plasma",
      "Urine",
      "Swab"
    ],
    "departments": [
      {
        "id": "46d19155-09a1-462e-b256-ce2a5741b091",
        "name": "General Medicine"
      },
      {
        "id": "cab7cc4e-3ec7-49d9-a393-bbdb93bf58c2",
        "name": "Pediatrics"
      }
    ]
  }
}
```

---

### 4.3 List Available Tests

`GET /diagnostic-orders/lookup?type=tests`

Browse the lab test catalog. Use this to populate the test selection in the registration form.

**Query Parameters:**

| Param | Type | Description |
| :--- | :--- | :--- |
| `search` | string | Search by test name or code |
| `department` | string | Filter by department |
| `category` | string | Filter by category |
| `page` | integer | Page number (default: 1) |
| `limit` | integer | Items per page (default: 10) |

**Response:**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "testId": "52949dc3-e8b3-4abc-8d5d-c95b00337f6c",
        "code": "URN-003",
        "testName": "24-Hour Urine Protein",
        "department": "Nephrology",
        "referenceRange": "<150",
        "sampleType": "URINE",
        "containerType": "Vacutainer",
        "tat": "24 hrs",
        "price": 0.0,
        "status": "Active",
        "lastUpdated": "2026-06-23T11:25:21.283249+05:30"
      }
    ],
    "total": 263
  }
}
```

---

### 4.4 Create Registration (New Order)

`POST /diagnostic-orders/orders`

Register a patient for lab tests. This creates the diagnostic order and auto-generates billing.

**Required Fields:**

| Field | Type | Required | Description |
| :--- | :--- | :---: | :--- |
| `patient_id` | UUID | ✅ | Patient UUID (from search) |
| `doctor_id` | UUID | ✅ | Referring doctor UUID |
| `department_id` | UUID | ✅ | Department UUID (from dropdowns) |
| `order_type` | string | ✅ | Must be `"LAB"` for lab orders |
| `context_type` | string | ✅ | `"OPD"`, `"IPD"`, or `"EMERGENCY"` |
| `context_id` | UUID | ✅ | Visit/encounter UUID |
| `source_module` | string | ✅ | `"OPD"`, `"WARD"`, etc. |
| `encounter_type` | string | ✅ | `"WALK_IN"`, `"REFERRAL"` |
| `lab_items` | array | ✅ | Array of test objects |

**Optional Fields:**

| Field | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `priority` | string | `"ROUTINE"` | `"ROUTINE"`, `"URGENT"`, `"STAT"` |
| `clinical_notes` | string | null | Doctor's clinical notes |
| `visit_number` | string | null | Visit reference number |
| `referral_code` | string | null | Referral code if applicable |

**`lab_items[]` object:**

| Field | Type | Required | Description |
| :--- | :--- | :---: | :--- |
| `test_id` | UUID | ✅* | Test UUID from catalog. Either `test_id` or `test_name` is required. |
| `test_name` | string | ✅* | Test name. Either `test_id` or `test_name` is required. |
| `notes` | string | — | Per-test notes |
| `fasting_required` | boolean | — | Default `false` |
| `sample_type` | string | — | Override sample type |

**Request Example:**
```json
{
  "patient_id": "8404356a-9d8d-494b-bd29-ddc76b0fb58b",
  "doctor_id": "cab7cc4e-3ec7-49d9-a393-bbdb93bf58c2",
  "department_id": "46d19155-09a1-462e-b256-ce2a5741b091",
  "order_type": "LAB",
  "priority": "ROUTINE",
  "context_type": "OPD",
  "context_id": "46d19155-09a1-462e-b256-ce2a5741b091",
  "source_module": "OPD",
  "encounter_type": "WALK_IN",
  "lab_items": [
    {
      "test_id": "52949dc3-e8b3-4abc-8d5d-c95b00337f6c",
      "notes": "Fasting sample required",
      "fasting_required": true
    }
  ]
}
```

**Success Response:** `201 Created`
```json
{
  "success": true,
  "code": 201,
  "message": "Registration completed successfully",
  "data": {
    "order_id": "uuid",
    "order_number": "ORD-2026-XXXXX",
    "patient": { "...patient details..." },
    "items": [ "...registered test items with barcodes..." ],
    "billing": { "...auto-generated invoice..." }
  }
}
```

---

### 4.5 Get Registration Detail

`GET /diagnostic-orders/orders/{id}`

Get the full detail of a registration/order.

**Path Parameters:**

| Param | Type | Required |
| :--- | :--- | :---: |
| `id` | UUID | ✅ |

**Response:**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "order_id": "2e737ce4-d61b-4ed6-8795-b38a902e3608",
    "order_item_id": "04440985-c6d9-421c-99be-7c8a91c481bf",
    "patient": {
      "id": "8404356a-9d8d-494b-bd29-ddc76b0fb58b",
      "name": "Aditya",
      "uhid": "PAT-2026-0045",
      "age": null,
      "gender": "MALE",
      "phone": "+91951383663"
    },
    "test": {
      "name": "24-Hour Urine Protein",
      "code": "URN-003"
    },
    "barcode": "BC-10019",
    "collector": null,
    "priority": "ROUTINE",
    "created_at": "2026-07-13T14:03:48.018865+05:30",
    "current_status": "NEW"
  }
}
```

---

### 4.6 Get Registration Billing

`GET /diagnostic-orders/billing/invoices`

List all billing invoices. Filter by patient or status.

**Query Parameters:**

| Param | Type | Description |
| :--- | :--- | :--- |
| `search` | string | Search by patient name, UHID, invoice number |
| `status` | string | `PAID`, `PENDING`, `PARTIALLY_PAID` |
| `paymentMethod` | string | `CASH`, `UPI`, `CARD`, `INSURANCE` |
| `startDate` | string | Filter from date (YYYY-MM-DD) |
| `endDate` | string | Filter to date (YYYY-MM-DD) |
| `page` | integer | Page number (default: 1) |
| `limit` | integer | Items per page (default: 10) |

**Response:**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "invoice_id": "989e0219-446d-4b99-929a-8857e9dbf6a2",
        "invoice_number": "INV-2026-92233",
        "patient_name": "E2E Simulation Patient",
        "patient_uhid": "PAT-2026-0004",
        "patient_age": 35,
        "patient_gender": "MALE",
        "patient_phone": "+919924826164",
        "rx_id": null,
        "rx_number": "Walk-in",
        "payment_method": "CASH",
        "amount": 0.0,
        "status": "PAID",
        "invoice_date": "2026-07-11"
      }
    ],
    "total": 1920
  }
}
```

---

### 4.7 Process Payment

`POST /diagnostic-orders/billing/invoices/{id}/actions`

Process a payment against a billing invoice.

**Path Parameters:**

| Param | Type | Required |
| :--- | :--- | :---: |
| `id` | UUID | ✅ |

**Request Body:**
```json
{
  "action": "payment",
  "amount": 500.00,
  "payment_method": "CASH",
  "reference_number": "REC-12345",
  "notes": "Cash payment received at counter"
}
```

**Success Response:**
```json
{
  "success": true,
  "code": 200,
  "message": "Payment processed successfully",
  "data": { "...updated invoice details..." }
}
```

---

### 4.8 Download Lab Card (PDF)

This endpoint returns a PDF file — the patient's lab card with barcodes for sample collection.

> **Note:** This returns binary PDF content, not JSON. Handle the response as a file download.

**Endpoint:** `GET /diagnostic-orders/orders/{id}` *(with appropriate Accept header for PDF)*

**Response Headers:**
```
Content-Type: application/pdf
Content-Disposition: attachment; filename=lab_card_{order_id}.pdf
```

---

### 4.9 List Orders (Worklist)

`GET /diagnostic-orders/orders`

View the lab worklist — all registered orders with current status.

**Query Parameters:**

| Param | Type | Description |
| :--- | :--- | :--- |
| `page` | integer | Page number (default: 1) |
| `limit` | integer | Items per page (default: 10) |
| `status` | string | Filter: `NEW`, `COLLECTED`, `PROCESSING`, `COMPLETED` |
| `priority` | string | Filter: `ROUTINE`, `URGENT`, `STAT` |
| `search` | string | Search by patient name, UHID, test name |

**Response:**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "order_id": "2e737ce4-d61b-4ed6-8795-b38a902e3608",
        "order_item_id": "04440985-c6d9-421c-99be-7c8a91c481bf",
        "patient": {
          "id": "8404356a-9d8d-494b-bd29-ddc76b0fb58b",
          "name": "Aditya",
          "uhid": "PAT-2026-0045",
          "age": null,
          "gender": "MALE",
          "phone": "+91951383663"
        },
        "test": {
          "name": "24-Hour Urine Protein",
          "code": "URN-003"
        },
        "barcode": "BC-10019",
        "collector": null,
        "priority": "ROUTINE",
        "created_at": "2026-07-13T14:03:48.018865+05:30",
        "current_status": "NEW"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 10,
      "total": 72,
      "totalPages": 8,
      "hasNext": true,
      "hasPrevious": false
    }
  }
}
```

---

### 4.10 Get Order Detail

`GET /diagnostic-orders/orders/{id}`

Get a single order by UUID. Same as [4.5 Get Registration Detail](#45-get-registration-detail).

---

### 4.11 List Samples

`GET /diagnostic-orders/orders` *(with sample view)*

View sample collection status for tracked specimens.

**Response:**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "sample_id": "2f7b0068-e0ee-46f8-9301-ccdd06660578",
        "barcode": "BC-10020",
        "patient_name": "Aditya",
        "patient_uhid": "PAT-2026-0045",
        "test_name": "Activated Partial Thromboplastin Time (APTT)",
        "machine_name": null,
        "technician_name": null,
        "status": "PENDING",
        "priority": "ROUTINE",
        "created_at": "2026-07-13T14:03:48.018865+05:30",
        "tat_minutes": 4513.82
      }
    ],
    "pagination": { "..." }
  }
}
```

---

### 4.12 List Billing Invoices

`GET /diagnostic-orders/billing/invoices`

Same as [4.6 Get Registration Billing](#46-get-registration-billing). Use `?format=csv` for CSV export.

---

## 5. Lookup Endpoint Reference

All lookups are accessed via `GET /diagnostic-orders/lookup?type=<value>`.

| `type` | Use Case | Response Shape |
| :--- | :--- | :--- |
| `dropdowns` | Registration form options | `{ visit_types, priorities, sample_types, departments }` |
| `tests` | Test catalog for selection | `{ items: [...], total }` |
| `packages` | Test packages/profiles | `{ items: [...], total }` |
| `departments` | Staff departments | `[ { departmentId, departmentName, capacity, ... } ]` |
| `printers` | Label printers | `[ { printerId, printerName, printerIp, status } ]` |

---

## 6. Error Responses

| Code | Meaning | Example |
| :--- | :--- | :--- |
| `400` | Validation error | `"Field 'patient_id' is required in payload"` |
| `400` | Invalid UUID | `"Invalid Order ID format. Must be a valid UUID."` |
| `401` | Missing/invalid token | `"Authentication required"` |
| `403` | Insufficient permission | `"Forbidden — 'diagnostics:order:view' permission required"` |
| `404` | Resource not found | `"Order not found"` |
| `500` | Server error | `"Internal server error"` |

---

## Quick Reference — Front Desk Endpoints

| Step | Method | Endpoint | Purpose |
| :---: | :--- | :--- | :--- |
| 1 | `GET` | `/lookup?type=dropdowns` | Load registration form options |
| 2 | `GET` | `/lookup?type=tests` | Browse test catalog |
| 3 | `GET` | `/orders?search=<name>` | Search existing patients/orders |
| 4 | `POST` | `/orders` | Create new registration |
| 5 | `GET` | `/orders/{id}` | View registration detail |
| 6 | `GET` | `/billing/invoices` | View billing invoices |
| 7 | `POST` | `/billing/invoices/{id}/actions` | Process payment |
| 8 | `GET` | `/orders` | View worklist |
