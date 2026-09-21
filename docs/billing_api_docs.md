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
* **Method:** `POST`
* **Required Permission:** `billing:payment:collect`
* **Request Body:**
  | Field | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `invoice_id` | UUID | **Mandatory** | Invoice UUID being paid against | Valid open/partially paid bill |
  | `patient_id` | UUID | Optional | Patient identifier | |
  | `amount` | Decimal | **Mandatory** | Paid amount value | Must be > 0.00 and <= `outstanding` |
  | `payment_mode` | String | **Mandatory** | Payment channel | `UPI`, `CARD`, `CASH`, `OTHER`, `BANK_TRANSFER`, `CHEQUE`, `INSURANCE` |
  | `reference_no` | String | Optional | Transaction / Auth / Cheque number | Optional for `UPI`, `CARD`, `OTHER`; omitted/not required for `CASH` (Max 100 chars) |

* **Validation Rules:**
  - Overpayments are strictly rejected (`422/400 ValidationError`).
  - Terminal bills in status `PAID`, `CANCELLED`, or `REFUNDED` reject payment collections (`409 ConflictError`).
  - Cross-tenant or cross-branch bills are rejected (`403 Forbidden`).

* **Status Transition Lifecycle:**
  - Initial state: `OPEN`
  - When payment < outstanding: status becomes `PARTIALLY_PAID`, `outstanding` is decremented.
  - When remaining balance is settled (outstanding reaches `0.00`): status becomes `PAID`.
  - Each payment inserts a distinct transaction row into `revenue.transactions`.

* **Example Request (Partial Payment via UPI):**
```json
{
  "invoice_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
  "patient_id": "0d2c0b64-c2c3-4d41-9457-4ea2e6d6eb10",
  "amount": 2000.00,
  "payment_mode": "UPI",
  "reference_no": "UPI-987654321"
}
```

* **Example Response (201 Created):**
```json
{
  "success": true,
  "message": "Payment collected successfully",
  "data": {
    "invoice_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "transaction_id": "018e6a1b-7890-7abc-def0-123456789abc",
    "amount_paid": 2000.00,
    "payment_mode": "UPI",
    "invoice_status": "PARTIALLY_PAID",
    "outstanding": 4000.00,
    "paid_at": "2026-09-21T04:30:00Z"
  }
}
```

---

## 4. Invoices & Billing Items

### 4.1 Pending OPD & IPD Bills Query
Retrieve all pending (unpaid / partially paid) bills for a patient.

* **Endpoint:** `GET /billing/bills`
* **Method:** `GET`
* **Required Permission:** `billing:view`
* **Query Parameters:**
  | Parameter | Type | Required? | Description |
  | :--- | :--- | :--- | :--- |
  | `patient_id` | UUID | Optional | Patient UUID. When passed, automatically filters to pending bills. |
  | `visit_type` | String | Optional | Filter by visit type: `OPD` or `IPD`. (Defaults to OPD and IPD when `patient_id` provided). |
  | `status` | String | Optional | Filter by status: `PENDING`, `OPEN`, `PARTIALLY_PAID`, `PAID`, `ALL`. |
  | `pending_only` | Boolean | Optional | `true` enforces pending-only filtering (`outstanding > 0`). |
  | `page` / `per_page` | Integer | Optional | Pagination controls (default page 1, 20 items). |

* **Pending Bills Filtering Rules:**
  - `status IN ('OPEN', 'PARTIALLY_PAID') AND outstanding > 0`
  - Fully paid bills (`PAID`), `CANCELLED`, `REFUNDED`, or bills where `outstanding = 0` are excluded.

* **Example Request:**
```http
GET /billing/bills?patient_id=0d2c0b64-c2c3-4d41-9457-4ea2e6d6eb10&visit_type=IPD
```

