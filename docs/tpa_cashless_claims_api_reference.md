# TPA & Cashless Claims API Reference

This document defines the request and response contracts for the TPA/Insurance and Cashless Claim management module, represented across the TPA Desk, Pre-Auth Submissions, and Claim Approval screens.

---

## 1. Get Claim Pre-Auth Queue (TPA Desk Dashboard)

Retrieves a list of all active cashless pre-authorization requests/claims pending review or submission at the TPA desk.

* **Endpoint**: `GET /admin/tpa/claims`
* **Method**: `GET`
* **Headers**:
  - `Authorization: Bearer <token>`
* **Query Parameters**:
  - `status` (Optional): Filter by claim status (`INITIATED`, `SUBMITTED`, `APPROVED`, `REJECTED`, `SETTLED`)
  - `search` (Optional): Filter by patient name, MRN (UHID), policy number, or TPA reference code

#### Response (200 OK)
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "e45ba234-c712-4eb3-81ef-8b26e55fc001",
        "tenant_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
        "bill_id": "90df8823-1102-45a8-9bde-452f38abf001",
        "patient_id": "f5b8d234-c712-4eb3-81ef-8b26e55fc244",
        "patient_name": "Rajesh Kumar",
        "patient_mrn": "UHID987654",
        "patient_insurance_id": "78a23412-fa42-4eb3-81ef-8b26e55fd201",
        "provider_name": "Star Health Insurance",
        "policy_number": "POL-10023984",
        "scheme_type": "TPA",
        "claimed_amount": 75000.00,
        "approved_amount": 0.00,
        "rejected_amount": 0.00,
        "status": "INITIATED",
        "tpa_reference": null,
        "submitted_at": null,
        "responded_at": null,
        "remarks": "Awaiting discharge summary and estimate for pre-auth upload",
        "documents": {},
        "created_at": "2026-07-03T10:15:30Z",
        "updated_at": "2026-07-03T10:15:30Z"
      }
    ],
    "total": 1
  }
}
```

---

## 2. Get Cashless Claim Details

Fetches the complete pre-authorization file for a specific patient cashless claim.

* **Endpoint**: `GET /admin/tpa/claims/{claim_id}`
* **Method**: `GET`
* **Headers**:
  - `Authorization: Bearer <token>`

#### Response (200 OK)
```json
{
  "success": true,
  "data": {
    "id": "e45ba234-c712-4eb3-81ef-8b26e55fc001",
    "tenant_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "bill_id": "90df8823-1102-45a8-9bde-452f38abf001",
    "patient_id": "f5b8d234-c712-4eb3-81ef-8b26e55fc244",
    "patient_name": "Rajesh Kumar",
    "patient_mrn": "UHID987654",
    "patient_insurance_id": "78a23412-fa42-4eb3-81ef-8b26e55fd201",
    "provider_name": "Star Health Insurance",
    "policy_number": "POL-10023984",
    "scheme_type": "TPA",
    "claimed_amount": 75000.00,
    "approved_amount": 50000.00,
    "rejected_amount": 25000.00,
    "status": "APPROVED",
    "tpa_reference": "TPA-REF-998877",
    "submitted_at": "2026-07-03T11:00:00Z",
    "responded_at": "2026-07-03T14:30:00Z",
    "remarks": "Approved 50K cashless. 25K rejected due to room rent capping exclusions.",
    "documents": {
      "pre_auth_form": "/claims/e45ba234/pre_auth.pdf",
      "diagnostic_reports": "/claims/e45ba234/reports.pdf",
      "approval_letter": "/claims/e45ba234/approval.pdf"
    },
    "created_at": "2026-07-03T10:15:30Z",
    "updated_at": "2026-07-03T14:30:00Z"
  }
}
```

---

## 3. Update Pre-Auth Status / TPA Decision (Edit Pre-Auth)

Submit TPA approval/rejection updates, cashless approved amounts, query statuses, or upload supporting pre-auth documents.

* **Endpoint**: `PATCH /admin/tpa/claims/{claim_id}`
* **Method**: `PATCH`
* **Headers**:
  - `Content-Type: application/json`
  - `Authorization: Bearer <token>`

#### Request Body
*(All fields are optional)*
```json
{
  "status": "APPROVED",
  "approved_amount": 50000.00,
  "rejected_amount": 25000.00,
  "tpa_reference": "TPA-REF-998877",
  "remarks": "Approved 50K cashless. 25K rejected due to room rent capping exclusions.",
  "documents": {
    "pre_auth_form": "/claims/e45ba234/pre_auth.pdf",
    "diagnostic_reports": "/claims/e45ba234/reports.pdf",
    "approval_letter": "/claims/e45ba234/approval.pdf"
  }
}
```

#### Response (200 OK)
```json
{
  "success": true,
  "data": {
    "id": "e45ba234-c712-4eb3-81ef-8b26e55fc001",
    "status": "APPROVED",
    "approved_amount": 50000.00,
    "rejected_amount": 25000.00,
    "tpa_reference": "TPA-REF-998877",
    "remarks": "Approved 50K cashless. 25K rejected due to room rent capping exclusions.",
    "documents": {
      "pre_auth_form": "/claims/e45ba234/pre_auth.pdf",
      "diagnostic_reports": "/claims/e45ba234/reports.pdf",
      "approval_letter": "/claims/e45ba234/approval.pdf"
    },
    "updated_at": "2026-07-03T14:30:00Z"
  }
}
```

---

## 4. Get Insurance / TPA Providers List

Retrieves registered Insurance/TPA companies configured in the hospital's administration settings.

* **Endpoint**: `GET /admin/tpa/providers`
* **Method**: `GET`
* **Headers**:
  - `Authorization: Bearer <token>`
* **Query Parameters**:
  - `search` (Optional): Filter by provider name or code
  - `status` (Optional): Filter by status (`ACTIVE`, `SUSPENDED`)

#### Response (200 OK)
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "282f1234-fa42-4eb3-81ef-8b26e55fd001",
        "provider_name": "Star Health TPA",
        "short_name_code": "STAR_HEALTH",
        "tpa_type": "Corporate TPA",
        "cashless_enabled": true,
        "pre_auth_mandatory": true,
        "status": "ACTIVE"
      }
    ],
    "total": 1
  }
}
```

---

## 5. Get Scheme Package Rates

Retrieves cashless surgery packages and procedure capping rates for a configured insurance scheme.

* **Endpoint**: `GET /admin/tpa/schemes/{scheme_id}/packages`
* **Method**: `GET`
* **Headers**:
  - `Authorization: Bearer <token>`

#### Response (200 OK)
```json
{
  "success": true,
  "data": {
    "scheme_id": "58a23412-fa42-4eb3-81ef-8b26e55fd201",
    "items": [
      {
        "procedure_name": "Angioplasty Package",
        "category": "CARDIOLOGY",
        "package_rate": 120000.00,
        "pre_auth_required": true
      },
      {
        "procedure_name": "Appendectomy",
        "category": "GENERAL_SURGERY",
        "package_rate": 45000.00,
        "pre_auth_required": true
      }
    ],
    "total": 2
  }
}
```
