# HMS — Billing Service API Documentation

This document covers all payment collection, invoice generation, billing packages, inpatient admission clearance, and patient wallet workflow endpoints for the HMS **Billing Service**.

---

## Table of Contents
1. [Global Conventions](#1-global-conventions)
2. [Inpatient (IPD) Billing Clearance](#2-inpatient-ipd-billing-clearance)
   * 2.1 [Get Billing Clearance Details](#21-get-billing-clearance-details)
   * 2.2 [Manual Billing Clearance Override](#22-manual-billing-clearance-override)
   * 2.3 [Update Clearance Details](#23-update-clearance-details)
3. [Bill Collection & Payments](#3-bill-collection--payments)
   * 3.1 [Collect Bill / Payment (Single & Split Modes)](#31-collect-bill--payment)
4. [Invoices & Billing Items](#4-invoices--billing-items)
   * 4.1 [Pending OPD & IPD Bills Query](#41-pending-opd--ipd-bills-query)
   * 4.2 [Get Invoice Details & Clinical Breakdown](#42-get-invoice-details--clinical-breakdown)
   * 4.3 [Generate or Refresh Invoice (AUTO & CUSTOM Modes)](#43-generate-or-refresh-invoice-auto--custom-modes)
   * 4.4 [Search Billing Catalogue Items](#44-search-billing-catalogue-items)
5. [Patient Wallet & Multi-Component Payments](#5-patient-wallet--multi-component-payments)
   * 5.1 [Collect Payment (Multi-Component Split Payments)](#51-collect-payment-single-mode--multi-component-split-payments)
   * 5.2 [Get Patient Wallet](#52-get-patient-wallet)
   * 5.3 [Get Patient Wallet Ledger](#53-get-patient-wallet-ledger)
   * 5.4 [Credit Patient Wallet](#54-credit-patient-wallet)
   * 5.5 [Debit Patient Wallet](#55-debit-patient-wallet)
   * 5.6 [Reverse Wallet Transaction](#56-reverse-wallet-transaction)
   * 5.7 [Double-Counting Prevention & Concurrency Protection Rules](#57-double-counting-prevention--concurrency-protection-rules)
   * 5.8 [RBAC Permission Reference](#58-rbac-permission-reference)
   * 5.9 [Supported reference_type Taxonomy](#59-supported-reference_type-taxonomy)
6. [Rate Cards & Master Catalogue Management](#6-rate-cards--master-catalogue-management)
   * 6.1 [List & Search Rate Cards](#61-list--search-rate-cards)
   * 6.2 [Create or Upsert Rate Card Item](#62-create-or-upsert-rate-card-item)
   * 6.3 [Update Rate Card Item Price & Status](#63-update-rate-card-item-price--status)
   * 6.4 [Synchronize Master Catalogues](#64-synchronize-master-catalogues)
   * 6.5 [Duplicate Avoidance Architecture & Normalization Rules](#65-duplicate-avoidance-architecture--normalization-rules)
   * 6.6 [Role-Based Access Control (DevOps / Support / SysAdmin Matrix)](#66-role-based-access-control-devops--support--sysadmin-matrix)
   * 6.7 [CLI Inspection & Admin Tool](#67-cli-inspection--admin-tool)

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
  | `patient_id` | UUID | Optional | Patient identifier | Must match invoice patient if provided |
  | `amount` | Decimal | **Mandatory** | Paid amount value | Must be > 0.00 and <= `outstanding` |
  | `payment_mode` | String | Optional | Primary payment channel | `UPI`, `CARD`, `CASH`, `WALLET`, `OTHER`, `BANK_TRANSFER`, `CHEQUE`, `INSURANCE`. Mandatory if `components` omitted. |
  | `reference_no` | String | Optional | Transaction / Auth / Cheque number | Optional for `UPI`, `CARD`, `OTHER`; omitted/not required for `CASH` (Max 100 chars) |
  | `components` | Array | Optional | Multi-component split payment items | Array of `{method, amount, reference_no}`. Sum must equal `amount`. |

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

### 4.3 Generate or Refresh Invoice (AUTO & CUSTOM Modes)
Creates or updates an invoice. Supports both automated clinical item aggregation (`AUTO` mode) and explicit user-specified billing items (`CUSTOM` mode).

* **Endpoint:** `POST /billing/invoices`
* **Method:** `POST`
* **Required Permission:** `billing:create`

#### Modes Overview
| Mode | `invoice_mode` | Description | Visit Requirement |
| :--- | :--- | :--- | :--- |
| **AUTO** (Default) | `"AUTO"` | Gathers unbilled line items across OPD/IPD clinical tables (beds, labs, surgeries, OT, pharmacy). | `visit_type` (`OPD` or `IPD`) and `visit_id` are required. |
| **CUSTOM** | `"CUSTOM"` | Bills strictly the explicit line items provided in the request payload. No clinical table aggregation occurs. | `patient_id` is mandatory; `visit_type` and `visit_id` are optional. |

---

#### A. Custom Invoice Request Format (`invoice_mode: "CUSTOM"`)

##### Request Attributes:
- `patient_id` (*UUID*, Mandatory): The target patient UUID.
- `invoice_mode` (*String*, Optional, default `"AUTO"`): Set to `"CUSTOM"` for custom invoices.
- `visit_type` (*String*, Optional): `"OPD"` or `"IPD"`.
- `visit_id` (*UUID*, Optional): Specific OPD visit or IPD admission UUID.
- `discount` (*Object*, Optional): Overall invoice discount.
  - `type`: `"PERCENTAGE"` or `"FIXED"`
  - `value`: Number (> 0)
  - `reason`: String (e.g. `"Senior Citizen Concession"`)
- `items` (*Array of Objects*, Mandatory for CUSTOM):
  1. **Catalogue Item (`type: "CATALOGUE"`):**
     - `catalogue_item_id` (*UUID*, Mandatory): Valid active rate card UUID from `revenue.rate_cards`.
     - `quantity` (*Decimal*, Mandatory): Quantity (> 0).
     - `unit_price` (*Decimal*, Optional): Manual price override. **Permission Required:** Overriding standard catalogue price requires `is_sysadmin=true`, `SYSTEM_ADMINISTRATOR`, `HOSPITAL_ADMIN`, or `billing:edit` / `billing:price:override` permissions; otherwise rejected with 403 `PermissionDeniedError`.
  2. **Miscellaneous Item (`type: "MISCELLANEOUS"`):**
     - `description` (*String*, Mandatory): Non-empty textual description of service/item.
     - `amount` (*Decimal*, Mandatory): Charge amount (> 0).
     - *Note:* Miscellaneous items are NOT catalogue items and do not require `catalogue_item_id` or category lookup.

##### Restricted Categories:
Custom invoices strictly **reject** items under clinical consultation and registration categories:
- `Registration`
- `Consultation`
- `Doctor Charges`

Attempting to submit items from these categories raises `400 Bad Request (ValidationError)`.

---

#### B. Custom Invoice Request Example

```json
{
  "patient_id": "0d2c0b64-c2c3-4d41-9457-4ea2e6d6eb10",
  "invoice_mode": "CUSTOM",
  "visit_type": "IPD",
  "visit_id": "5a58fc86-4e36-4f6b-a69a-c06cedbe8f46",
  "discount": {
    "type": "PERCENTAGE",
    "value": 10.0,
    "reason": "Hospital Foundation Discount"
  },
  "items": [
    {
      "type": "CATALOGUE",
      "catalogue_item_id": "00fd2c59-8e6a-4eb5-9eb4-1ca8d717a82b",
      "quantity": 2
    },
    {
      "type": "MISCELLANEOUS",
      "description": "Special medical equipment sanitization & handling",
      "amount": 1500.00
    }
  ]
}
```

#### C. Custom Invoice Response Example (201 Created)

```json
{
  "success": true,
  "data": {
    "invoice_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "invoice_number": "BILL-20260922-0042",
    "status": "OPEN",
    "invoice_mode": "CUSTOM",
    "visit_type": "IPD",
    "bill_date": "2026-09-22",
    "patient": {
      "id": "0d2c0b64-c2c3-4d41-9457-4ea2e6d6eb10",
      "mrn": "UHID-100234",
      "full_name": "Ravi Kumar",
      "phone": "+91 9876543210"
    },
    "items": [
      {
        "id": "713da38f-9a05-4f40-97b1-2ff0d48cb912",
        "type": "CATALOGUE",
        "catalogue_item_id": "00fd2c59-8e6a-4eb5-9eb4-1ca8d717a82b",
        "category": "Laboratory",
        "description": "Arterial Blood Gas (ABG)",
        "quantity": 2.0,
        "unit_price": 450.00,
        "amount": 900.00,
        "discount": 90.00,
        "tax_amount": 0.00,
        "total_amount": 810.00
      },
      {
        "id": "842db19a-9b12-4f11-87b3-1ff0d48ca123",
        "type": "MISCELLANEOUS",
        "category": "OTHER",
        "description": "Special medical equipment sanitization & handling",
        "quantity": 1.0,
        "unit_price": 1500.00,
        "amount": 1500.00,
        "discount": 150.00,
        "tax_amount": 0.00,
        "total_amount": 1350.00
      }
    ],
    "totals": {
      "subtotal": 2400.00,
      "subtotal_amount": 2400.00,
      "discount": 240.00,
      "discount_amount": 240.00,
      "tax": 0.00,
      "tax_amount": 0.00,
      "total_amount": 2160.00,
      "paid_amount": 0.00,
      "outstanding": 2160.00
    }
  }
}
```

---

#### D. Legacy AUTO Invoice Request Example (100% Backward Compatible)

```json
{
  "patient_id": "0d2c0b64-c2c3-4d41-9457-4ea2e6d6eb10",
  "visit_type": "OPD",
  "visit_id": "opd-visit-uuid-1111"
}
```

---

### 4.4 Search Billing Catalogue Items
Searches standard billable rate card items for the active tenant branch. Used by custom invoice UIs and rate lookups.

* **Endpoint:** `GET /billing/catalogue/items`
* **Method:** `GET`
* **Required Permission:** `billing:view` or `billing:create`
* **Query Parameters:**
  | Parameter | Type | Required? | Description |
  | :--- | :--- | :--- | :--- |
  | `search` | String | Optional | Search substring across item name, description, and item code. |
  | `category` | String | Optional | Filter by catalogue category (e.g. `Laboratory`, `Equipment`, `Room & Admission`, `Ambulance`, `Pharmacy`). |
  | `is_active` | Boolean | Optional | Filter by active state (`true`/`false`, default `true`). |
  | `page` | Integer | Optional | Page number (default: 1). |
  | `limit` | Integer | Optional | Number of items per page (default: 50, max: 100). |

* **Security & Exclusion Filtering:**
  - Rate items are strictly scoped to the active tenant/branch (`branch_id = tenant_id` or `branch_id IS NULL`).
  - Restricted clinical categories (`Registration`, `Consultation`, `Doctor Charges`) are automatically filtered out from search results.

* **Example Request:**
  ```http
  GET /billing/catalogue/items?search=Blood&page=1&limit=10
  ```

* **Example Response (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "id": "00fd2c59-8e6a-4eb5-9eb4-1ca8d717a82b",
        "code": "BCH-039",
        "name": "Arterial Blood Gas (ABG)",
        "category": "Laboratory",
        "unit": "unit",
        "price": "450.00",
        "is_active": true
      },
      {
        "id": "e5e82d23-96f8-4981-b068-52a12dc10de4",
        "code": "MIC-001",
        "name": "Blood Culture & Sensitivity",
        "category": "Laboratory",
        "unit": "unit",
        "price": "650.00",
        "is_active": true
      }
    ],
    "total": 12,
    "page": 1,
    "limit": 10
  }
}
```

---

## 5. Patient Wallet & Multi-Component Payments

A secure, transactional, audit-trailed **Patient Wallet & Financial Credit Ledger** with support for multi-component split payments and insurance settlement credits.

### 5.1 Collect Payment (Single-Mode & Multi-Component Split Payments)
Records a payment against an open invoice. Supports both legacy single payment modes and split payments (e.g. Wallet + UPI + Cash).

* **Endpoint:** `POST /billing/collect`
* **Method:** `POST`
* **Required Permission:** `billing:payment:collect`

#### A. Multi-Component Split Payment Request
```json
{
  "invoice_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
  "patient_id": "0d2c0b64-c2c3-4d41-9457-4ea2e6d6eb10",
  "amount": 5000.00,
  "components": [
    {
      "method": "WALLET",
      "amount": 2000.00
    },
    {
      "method": "UPI",
      "amount": 2000.00,
      "reference_no": "UPI-TXN-987654"
    },
    {
      "method": "CASH",
      "amount": 1000.00
    }
  ]
}
```

#### B. Legacy Single-Mode Payment Request (100% Backward Compatible)
```json
{
  "invoice_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
  "amount": 2000.00,
  "payment_mode": "UPI",
  "reference_no": "UPI-TXN-123456"
}
```

* **Success Response (201 Created):**
```json
{
  "success": true,
  "message": "Payment collected successfully",
  "data": {
    "invoice_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "transaction_id": "018e652a-923f-7e04-89ac-3b4a2e5c89ad",
    "amount_paid": 5000.00,
    "payment_mode": "OTHER",
    "invoice_status": "PAID",
    "outstanding": 0.00,
    "paid_at": "2026-09-21T13:41:24.710875+05:30",
    "components": [
      {
        "id": "c1f72a4d-1a89-4e02-b2d9-36a5b82c19e4",
        "method": "WALLET",
        "amount": 2000.00,
        "reference_no": null,
        "wallet_transaction_id": "018e652a-912f-7c12-98ab-4d2a1e8c76ad"
      },
      {
        "id": "d2f83b5e-2b9a-4f13-c3ea-47b6c93d20f5",
        "method": "UPI",
        "amount": 2000.00,
        "reference_no": "UPI-TXN-987654",
        "wallet_transaction_id": null
      },
      {
        "id": "e3a94c6f-3ca0-4a24-d4fb-58c7da4e31a6",
        "method": "CASH",
        "amount": 1000.00,
        "reference_no": null,
        "wallet_transaction_id": null
      }
    ]
  }
}
```

---

### 5.2 Get Patient Wallet
Retrieves the patient's logical wallet and current available balance. Automatically creates an active zero-balance wallet if none exists.

* **Endpoint:** `GET /billing/wallet/{patient_id}`
* **Method:** `GET`
* **Required Permission:** `wallet:view` or `billing:view` (accessible by Admin, Billing Executive, and Receptionist)
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "37b3eaca-7fe9-45fa-b616-3ab226b84f5f",
    "patient_id": "0d2c0b64-c2c3-4d41-9457-4ea2e6d6eb10",
    "currency": "INR",
    "available_balance": 25000.00,
    "status": "ACTIVE",
    "created_at": "2026-09-21T13:39:06.921295+05:30",
    "updated_at": "2026-09-21T13:41:24.710875+05:30"
  }
}
```

---

### 5.3 Get Patient Financial Statement & Wallet Ledger
Retrieves a unified, comprehensive **Financial Statement & Payment Ledger** (reflecting as a bank statement) for the patient. It shows all payments done across bills (both **full** and **partial** payments), their payment modes, direct wallet transactions, and total balance summaries.

* **Endpoint:** `GET /billing/wallet/{patient_id}/ledger`
* **Method:** `GET`
* **Required Permission:** `wallet:ledger:view`, `wallet:view`, or `billing:view` (accessible by Admin, Billing Executive, and Receptionist)
* **Query Parameters:**
  | Parameter | Type | Required | Description | Example |
  | :--- | :--- | :--- | :--- | :--- |
  | `payment_mode` | string | No | Filter by payment mode (case-insensitive). Matches single-mode or split component methods. | `CASH`, `UPI`, `WALLET`, `CARD`, `INSURANCE` |
  | `visit_type` | string | No | Filter by visit type (`OPD` or `IPD`, case-insensitive). | `OPD`, `IPD` |
  | `payment_type` | string | No | Filter by payment status category. | `FULL`, `PARTIAL`, `ADVANCE` |
  | `from_date` | string | No | Filter transactions on or after this date (`YYYY-MM-DD`). | `2026-07-01` |
  | `to_date` | string | No | Filter transactions on or before this date (`YYYY-MM-DD`). | `2026-08-31` |
  | `page` | integer | No | Page number for pagination (default: `1`). | `1` |
  | `per_page` | integer | No | Items per page (default: `20`, max: `100`). | `20` |

* **Success Response (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "wallet": {
      "id": "f20caf38-438f-4700-a0cd-68dd068b55f3",
      "patient_id": "8b5d7493-e9e9-43e5-99a4-e0467993911d",
      "currency": "INR",
      "available_balance": 0.00,
      "status": "ACTIVE",
      "created_at": "2026-09-21T17:35:53.452400+05:30",
      "updated_at": "2026-09-21T17:35:53.452400+05:30"
    },
    "summary": {
      "total_billed": 94950.00,
      "total_paid": 12450.00,
      "total_outstanding": 82518.00,
      "wallet_balance": 0.00
    },
    "transactions": [
      {
        "id": "b1e454bc-21e0-4d7f-b3bb-90fd6b554166",
        "wallet_id": "f20caf38-438f-4700-a0cd-68dd068b55f3",
        "patient_id": "8b5d7493-e9e9-43e5-99a4-e0467993911d",
        "transaction_type": "PAYMENT",
        "payment_type": "FULL",
        "payment_mode": "CASH",
        "amount": 11200.00,
        "narration": "IPD Payment - BILL-20260814-0001 (Full Payment)",
        "invoice_number": "BILL-20260814-0001",
        "bill_id": "95f27813-185e-4899-91c1-8b85f110a751",
        "visit_type": "IPD",
        "bill_total": 41600.00,
        "bill_paid": 11200.00,
        "bill_outstanding": 30400.00,
        "bill_status": "PAID",
        "status": "SUCCESS",
        "reference_no": null,
        "components": null,
        "created_at": "2026-08-14T06:34:50.916507+05:30",
        "collected_at": "2026-08-14T06:34:50.916507+05:30"
      },
      {
        "id": "2c369f2b-9a70-4bc9-838c-02c376786558",
        "wallet_id": "f20caf38-438f-4700-a0cd-68dd068b55f3",
        "patient_id": "8b5d7493-e9e9-43e5-99a4-e0467993911d",
        "transaction_type": "PAYMENT",
        "payment_type": "PARTIAL",
        "payment_mode": "INSURANCE",
        "amount": 250.00,
        "narration": "OPD Payment - BILL-20260729-0021 (Partial Payment)",
        "invoice_number": "BILL-20260729-0021",
        "bill_id": "6362d743-a728-431f-9946-c813edc9fde7",
        "visit_type": "OPD",
        "bill_total": 500.00,
        "bill_paid": 250.00,
        "bill_outstanding": 250.00,
        "bill_status": "PARTIALLY_PAID",
        "status": "SUCCESS",
        "reference_no": null,
        "components": null,
        "created_at": "2026-07-30T06:58:16.392177+05:30",
        "collected_at": "2026-07-30T06:58:16.392177+05:30"
      }
    ],
    "total_count": 4,
    "page": 1,
    "per_page": 20,
    "total_pages": 1
  }
}
```

---

### 5.4 Credit Patient Wallet
Credits approved funds (insurance settlement, cash advance, refund, or adjustment) into a patient's wallet.

* **Endpoint:** `POST /billing/wallet/credit`
* **Method:** `POST`
* **Required Permission:** `wallet:credit`
* **Request Body:**
```json
{
  "patient_id": "0d2c0b64-c2c3-4d41-9457-4ea2e6d6eb10",
  "amount": 50000.00,
  "credit_source": "INSURANCE",
  "reference_type": "INSURANCE_CLAIM",
  "reference_id": "CLAIM-776655",
  "idempotency_key": "IDEMP-INS-776655",
  "notes": "Cashless pre-auth settled by Star Health",
  "ipd_id": "ipd-admission-uuid-1111"
}
```
* **Success Response (201 Created):** Returns `WalletTransactionOut`.

---

### 5.5 Debit Patient Wallet
Direct internal debit for hospital deductions or manual balance adjustments.

* **Endpoint:** `POST /billing/wallet/debit`
* **Method:** `POST`
* **Required Permission:** `wallet:debit`
* **Request Body:**
```json
{
  "patient_id": "0d2c0b64-c2c3-4d41-9457-4ea2e6d6eb10",
  "amount": 500.00,
  "reference_type": "DUE_ADJUSTMENT",
  "notes": "Consumables charge adjustment"
}
```
* **Success Response (200 OK):** Returns `WalletTransactionOut`.

---

### 5.6 Reverse Wallet Transaction
Compensating reversal that restores or deducts funds, strictly preserving ledger immutability.

* **Endpoint:** `POST /billing/wallet/reverse`
* **Method:** `POST`
* **Required Permission:** `wallet:adjust` or `wallet:refund`
* **Request Body:**
```json
{
  "transaction_id": "018e652a-912f-7c12-98ab-4d2a1e8c76ad",
  "reason": "Attending doctor cancelled scheduled procedure"
}
```
* **Success Response (200 OK):** Returns `WalletTransactionOut` with `transaction_type = "REVERSAL"` and `reversal_of = "<original_id>"`.

---

### 5.7 Double-Counting Prevention & Concurrency Protection Rules

1. **Insurance Settlement vs. Direct Bill Offset**:
   * If an insurance claim directly offsets a bill (`revenue.bills.insurance_claim_id`), it is applied as an approved insurance deduction on that bill. In this case, it cannot also be credited into the patient wallet.
   * If an insurance claim settlement is credited to the patient's wallet (`POST /billing/wallet/credit` with `credit_source = 'INSURANCE'` and `reference_type = 'INSURANCE_CLAIM'`), the system records the claim reference.
   * The system prevents double counting by:
     * Checking `revenue.bills` to ensure the claim is not already applied as an invoice deduction.
     * Checking `revenue.wallet_transactions` to ensure the claim ID has not already been credited.
     * Enforcing `idempotency_key` unique constraints to protect against repeated webhook callbacks.

2. **Concurrency & Overdraft Protection**:
   * All wallet debit operations acquire exclusive row-level locks within the database transaction:
     ```sql
     SELECT * FROM revenue.wallets WHERE id = $1 FOR UPDATE;
     ```
   * Simultaneous debit requests (e.g. concurrent checkout threads) are serialized at the database layer.
   * Database check constraint `CHECK (available_balance >= 0)` guarantees that a wallet balance can never become negative.

3. **Ledger Immutability**:
   * Wallet ledger entries in `revenue.wallet_transactions` are append-only and strictly immutable.
   * Updates and physical deletions are prohibited. Corrections are performed via `POST /billing/wallet/reverse`, creating a compensating `REVERSAL` transaction referencing the original transaction ID (`reversal_of`).

---

### 5.8 RBAC Permission Reference

| Permission | Description | Recommended Roles |
| :--- | :--- | :--- |
| `billing:view` | View invoices, bill lists, and dashboard stats | Billing Clerk, Cashier, Receptionist, Admin, Accountant |
| `billing:create` | Generate or refresh patient invoices | Billing Clerk, Admin, Accountant |
| `billing:payment:collect` | Collect bill payments (single-mode or multi-component including Wallet) | Cashier, Billing Clerk, Admin |
| `billing:clearance:view` | View IPD discharge clearance status | Nursing Staff, Ward Incharge, Billing Clerk, Admin |
| `billing:clearance:manage` | Override clearance / issue manual NOC | Admin (`ADM-001`), Accountant (`ACC-001`) |
| `wallet:view` | View patient wallet balance and status | Admin, Billing Manager, Authorized Accounts Staff |
| `wallet:ledger:view` | Inspect full audit ledger of wallet transactions | Admin, Auditor, Billing Manager |
| `wallet:credit` | Credit funds into patient wallet (Insurance/Advances) | Admin, Authorized TPA/Billing Manager |
| `wallet:debit` | Direct internal wallet debit | Admin, Authorized Billing Manager |
| `wallet:adjust` / `wallet:refund` | Reverse or adjust wallet transactions | Admin, Finance Head |

---

### 5.9 Supported `reference_type` Taxonomy

Every entry in `revenue.wallet_transactions` must record a `reference_type` and `reference_id` explaining **where** money came from or **where** it was consumed.

| `reference_type` | Applicable `transaction_type` | Applicable `credit_source` | Description | Expected `reference_id` Content |
| :--- | :--- | :--- | :--- | :--- |
| `INSURANCE_CLAIM` | `CREDIT` | `INSURANCE` | Insurance cashless pre-auth, TPA reimbursement settlement, or approved claim amount | Insurance Claim UUID / TPA Reference ID |
| `BILL_PAYMENT` | `DEBIT` | *(None)* | Wallet funds consumed as part of bill checkout (single or multi-component payment) | Bill / Invoice UUID |
| `ADVANCE_DEPOSIT` / `CASH_ADVANCE` | `CREDIT` | `ADVANCE` | Patient deposits cash, card, or UPI advance payment into wallet | Advance Receipt # / Payment Gateway Reference |
| `REFUND` | `CREDIT` | `REFUND` | Refund from cancelled investigation, procedure, or excess bill payment credited to wallet | Refund Record UUID / Cancellation Order ID |
| `PROCEDURE_CHARGE` | `DEBIT` | *(None)* | Internal procedure cost deducted directly from wallet | Clinical Procedure UUID |
| `SURGERY_CHARGE` | `DEBIT` | *(None)* | Surgery or OT package charge deducted from wallet | Surgery Booking / OT Record UUID |
| `DUE_ADJUSTMENT` | `DEBIT` | *(None)* | Authorized deduction for hospital dues, concession recoveries, or administrative debit adjustments | Adjustment Order # / Approval Reference |
| `BALANCE_ADJUSTMENT` | `CREDIT` | `ADJUSTMENT` | Discretionary concession, billing waiver, rounding balance adjustment, or credit note | Credit Note # / Management Approval ID |
| `OTHER_AUTHORIZED` | `CREDIT` / `DEBIT` | `OTHER_AUTHORIZED_CREDIT` | Government scheme subsidy, corporate tie-up credit, or special authorized credit | Sanction Letter # / Corporate Account ID |
| `REVERSAL` | `REVERSAL` | `ADJUSTMENT` | Compensating reversal of an erroneous prior credit or debit | Target `wallet_transactions.id` (UUID) |

#### Context Linkage Rule:
When `reference_type` is associated with a specific visit context, the corresponding foreign keys must also be populated where available:
* **IPD Admissions:** `ipd_id` pointing to `ipd.ipd_admissions(id)`
* **OPD Visits:** `opd_id` pointing to `clinical.opd_visits(id)`
* **Financial Accounts / Invoices:** `financial_account_id` pointing to `revenue.bills(id)`
* **Clinical Events:** `surgery_id` or `procedure_id`

---

## 6. Rate Cards & Master Catalogue Management

The Rate Card engine centralizes pricing masters for doctor consultations, laboratory tests, pharmacy medicines, room & bed occupancies, surgical procedures, and emergency charges across branches.

### 6.1 List & Search Rate Cards
Queries rate cards with advanced filters, pagination, and enriched doctor details (name, employee code, department).

* **Endpoint:** `GET /billing/catalogue/rate-cards`
* **Method:** `GET`
* **Required Permission:** `billing:view` or `billing:create`
* **Query Parameters:**
  | Parameter | Type | Required? | Description | Example |
  | :--- | :--- | :--- | :--- | :--- |
  | `category` | String | Optional | Filter by category: `CONSULTATION`, `PHARMACY`, `LABORATORY`, `ROOM`, `PROCEDURE`, `EMERGENCY`, `OTHER` | `CONSULTATION` |
  | `sub_category` | String | Optional | Sub-classification (e.g. `OPD`, `IPD`, `GENERAL`, `ICU`) | `OPD` |
  | `doctor_id` | UUID | Optional | Filter doctor consultation rate cards by staff UUID | `d049e6f2-bf83-4927-9ec9-974a6b251f28` |
  | `active` | Boolean | Optional | Filter by active state (`true`/`false`) | `true` |
  | `search` / `q` | String | Optional | Text search across `service_code`, `service_name`, doctor name, and department | `Cardiology` |
  | `page` | Integer | Optional | Page number (default: `1`) | `1` |
  | `limit` / `per_page` | Integer | Optional | Items per page (default: `50`, max: `100`) | `50` |

* **Example Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "total": 42,
    "page": 1,
    "limit": 50,
    "items": [
      {
        "id": "18f2d80d-8302-4ae6-b816-09252c80c213",
        "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
        "service_code": "DOC-CON-DR-SAI-001",
        "service_name": "General OPD Consultation - Dr. Sai Karthik",
        "category": "CONSULTATION",
        "sub_category": "OPD",
        "rate": 500.00,
        "cgst_rate": 0.00,
        "sgst_rate": 0.00,
        "igst_rate": 0.00,
        "is_active": true,
        "doctor_id": "d049e6f2-bf83-4927-9ec9-974a6b251f28",
        "doctor_name": "Dr. Sai Karthik",
        "doctor_employee_code": "DR-SAI-001",
        "doctor_department": "General Medicine"
      }
    ]
  }
}
```

---

### 6.2 Create or Upsert Rate Card Item
Adds a new rate card or idempotently updates an existing item matching `(branch_id, service_code)` or `(branch_id, category, doctor_id)`.

* **Endpoint:** `POST /billing/catalogue/rate-cards`
* **Method:** `POST`
* **Required Roles:** `DEVOPS_ENGINEER` (`ITC-002`), `SUPPORT_ENGINEER` (`ITC-003`), `SYSTEM_ADMINISTRATOR` (`ITC-001`)
* **Request Body:**
  | Field | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `service_code` | String | **Mandatory** | Unique service identifier (normalized to uppercase) | Max 100 chars, e.g. `DOC-CON-DR-SAI` |
  | `service_name` | String | **Mandatory** | Display name of the service | Max 255 chars |
  | `category` | String | **Mandatory** | Service category | `CONSULTATION`, `PHARMACY`, `LABORATORY`, `ROOM`, `PROCEDURE`, `EMERGENCY`, `OTHER` |
  | `sub_category` | String | Optional | Sub-tier classification | e.g. `OPD`, `IPD`, `GENERAL`, `ICU` |
  | `rate` | Decimal | **Mandatory** | Base charge amount (INR) | `>= 0.00` |
  | `cgst_rate` | Decimal | Optional | Central GST percentage | Default `0.00`, `>= 0.00` |
  | `sgst_rate` | Decimal | Optional | State GST percentage | Default `0.00`, `>= 0.00` |
  | `igst_rate` | Decimal | Optional | Integrated GST percentage | Default `0.00`, `>= 0.00` |
  | `doctor_id` | UUID | Optional | Doctor staff user UUID (for consultation charges) | Valid `workforce.staff_profiles` UUID |
  | `is_active` | Boolean | Optional | Active state flag | Default `true` |
  | `metadata` | Object | Optional | Additional properties & tags | JSON object |

* **Example Request:**
```json
{
  "service_code": "DOC-CON-DR-SHARMA",
  "service_name": "Senior Cardiology Consultation",
  "category": "CONSULTATION",
  "sub_category": "OPD",
  "rate": 850.00,
  "cgst_rate": 0.00,
  "sgst_rate": 0.00,
  "igst_rate": 0.00,
  "doctor_id": "d049e6f2-bf83-4927-9ec9-974a6b251f28",
  "is_active": true
}
```

* **Example Response (201 Created):**
```json
{
  "success": true,
  "message": "Rate card item created/upserted successfully",
  "data": {
    "id": "18f2d80d-8302-4ae6-b816-09252c80c213",
    "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "service_code": "DOC-CON-DR-SHARMA",
    "service_name": "Senior Cardiology Consultation",
    "category": "CONSULTATION",
    "sub_category": "OPD",
    "rate": 850.00,
    "cgst_rate": 0.00,
    "sgst_rate": 0.00,
    "igst_rate": 0.00,
    "doctor_id": "d049e6f2-bf83-4927-9ec9-974a6b251f28",
    "is_active": true
  }
}
```

---

### 6.3 Update Rate Card Item Price & Status
Updates the price, GST rates, or active status of an existing rate card item.

* **Endpoint:** `PUT /billing/catalogue/rate-cards/{rate_card_id}`
* **Method:** `PUT` / `PATCH`
* **Required Roles:** `DEVOPS_ENGINEER` (`ITC-002`), `SUPPORT_ENGINEER` (`ITC-003`), `SYSTEM_ADMINISTRATOR` (`ITC-001`)
* **Request Body:**
```json
{
  "rate": 900.00,
  "is_active": true
}
```
* **Example Response (200 OK):**
```json
{
  "success": true,
  "message": "Rate card item updated successfully",
  "data": {
    "id": "18f2d80d-8302-4ae6-b816-09252c80c213",
    "service_code": "DOC-CON-DR-SHARMA",
    "rate": 900.00,
    "is_active": true,
    "updated_at": "2026-09-23T04:30:00Z"
  }
}
```

---

### 6.4 Synchronize Master Catalogues
Automatically reads all Doctors in `workforce.staff_profiles`, Master Lab tests in `laboratory.test_master`, Medicines in `pharmacy.medicines`, and Bed charges in `ipd.rooms` / `ipd.beds`, creating or updating their corresponding rate card items idempotently.

* **Endpoint:** `POST /billing/catalogue/rate-cards/sync`
* **Method:** `POST`
* **Required Roles:** `DEVOPS_ENGINEER` (`ITC-002`), `SUPPORT_ENGINEER` (`ITC-003`), `SYSTEM_ADMINISTRATOR` (`ITC-001`)
* **Example Response (200 OK):**
```json
{
  "success": true,
  "message": "Rate cards synchronized successfully",
  "data": {
    "total_synced": 38,
    "details": {
      "doctors_synced": 4,
      "lab_tests_synced": 18,
      "medicines_synced": 12,
      "rooms_synced": 4
    }
  }
}
```

---

### 6.5 Duplicate Avoidance Architecture & Normalization Rules

1. **Normalized Service Codes:**
   - Leading/trailing whitespace is trimmed.
   - Letters are converted to uppercase.
   - Spaces and special characters are replaced by dashes.
2. **Deterministic Code Conventions:**
   - Doctor Consultation: `DOC-CON-<EMPLOYEE_CODE>` (or `DOC-CON-<UUID_SHORT>`)
   - Lab Tests: `LAB-<TEST_CODE>`
   - Medicines: `MED-<SKU>`
   - Rooms & Beds: `ROOM-<BED_TYPE>`
3. **Natural Composite Keys:**
   - Unique constraint on `(branch_id, service_code)`.
   - Secondary deduplication on `(branch_id, category, doctor_id)` for doctor consultation charges.
4. **Idempotent Upsert Logic:**
   - When a match is found during insertion or synchronization, the SQL layer executes an `ON CONFLICT (branch_id, service_code) DO UPDATE SET rate = EXCLUDED.rate, ...` preventing duplicate entries.

---

### 6.6 Role-Based Access Control (DevOps / Support / SysAdmin Matrix)

| Endpoint | Method | `ITC-001` (SysAdmin) | `ITC-002` (DevOps) | `ITC-003` (Support) | Cashier / Receptionist / Doctor |
|:---|:---:|:---:|:---:|:---:|:---:|
| `GET /billing/catalogue/rate-cards` | GET | Allowed | Allowed | Allowed | Allowed (`billing:view`) |
| `POST /billing/catalogue/rate-cards` | POST | Allowed | Allowed | Allowed | **403 Forbidden** |
| `PUT /billing/catalogue/rate-cards/{id}` | PUT | Allowed | Allowed | Allowed | **403 Forbidden** |
| `POST /billing/catalogue/rate-cards/sync` | POST | Allowed | Allowed | Allowed | **403 Forbidden** |

---

### 6.7 CLI Inspection & Admin Tool

For backend developers, support engineers, and DevOps inspecting rate cards from EC2 bastion hosts or CloudShell:

```bash
cd uat-bootstrap
source .venv/bin/activate

# 1. Summary of rate cards across all categories
python scripts/list_rate_cards_catalogue.py

# 2. Filter by category
python scripts/list_rate_cards_catalogue.py --category CONSULTATION
python scripts/list_rate_cards_catalogue.py --category PHARMACY --limit 25

# 3. Search by name or code
python scripts/list_rate_cards_catalogue.py --search "Cardiology"

# 4. JSON output
python scripts/list_rate_cards_catalogue.py --json > rate_cards_catalogue.json
```





