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
   * 4.3 [Generate or Refresh Invoice](#43-generate-or-refresh-invoice)
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
* **Required Permission:** `wallet:view` or `billing:view`
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

### 5.3 Get Patient Wallet Ledger
Retrieves the append-only, immutable transaction history for the patient's wallet.

* **Endpoint:** `GET /billing/wallet/{patient_id}/ledger?page=1&per_page=20`
* **Method:** `GET`
* **Required Permission:** `wallet:ledger:view` or `wallet:view`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "wallet": {
      "id": "37b3eaca-7fe9-45fa-b616-3ab226b84f5f",
      "patient_id": "0d2c0b64-c2c3-4d41-9457-4ea2e6d6eb10",
      "currency": "INR",
      "available_balance": 25000.00,
      "status": "ACTIVE",
      "created_at": "2026-09-21T13:39:06.921295+05:30",
      "updated_at": "2026-09-21T13:41:24.710875+05:30"
    },
    "transactions": [
      {
        "id": "018e652a-912f-7c12-98ab-4d2a1e8c76ad",
        "wallet_id": "37b3eaca-7fe9-45fa-b616-3ab226b84f5f",
        "patient_id": "0d2c0b64-c2c3-4d41-9457-4ea2e6d6eb10",
        "transaction_type": "DEBIT",
        "credit_source": null,
        "amount": 2000.00,
        "balance_before": 27000.00,
        "balance_after": 25000.00,
        "financial_account_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
        "ipd_id": "ipd-admission-uuid-1111",
        "opd_id": null,
        "surgery_id": null,
        "procedure_id": null,
        "payment_id": null,
        "reference_type": "BILL_PAYMENT",
        "reference_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
        "status": "SUCCESS",
        "idempotency_key": null,
        "created_by": "f8bb5a02-0eb1-4366-814c-0763ba4f2b74",
        "created_at": "2026-09-21T13:41:24.710875+05:30",
        "reversal_of": null,
        "metadata": {
          "invoice_number": "BILL-20260921-0001"
        }
      },
      {
        "id": "018e652a-89aa-7b01-81cb-2e1c9a7b65fa",
        "wallet_id": "37b3eaca-7fe9-45fa-b616-3ab226b84f5f",
        "patient_id": "0d2c0b64-c2c3-4d41-9457-4ea2e6d6eb10",
        "transaction_type": "CREDIT",
        "credit_source": "INSURANCE",
        "amount": 50000.00,
        "balance_before": 0.00,
        "balance_after": 50000.00,
        "financial_account_id": null,
        "ipd_id": "ipd-admission-uuid-1111",
        "opd_id": null,
        "surgery_id": null,
        "procedure_id": null,
        "payment_id": null,
        "reference_type": "INSURANCE_CLAIM",
        "reference_id": "CLAIM-776655",
        "status": "SUCCESS",
        "idempotency_key": "IDEMP-INS-776655",
        "created_by": "f8bb5a02-0eb1-4366-814c-0763ba4f2b74",
        "created_at": "2026-09-21T13:39:20.123456+05:30",
        "reversal_of": null,
        "metadata": {
          "notes": "Cashless pre-auth settled"
        }
      }
    ],
    "total_count": 2,
    "page": 1,
    "per_page": 20
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




