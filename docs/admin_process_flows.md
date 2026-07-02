# HMS Admin — Process Flow Documentation

**Product:** Arovita Hospital Management System (HMS)
**Module:** Admin Service
**Version:** 1.0 (LJB Deployment)
**Last Updated:** 2026-07-02
**Prepared By:** Arovita Engineering Team

---

## Table of Contents

1. [Staff Onboarding Flow](#1-staff-onboarding-flow)
2. [Insurance & TPA Claims Flow](#2-insurance--tpa-claims-flow)
3. [Inventory Management Flow](#3-inventory-management-flow)
4. [Procurement Flow](#4-procurement-flow)
5. [Leave Management Flow](#5-leave-management-flow)
6. [Cross-Flow Integration Summary](#6-cross-flow-integration-summary)

---

## 1. Staff Onboarding Flow

### Overview

New hospital staff are created by an Admin or HR Manager and must complete a structured onboarding checklist before they can log in and access the system. The account cannot be activated until all 6 mandatory steps are satisfied and compliance documents are verified.

---

### 1.1 Lifecycle States

```
[NOT_ACTIVATED] --> [IN_PROGRESS] --> [READY_FOR_ACTIVATION] --> [ACTIVE]
                                                                      |
                                                               [SUSPENDED]
```

| State | Description |
|---|---|
| `NOT_ACTIVATED` | Admin has created the staff profile; no login yet |
| `IN_PROGRESS` | Staff has logged in for the first time and begun onboarding |
| `READY_FOR_ACTIVATION` | All 6 onboarding checklist items are 100% complete |
| `ACTIVE` | Admin/Compliance Officer has formally activated the account |
| `SUSPENDED` | Admin has disabled the account (reversible) |

---

### 1.2 Step-by-Step Flow

#### PHASE 1 — Admin Creates Staff Profile

**Actor:** Admin / HR Manager  
**Endpoint:** `POST /hrms/staff` (HRMS Service)

The admin creates the staff record with basic details: name, email, phone, role, department, branch, and joining date. The system:
- Generates a unique Employee Code (e.g., `DOC-00456`)
- Creates a Cognito user account with a temporary password
- Sends a welcome email/WhatsApp with login instructions
- Sets onboarding status to `NOT_ACTIVATED`

**Required fields:**
- `full_name`, `email`, `phone`, `role_id`, `department_id`, `branch_id`, `joining_date`

**Optional at creation:**
- `consultation_fee` (for doctors), `specialization_id`, `registration_number`

---

#### PHASE 2 — Staff First Login

**Actor:** Staff Member  
**Endpoints:**
- `POST /auth/login` — Staff logs in with temporary credentials
- System triggers MFA OTP to registered phone
- `POST /auth/login/mfa/verify` — Staff completes MFA verification

On first login, the system creates an onboarding progress record and sets status to `IN_PROGRESS`. Staff is redirected to the onboarding wizard.

---

#### PHASE 3 — Staff Completes Onboarding Checklist

**Actor:** Staff Member  
All 6 items must be completed before activation is possible:

| Step | Checklist Item | API Endpoint |
|---|---|---|
| 1 | **Schedule Setup** — Working shift, days, hours | `POST /hrms/staff/{user_id}/schedule` |
| 2 | **Payroll Setup** — Annual CTC, currency, pay grade | `POST /hrms/staff/{user_id}/payroll` |
| 3 | **Bank Account** — Account number, IFSC, bank name | `POST /hrms/staff/{user_id}/bank-accounts` |
| 4 | **Tax Profile** — PAN number, tax regime selection | `POST /hrms/staff/{user_id}/tax-profile` |
| 5 | **Documents Upload** — Role-specific mandatory documents | `POST /hrms/staff/{user_id}/documents` |
| 6 | **Training Completion** — Mandatory compliance courses | `PUT /hrms/staff/{user_id}/training/{training_id}` |

**Checking progress:**
```
GET /hrms/staff/{user_id}/onboarding
```
Returns completion percentage and which items are blocking activation.

---

#### PHASE 4 — Document Verification (Admin/Compliance Officer)

**Actor:** Compliance Officer or Admin  
Before the account can be activated, all role-specific mandatory documents must be verified.

**Mandatory Documents by Role:**

| Role | Required Documents |
|---|---|
| `DOCTOR` | Medical Registration Certificate, MBBS/MD Degree, ID Proof |
| `NURSE` / `HEAD_NURSE` | Nursing Council License, ID Proof |
| `LAB_TECHNICIAN` | Lab Technician Registration Certificate, ID Proof |
| `RADIOLOGIST` | Medical Registration Certificate, ID Proof |
| `PHARMACIST` | Pharmacy Council License, ID Proof |
| All Other Roles | Government ID Proof |

**Verification flow:**
```
GET  /hrms/staff/{user_id}/documents          -- View uploaded docs
POST /hrms/documents/{doc_id}/verify          -- Approve document
POST /hrms/documents/{doc_id}/reject          -- Reject with reason
```

A document must transition: `PENDING` → `VERIFIED` (approved) or `REJECTED` (returned to staff).

---

#### PHASE 5 — Account Activation

**Actor:** Admin or Compliance Officer  
Once all checklist items are 100% complete and all documents are `VERIFIED`:

```
POST /hrms/staff/{user_id}/activate
```

- Sets `onboarding_status` to `ACTIVE`
- Enables `login_enabled = TRUE` on the identity record
- Sends activation notification to staff via WhatsApp/email
- Audit log entry created: `staff.activation`

**If blocked:** The `GET /hrms/staff/{user_id}/onboarding` response will list the exact blocking conditions in the `blocking_documents` array.

---

#### PHASE 6 — Post-Activation (Admin Optional Actions)

| Action | Endpoint |
|---|---|
| Update consultation fee (Doctors) | `PATCH /admin/staff/{user_id}/fee` |
| Change role / promote | `PATCH /admin/staff/{user_id}/role` |
| Assign extra custom permissions | `POST /admin/staff/{user_id}/permissions` |
| Revoke custom permissions | `DELETE /admin/staff/{user_id}/permissions` |
| Suspend account | `PATCH /admin/staff/{user_id}/deactivate` |
| View staff details | `GET /admin/staff/{user_id}` |

---

### 1.3 Onboarding Flow Diagram

```
Admin creates staff   Staff first login     Staff fills checklist
     |                      |                     |
[POST /hrms/staff]    [POST /auth/login]    Schedule + Payroll +
     |                      |              Bank + Tax + Documents
     v                      v              + Training
Status: NOT_ACTIVATED  Status: IN_PROGRESS       |
                                                  v
                                        Compliance Officer
                                        verifies documents
                                                  |
                                                  v
                                        [POST .../activate]
                                                  |
                                                  v
                                          Status: ACTIVE
                                        Staff can log in fully
```

---

## 2. Insurance & TPA Claims Flow

### Overview

The TPA (Third-Party Administrator) module manages the relationship between the hospital and insurance companies. When a patient is admitted with insurance coverage, a TPA review is created and tracked through an approval pipeline until settlement.

---

### 2.1 TPA Lifecycle States

```
[PROVIDER_REGISTERED] --> [SCHEME_LINKED]
        |
        v
[CLAIM_SUBMITTED] --> [UNDER_REVIEW] --> [APPROVED] --> [SETTLED]
                                    \--> [REJECTED]
                                    \--> [MORE_INFO_REQUIRED]
```

---

### 2.2 Step-by-Step Flow

#### STEP 1 — Register TPA Provider

**Actor:** System Administrator / Hospital Admin  
**Endpoint:** `POST /admin/tpa/providers`

Register the insurance company/TPA that the hospital has a tie-up with.

**Request:**
```json
{
  "provider_name": "Star Health Insurance TPA",
  "short_name_code": "STAR-TPA",
  "contact_name": "Mr. Rajesh Kumar",
  "contact_email": "tpa.star@starhealth.in",
  "contact_phone": "+919999888877",
  "status": "ACTIVE"
}
```

**After registration:**
- Provider gets a unique UUID
- Appears in TPA provider list for patient claim linking

---

#### STEP 2 — Link Coverage Schemes

**Actor:** Admin  
**Endpoint:** `POST /admin/tpa/schemes`

Define the policy coverage rules — which procedures are covered, coverage limits, co-pay percentages, etc. Each scheme links to a TPA provider.

---

#### STEP 3 — Patient Presents Insurance at Reception

**Actor:** Receptionist  
When a patient is admitted with a TPA/insurance card, the Receptionist:
1. Searches the patient by UHID
2. Links the patient's insurance provider and policy number to the admission
3. A TPA review record is auto-created on admission: `POST /ipd/admissions` (with `tpa_provider_id` in payload)

---

#### STEP 4 — TPA Review Created & Submitted

**Actor:** Billing Executive / Admin  
A claim is formally submitted to the TPA with:
- Patient details (UHID, diagnosis, admission date)
- Itemized bill of services rendered
- Estimated or actual claim amount

**Endpoint:** `POST /admin/tpa/reviews` (or auto-created on IPD admission)

**Claim statuses:**

| Status | Meaning |
|---|---|
| `SUBMITTED` | Claim sent to TPA portal |
| `UNDER_REVIEW` | TPA is reviewing the claim |
| `MORE_INFO_REQUIRED` | TPA has requested additional documents |
| `APPROVED` | Claim approved for the stated amount |
| `PARTIALLY_APPROVED` | TPA approved a lesser amount |
| `REJECTED` | Claim denied by TPA |
| `SETTLED` | Payment received from TPA |

---

#### STEP 5 — Claim Review & Decision

**Actor:** Admin / Billing Executive  
Admin monitors and updates claim statuses:

```
GET  /admin/tpa/reviews                          -- List all claims (filter by status/provider)
GET  /admin/tpa/reviews/{review_id}              -- Claim detail view
PATCH /admin/tpa/claims/{claim_id}               -- Update authorization parameters
POST /admin/tpa/reviews/{review_id}/approve      -- Mark as Approved
POST /admin/tpa/reviews/{review_id}/reject       -- Mark as Rejected (with reason)
```

If TPA requests more info:
- Admin uploads supporting documents
- Resubmits the claim for re-review

---

#### STEP 6 — Settlement

**Actor:** Billing Executive  
Once the TPA approves and transfers funds:
- Mark claim as `SETTLED`
- Reconcile the remaining balance against the patient's bill
- Patient pays the co-pay / uncovered balance
- Generate final receipt

---

### 2.3 TPA Flow Diagram

```
Register TPA         Patient Admitted       Claim Submitted
Provider             with Insurance              |
   |                      |              [POST /admin/tpa/reviews]
[POST .../providers]  [POST /ipd/       Status: SUBMITTED
   |                  admissions]              |
   v                  + tpa_id                 v
Provider Active            |            TPA Under Review
                          v                    |
                    TPA Review          [PATCH .../claims]
                    Auto-Created               |
                                       /       |       \
                                 APPROVED  MORE_INFO  REJECTED
                                    |       REQUIRED      |
                                    |           |    Notify patient
                                    |      Upload docs    |
                                    v      + Resubmit     v
                                SETTLED              Write-off /
                                                    Patient pays all
```

---

## 3. Inventory Management Flow

### Overview

The inventory module tracks all medical supplies, drugs, and consumables. Stock is received via procurement, consumed via pharmacy dispensing or ward usage, and monitored for low stock and expiry.

---

### 3.1 Inventory Item Lifecycle

```
[CREATED] --> [IN_STOCK: GOOD] --> [IN_STOCK: LOW] --> [OUT_OF_STOCK]
                   |
              [EXPIRY_WARNING] --> [EXPIRED]
```

**Stock Status Levels:**

| Status | Condition |
|---|---|
| `GOOD` | Stock > reorder level |
| `LOW` | Stock <= reorder level (triggers alert) |
| `CRITICAL` | Stock <= 25% of reorder level |
| `OUT_OF_STOCK` | Stock = 0 |
| `EXPIRY_WARNING` | Item expiring within 30/60/90 days |
| `EXPIRED` | Past expiry date — must be quarantined |

---

### 3.2 Step-by-Step Flow

#### STEP 1 — Create Inventory Item (Master Record)

**Actor:** Pharmacy Manager / Admin  
**Endpoint:** `POST /admin/inventory/medical`

Register a new drug/consumable in the master inventory catalogue:

**Request:**
```json
{
  "item_code": "MED-PAR-650",
  "item_name": "Paracetamol 650mg Tablets",
  "category": "TABLETS",
  "batch_number": "BATCH-2026-A",
  "expiry_date": "2028-05-31",
  "opening_balance": 5000,
  "reorder_level": 500,
  "mrp": 2.50
}
```

**Item categories:** `TABLETS`, `CAPSULES`, `INJECTION`, `SYRUP`, `OINTMENT`, `SURGICAL_CONSUMABLE`, `EQUIPMENT`

---

#### STEP 2 — Monitor Inventory Levels

**Actor:** Pharmacy Manager / Admin  
**Endpoint:** `GET /admin/inventory?item_type=medical`

Monitor all items with:
- Current stock quantity
- Reorder level vs current stock comparison
- Expiry date warning flags
- Status level (GOOD / LOW / CRITICAL / OUT_OF_STOCK)

**Smart Alert Integration:**  
When stock drops below reorder level, the system auto-generates an alert:
```
POST /admin/alerts
{
  "alert_type": "INVENTORY_LOW",
  "severity": "HIGH",
  "title": "Critical Medicine Shortage",
  "message": "Paracetamol 650mg stock below minimum threshold (150 units remaining).",
  "source_module": "pharmacy"
}
```

Admin/Pharmacy Manager receives a WhatsApp notification and sees the alert on the dashboard.

---

#### STEP 3 — Stock Consumption

Stock is reduced automatically when:
- Pharmacist dispenses a prescription (`POST /pharmacy/dispense`)
- Nurse records drug administration from MAR
- Ward staff marks surgical consumable used

No manual deduction needed — the system auto-updates quantities on each dispensing event.

---

#### STEP 4 — Inter-Department Stock Transfer

**Actor:** Pharmacy Manager / Admin  
When stock needs to be moved between departments (e.g., Pharmacy to Emergency Ward):

**Endpoint:** `POST /admin/inventory/transfers`

```json
{
  "item_id": "inv-uuid-123456",
  "from_department": "pharmacy",
  "to_department": "emergency",
  "quantity": 100,
  "reason": "Emergency shortage of Paracetamol"
}
```

Transfer creates an audit trail for stock accountability.

---

#### STEP 5 — Expiry Management

**Actor:** Pharmacy Manager  
**Endpoint:** `GET /admin/inventory?expiry_warning=true`

Items approaching expiry are flagged with:
- Days until expiry
- Quantity at risk

**Actions available:**
- Quarantine expired/near-expiry batch
- Initiate return to supplier (via procurement module)
- Write off expired stock (with audit log entry)

---

### 3.3 Inventory Flow Diagram

```
Admin creates          Stock Received       Consumption
Inventory Item         (via GRN)            Events
     |                     |                    |
[POST /inventory]   [POST /procurement/   Pharmacy dispense
     |               grn] Stock +1         Nurse MAR entry
     v                                     Ward usage
Item In Catalogue                              |
     |                                         v
     v                                   Stock decrements
Level Monitoring                         auto on each event
[GET /inventory]                               |
     |                                         v
     v                               [GET /inventory] shows
  GOOD -> LOW -> CRITICAL               updated levels
     |                                         |
     v                                         v
Alert auto-fired               LOW threshold crossed
[POST /alerts]                         |
     |                                 v
     v                    Alert fired + Procurement
Admin notified                 PO created
(WhatsApp + Dashboard)
```

---

## 4. Procurement Flow

### Overview

The procurement module manages the end-to-end purchasing cycle: from identifying low-stock or new requirements, to vendor selection, purchase order (PO) creation, goods receipt, and invoice reconciliation.

---

### 4.1 Procurement Lifecycle States

```
[REQUIREMENT IDENTIFIED]
         |
         v
   [PO: DRAFT] --> [PO: SUBMITTED] --> [PO: APPROVED] --> [PO: DELIVERED]
                                                                  |
                                                          [GRN: CREATED]
                                                                  |
                                                     [INVOICE: RECONCILED]
```

---

### 4.2 Step-by-Step Flow

#### STEP 1 — Requirement Identified

**Triggers:**
- Inventory alert fires when stock drops below `reorder_level`
- Manual review by Pharmacy Manager
- Upcoming surgery requiring specific consumables
- New medicine addition to formulary

---

#### STEP 2 — Vendor Lookup

**Actor:** Admin / Procurement Officer  
Vendors (suppliers) must be registered in the system before a PO can be raised.

```
GET /admin/procurement/vendors              -- List registered vendors
POST /admin/procurement/vendors            -- Register new vendor
```

A vendor record contains:
- Company name, GST number, contact details
- Product categories they supply
- Payment terms and lead time

---

#### STEP 3 — Create Purchase Order (PO)

**Actor:** Pharmacy Manager / Admin  
**Endpoint:** `POST /admin/procurement/orders`

**Request:**
```json
{
  "vendor_id": "vendor-uuid-123456",
  "expected_delivery_date": "2026-07-15",
  "items": [
    { "item_code": "MED-PAR-650", "quantity": 10000, "unit_price": 2.10 },
    { "item_code": "MED-AMO-500", "quantity": 5000,  "unit_price": 4.50 }
  ]
}
```

**Response:** PO is created with status `DRAFT` and a PO number (e.g., `PO-2026-0089`).

**PO fields auto-calculated:**
- Total amount (quantity x unit_price per line)
- Expected GST / tax amounts
- Expected delivery date

---

#### STEP 4 — PO Approval

**Actor:** Admin / Department Head  
The PO is reviewed and approved before being sent to the vendor.

```
POST /admin/procurement/orders/{po_id}/approve     -- Approve PO
POST /admin/procurement/orders/{po_id}/reject      -- Reject with reason
```

After approval, the PO:
- Status changes to `SUBMITTED`
- System generates a printable PO PDF
- Vendor is notified (if email integration configured)

---

#### STEP 5 — Goods Receipt Note (GRN)

**Actor:** Pharmacy Manager / Store Keeper  
**Endpoint:** `POST /admin/procurement/grn`

When physical goods arrive from the vendor, a Goods Receipt Note is created:

```json
{
  "po_id": "po-uuid-123456",
  "received_date": "2026-07-15",
  "items": [
    {
      "item_code": "MED-PAR-650",
      "quantity_ordered": 10000,
      "quantity_received": 9800,
      "batch_number": "BATCH-2026-B",
      "expiry_date": "2028-06-30"
    }
  ]
}
```

**On GRN creation:**
- Inventory stock is automatically incremented by received quantity
- Discrepancies (ordered vs received) are flagged
- Batch and expiry details are recorded per item
- PO status updates to `PARTIALLY_DELIVERED` or `DELIVERED`

---

#### STEP 6 — Invoice Reconciliation

**Actor:** Billing Executive / Admin  
**Endpoint:** `POST /admin/procurement/invoices/{invoice_id}/reconcile`

The vendor invoice is matched against:
- Quantities in the GRN (was everything billed actually received?)
- Unit prices in the PO (does the invoice match agreed pricing?)
- GST amounts

**Reconciliation outcomes:**

| Outcome | Action |
|---|---|
| Fully matched | Mark invoice as `RECONCILED`, release payment |
| Price discrepancy | Raise dispute with vendor, hold payment |
| Quantity discrepancy | Process debit note for short-supplied items |
| Damaged goods | Create return order to vendor |

---

#### STEP 7 — Payment & Closure

After successful reconciliation:
- Finance team approves payment to vendor
- PO and GRN status marked `CLOSED`
- Audit log entry: `procurement.po.closed`

---

### 4.3 Procurement Flow Diagram

```
Low Stock Alert          Vendor Selected
or Manual Request              |
       |               [GET /procurement/vendors]
       v                       |
Requirement Identified         v
       |               [POST /procurement/orders]
       v               PO Status: DRAFT
Raise Purchase Order           |
       |                       v
       v               Admin Approves PO
[POST .../orders]      PO Status: SUBMITTED
       |                       |
       v                       v
   PO Created           Vendor Delivers Goods
   Status: DRAFT               |
                        [POST .../grn]
                        Stock auto-incremented
                        PO: DELIVERED
                               |
                               v
                        Invoice Received
                        [POST .../reconcile]
                               |
                     /---------+----------\
                  Matched             Discrepancy
                     |                    |
               Release Payment       Dispute / Debit Note
               PO: CLOSED
```

---

## 5. Leave Management Flow

### Overview

Staff can apply for leave through the system, and admins/managers can approve or reject requests. Leave balances are tracked per leave type and year.

---

### 5.1 Leave Types

| Leave Type | Description |
|---|---|
| `CASUAL` | Short-notice personal leave |
| `SICK` | Medical leave (may require certificate) |
| `EARNED` | Accrued paid leave |
| `MATERNITY` | Statutory maternity leave |
| `PATERNITY` | Statutory paternity leave |
| `UNPAID` | Leave without pay |

---

### 5.2 Step-by-Step Flow

#### STEP 1 — Staff Applies for Leave

**Actor:** Staff Member  
**Endpoint:** `POST /admin/staff/me/leaves/apply`

```json
{
  "leave_type": "CASUAL",
  "from_date": "2026-07-10",
  "to_date": "2026-07-15",
  "reason": "Family gathering"
}
```

Status set to `PENDING`. System checks for:
- Overlapping existing leave requests (returns `409 Conflict` if overlap found)
- Sufficient leave balance

---

#### STEP 2 — Admin Reviews

**Actor:** Admin / HR Manager  
**Endpoints:**
```
GET  /admin/leaves/pending               -- View all pending leave requests
GET  /admin/staff/{user_id}/leaves       -- View specific staff leave history
```

---

#### STEP 3 — Approve or Reject

**Actor:** Admin  
```
POST /admin/leaves/{leave_id}/approve    -- Approve leave
POST /admin/leaves/{leave_id}/reject     -- Reject with reason
```

- On approval: leave balance is deducted, staff notified
- On rejection: staff is notified with reason, balance unchanged

---

#### STEP 4 — Leave Balance Monitoring

```
GET /admin/staff/me/leaves/balance?year=2026    -- Staff's own balance
GET /admin/staff/{user_id}/leaves/balance       -- Admin viewing staff balance
```

Returns per-type balance: total entitled, used, remaining.

---

## 6. Cross-Flow Integration Summary

The following table shows how the four major flows interact with each other and with other HMS modules:

| Flow | Triggers | Outputs | Affects |
|---|---|---|---|
| **Staff Onboarding** | Admin creates profile | Active staff account with role | Auth service, HRMS, Audit logs |
| **Leave Management** | Staff submits request | Approved/rejected leave record | Staff availability, scheduling |
| **Inventory** | Stock consumed or GRN received | Updated stock levels, alerts | Pharmacy dispensing, Procurement |
| **Procurement** | Low stock alert or manual PO | GRN stock increase, invoice reconciled | Inventory levels, Finance |
| **TPA/Insurance** | Patient admitted with insurance | Claim record, settlement | Patient billing, Finance |

### Shared Infrastructure

All flows share the following admin infrastructure:

| Component | Role |
|---|---|
| **Smart Alerts** (`/admin/alerts`) | Fires notifications across all flows (low stock, claim pending, doc expiry) |
| **Audit Logs** (`/admin/compliance/audit`) | Every state change is immutably recorded |
| **Settings** (`/admin/settings`) | Configures thresholds for alerts, leave policies, reorder levels |
| **Role Permissions** (`/admin/roles`, `/admin/staff/{id}/permissions`) | Controls who can perform which actions in each flow |

---

*Document maintained by Arovita Engineering. For full API payload specifications, refer to [admin_api_docs.md](./admin_api_docs.md) and [staff_onboarding_guide.md](./staff_onboarding_guide.md).*