* **Example Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "invoice_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
        "invoice_number": "BILL-20260921-0001",
        "visit_type": "IPD",
        "status": "PARTIALLY_PAID",
        "total_amount": 10000.00,
        "paid_amount": 4000.00,
        "outstanding": 6000.00,
        "bill_date": "2026-09-21",
        "patient_id": "0d2c0b64-c2c3-4d41-9457-4ea2e6d6eb10",
        "patient_name": "Ravi Kumar",
        "mrn": "UHID-100234",
        "patient_age": 42,
        "patient_gender": "M"
      }
    ],
    "pagination": {
      "page": 1,
      "per_page": 20,
      "total": 1,
      "pages": 1
    }
  }
}
```

---

### 4.2 Get Invoice Details & Clinical Breakdown
Retrieves full invoice details, line items grouped by clinical billing source, and financial totals.

* **Endpoint:** `GET /billing/invoices/{invoice_id}`
* **Method:** `GET`
* **Required Permission:** `billing:view`
* **Clinical Breakdown Sources Supported:**
  - `BED`: Bed accommodation & nursing charges
  - `SURGERY`: Surgical procedure costs
  - `PROCEDURE`: Minor / major clinical procedures
  - `OT`: Operating theater & anaesthesia charges
  - `LAB`: Diagnostic laboratory tests
  - `IMAGING`: Radiology & imaging scans
  - `PHARMACY`: Prescribed drugs & consumables
  - `OTHER`: Miscellaneous charges

* **Financial Summary Fields:**
  - `subtotal` / `subtotal_amount`: Total gross billable charges
  - `discount` / `discount_amount`: Concessions or discount reductions
  - `tax` / `tax_amount`: GST / applicable taxes
  - `total_amount`: Final payable invoice total
  - `paid_amount`: Cumulative payments collected to date
  - `outstanding`: Current unpaid balance

* **Example Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "invoice_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "invoice_number": "BILL-20260921-0001",
    "status": "PARTIALLY_PAID",
    "visit_type": "IPD",
    "bill_date": "2026-09-21",
    "patient": {
      "id": "0d2c0b64-c2c3-4d41-9457-4ea2e6d6eb10",
      "mrn": "UHID-100234",
      "full_name": "Ravi Kumar",
      "phone": "+91 9876543210"
    },
    "by_source": {
      "BED": {
        "items": [
          {
            "description": "ICU Bed Charges (2 days)",
            "quantity": 2.00,
            "unit_price": 3500.00,
            "amount": 7000.00,
            "discount": 0.00,
            "tax_amount": 0.00
          }
        ],
        "subtotal": 7000.00
      },
      "LAB": {
        "items": [
          {
            "description": "Complete Blood Count (CBC)",
            "quantity": 1.00,
            "unit_price": 500.00,
            "amount": 500.00,
            "discount": 0.00,
            "tax_amount": 0.00
          }
        ],
        "subtotal": 500.00
      },
      "PROCEDURE": {
        "items": [
          {
            "description": "Procedure: Central Line Insertion",
            "quantity": 1.00,
            "unit_price": 2500.00,
            "amount": 2500.00,
            "discount": 0.00,
            "tax_amount": 0.00
          }
        ],
        "subtotal": 2500.00
      }
    },
    "totals": {
      "subtotal": 10000.00,
      "subtotal_amount": 10000.00,
      "discount": 0.00,
      "discount_amount": 0.00,
      "tax": 0.00,
      "tax_amount": 0.00,
      "total_amount": 10000.00,
      "paid_amount": 4000.00,
      "outstanding": 6000.00
    }
  }
}
```

---

### 4.3 Generate or Refresh Invoice
Collects all unbilled line items across OPD/IPD clinical tables and creates or updates an invoice.

* **Endpoint:** `POST /billing/invoices`
* **Method:** `POST`
* **Required Permission:** `billing:create`
* **Request Body:**
```json
{
  "patient_id": "0d2c0b64-c2c3-4d41-9457-4ea2e6d6eb10",
  "visit_type": "IPD",
  "visit_id": "ipd-admission-uuid-1111"
}
```
* **Success Response (201 Created):** Returns full `InvoiceOut` structure.

