# HMS — Billing Service API Documentation

This document covers all payment collection, invoice generation, billing packages, and inpatient admission clearance workflow endpoints for the HMS **Billing Service**.

---

## 1. Global Conventions

### Base URL
All requests are sent to the Billing Service API Gateway stage:
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

## 2. Inpatient (IPD) Billing Clearance

IPD discharge requires that the patient has cleared all financial outstanding balances. This clearance is queried by the IPD service during discharge checks.

### 2.1 Get Billing Clearance Details
Checks if an admitted patient has any outstanding balances preventing discharge.

* **Endpoint:** `GET /billing/clearance/{admission_id}`
* **Required Permission:** `billing:clearance:view`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "admission_id": "admit-uuid-1111",
    "patient_id": "patient-uuid-2222",
    "patient_name": "Ravi Kumar",
    "total_charges": 45000.00,
    "total_paid": 45000.00,
    "outstanding_balance": 0.00,
    "is_cleared": true,
    "cleared_at": "2026-06-23T12:00:00Z",
    "clearance_notes": "All payments settled via UPI."
  }
}
```

---

### 2.2 Manual Billing Clearance Override
Used by the billing desk to manually override and force-clear a patient's billing status (e.g. for charity cases, corporate credit guarantees, or emergency exemptions).

* **Endpoint:** `POST /billing/clearance/{admission_id}/clear`
* **Required Permission:** `billing:clearance:manage` (authorized roles: `ADM-001`/Admin, `ACC-001`/Accountant)
* **Request Body:**
  | Field | Type | Required? | Description |
  | :--- | :--- | :--- | :--- |
  | `clearance_type` | String | **Mandatory** | Reason category (`CREDIT_GUARANTEE`, `CHARITY`, `EMPLOYEE_BENEFIT`, `FORCE_MAJEURE`, `SETTLED`) |
  | `approved_by` | UUID | **Mandatory** | Admin staff user UUID authorizing the clearance |
  | `notes` | String | Optional | Authorized overrides detail notes |

* **Example Request:**
```json
{
  "clearance_type": "CREDIT_GUARANTEE",
  "approved_by": "admin-uuid-9999",
  "notes": "Approved by MD. Corporate billing will settle post-discharge."
}
```
* **Example Response (200 OK):**
```json
{
  "success": true,
  "message": "Billing clearance status forced successfully",
  "data": {
    "admission_id": "admit-uuid-1111",
    "is_cleared": true,
    "outstanding_balance": 15000.00,
    "clearance_type": "CREDIT_GUARANTEE",
    "notes": "Approved by MD. Corporate billing will settle post-discharge."
  }
}
```

---

### 2.3 Update Clearance Details
* **Endpoint:** `PATCH /billing/clearance/{admission_id}`
* **Request Body:**
```json
{
  "notes": "Additional corporate billing vouchers attached."
}
```

---

## 3. Bill Collection & Payments

### 3.1 Collect Bill / Payment
Post a payment collection event.

* **Endpoint:** `POST /billing/collect`
* **Required Permission:** `billing:payment:collect`
* **Request Body:**
  | Field | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `patient_id` | UUID | **Mandatory** | Patient identifier | |
  | `invoice_id` | UUID | Optional | Invoice link | |
  | `payment_mode` | String | **Mandatory** | Payment channel | `CASH`, `UPI`, `CARD`, `INSURANCE_TPA`, `NET_BANKING` |
  | `amount` | Decimal | **Mandatory** | Paid amount value | Must be > 0.00 |
  | `reference_no` | String | Optional | Txn ID / Cheque number | Max 100 chars |
  | `payment_date` | DateTime | Optional | Timestamp | Defaults to current time |

* **Example Request:**
```json
{
  "patient_id": "patient-uuid-2222",
  "invoice_id": "invoice-uuid-5555",
  "payment_mode": "UPI",
  "amount": 2500.00,
  "reference_no": "UPI-83810294"
}
```
* **Example Response (201 Created):**
```json
{
  "success": true,
  "message": "Payment collected successfully",
  "data": {
    "receipt_id": "rec-uuid-8877",
    "patient_id": "patient-uuid-2222",
    "amount": 2500.00,
    "payment_mode": "UPI",
    "status": "COMPLETED",
    "reference_no": "UPI-83810294",
    "collected_by": "user-uuid-receptionist",
    "created_at": "2026-06-23T18:10:00Z"
  }
}
```

---

## 4. Invoices & Billing Items

### 4.1 Generate Invoice
* **Endpoint:** `POST /billing/invoices`
* **Request Body:**
```json
{
  "patient_id": "patient-uuid-2222",
  "admission_id": "admit-uuid-1111",
  "items": [
    {
      "item_code": "CHG-ROOM-GEN",
      "quantity": 3,
      "unit_price": 1500.00,
      "description": "General ward room charge - 3 days"
    },
    {
      "item_code": "CHG-CONS-GEN",
      "quantity": 1,
      "unit_price": 500.00,
      "description": "Doctor OPD consultation"
    }
  ]
}
```
* **Success Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "invoice_id": "invoice-uuid-5555",
    "invoice_number": "INV-2026-0924",
    "total_amount": 5000.00,
    "tax_amount": 0.00,
    "net_payable": 5000.00,
    "payment_status": "UNPAID"
  }
}
```

---

### 4.2 Get Invoice Details
* **Endpoint:** `GET /billing/invoices/{invoice_id}`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "invoice_id": "invoice-uuid-5555",
    "invoice_number": "INV-2026-0924",
    "total_amount": 5000.00,
    "payment_status": "PAID",
    "items": [ ... ]
  }
}
```
