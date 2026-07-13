# Pharmacy Manager API Specification (HMS & Standalone Retail)

This document is the master API specification for the **Pharmacy Service** within the Hospital Management System (HMS). It covers both **Hospital Integrated Mode** (OPD, IPD, Emergency, ICU, OT/Surgery, Discharge) and **Standalone Retail Mode** (Walk-in, OTC, External Prescriptions), aligning with the requirements in [Arovita_HMS_Pharmacy_Module.md](file:///c:/Users/saika/OneDrive/Desktop/Arovita/ops-hms-ljb/Arovita_HMS_Pharmacy_Module.md).

---

## 1. Global Conventions

### Base URL
```
https://<api-id>.execute-api.ap-south-1.amazonaws.com/<stage>
```

### Authentication & Tenant Headers
All requests must include a valid JWT token and tenant context:
```http
Authorization: Bearer <access_token>
Content-Type: application/json
X-Tenant-Id: <tenant-uuid>
```

### Standard Response Envelope

#### Success Envelope (`200 OK` / `201 Created`)
```json
{
  "success": true,
  "code": 200,
  "message": "Action completed successfully",
  "data": {}
}
```

#### Error Envelope (`400` / `401` / `403` / `404` / `409` / `500`)
```json
{
  "success": false,
  "code": 400,
  "error": "Detailed error message explaining the failure"
}
```

---

## 2. API Endpoint Directory

* **Section 3: Dashboard & KPI Metrics**
  * `GET /pharmacy/dashboard/summary` — Quick stats overview
  * `GET /pharmacy/dashboard/alerts` — Critical system notifications
* **Section 4: Patient & Customer Management**
  * `GET /patients/search` — Search hospital database
  * `POST /pharmacy/customers` — Register walk-in customer
* **Section 5: Prescription Lifecycle & Versioning**
  * `POST /pharmacy` — Create new prescription
  * `GET /pharmacy/{prescription_id}` — Get prescription details
  * `PUT /pharmacy/{prescription_id}` — Update prescription (version increment)
  * `POST /pharmacy/{prescription_id}/cancel` — Cancel active prescription
  * `GET /pharmacy/{prescription_id}/history` — Event timeline log
  * `GET /pharmacy/{prescription_id}/versions` — List previous versions
* **Section 6: Dispensing Cart Engine**
  * `POST /pharmacy/carts` — Initialize cart
  * `GET /pharmacy/carts/{id}` — Get cart details & picking directions
  * `POST /pharmacy/carts/{id}/items` — Reserve medicine batch (FEFO)
  * `POST /pharmacy/carts/{id}/verify` — Perform clinical validation checks
  * `POST /pharmacy/carts/{id}/checkout` — Finalize billing & dispense
  * `POST /pharmacy/carts/cleanup` — Clean expired carts
* **Section 7: Inventory & Medicine Catalogue**
  * `GET /pharmacy/medicines` — Search catalog
  * `GET /pharmacy/inventory/batches` — Fetch batches sorted by FEFO
* **Section 8: Returns & Refund Override**
  * `POST /pharmacy/returns` — Request return/exchange
  * `POST /pharmacy/returns/{id}/approve` — Approve refund with manager override
* **Section 9: Shift Operations**
  * `POST /pharmacy/shifts/handover` — Reconcile & hand over shift cash
  * `GET /pharmacy/shifts/handover/history` — Shift handover logs
* **Section 10: Procurement Management**
  * `POST /pharmacy/procurement/suppliers` — Create Supplier
  * `POST /pharmacy/procurement/requisitions` — Create Purchase Requisition
  * `POST /pharmacy/procurement/requisitions/{id}/approve` — Requisition approval
  * `POST /pharmacy/procurement/orders` — Create Purchase Order (PO)
  * `POST /pharmacy/procurement/grns` — Create Goods Receipt Note (GRN)
  * `POST /pharmacy/procurement/grns/{id}/verify` — Verify GRN (commit to stock)

---

## 3. Dashboard & KPI Metrics

### 3.1 Get Dashboard Summary
* **Endpoint:** `GET /pharmacy/dashboard/summary`
* **Purpose:** Fetches KPI metrics across prescriptions, billing, and inventory.
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "prescription_kpis": {
      "total_prescriptions": 340,
      "pending_verification": 15,
      "pending_dispensing": 10,
      "dispensed_today": 112,
      "hold_prescriptions": 4,
      "cancelled_prescriptions": 8
    },
    "billing_kpis": {
      "todays_sales": 48650.00,
      "todays_bills": 32,
      "pending_payments": 5,
      "refunds": 2240.00
    },
    "inventory_kpis": {
      "inventory_value": 1250000.00,
      "low_stock_items": 14,
      "out_of_stock_items": 3,
      "near_expiry_items": 22,
      "expired_items": 5
    }
  }
}
```

### 3.2 Get Dashboard Alerts
* **Endpoint:** `GET /pharmacy/dashboard/alerts`
* **Purpose:** Returns active, high-priority notifications including drug recalls, low stock levels, expired batches, controlled drug authorization alerts, and pending approvals.
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "id": "alert-uuid-1",
      "type": "OUT_OF_STOCK",
      "medicine_name": "Amoxicillin 500mg",
      "batch_number": "N/A",
      "current_stock": 0,
      "timestamp": "2026-07-13T10:00:00Z"
    },
    {
      "id": "alert-uuid-2",
      "type": "EXPIRY_WARNING",
      "medicine_name": "Cetirizine 10mg",
      "batch_number": "CTZ1023",
      "expiry_date": "2026-07-30",
      "timestamp": "2026-07-13T10:00:00Z"
    },
    {
      "id": "alert-uuid-3",
      "type": "CONTROLLED_DRUG_PENDING_APPROVAL",
      "medicine_name": "Fentanyl 50mcg",
      "prescription_number": "RX-2026-9081",
      "timestamp": "2026-07-13T10:15:00Z"
    }
  ]
}
```

---

## 4. Patient & Customer Management

### 4.1 Unified Patient Lookup
* **Endpoint:** `GET /patients/search`
* **Query Parameters:**
  | Parameter | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `q` | String | **Mandatory** | Query matches patient Name, UHID, or Mobile Number | Min length: 3 |
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "patient_id": "e2c07a01-2092-4876-8889-112233445566",
      "uhid": "PAT-2026-0089",
      "full_name": "Priyanka Nair",
      "age": 32,
      "gender": "FEMALE",
      "mobile": "+919781186435",
      "allergies": ["Penicillin"],
      "insurance_status": "ACTIVE"
    }
  ]
}
```

### 4.2 Register Walk-in Customer
* **Endpoint:** `POST /pharmacy/customers`
* **Request Body:**
  | Field | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `full_name` | String | **Mandatory** | Customer full name | Min length: 2 |
  | `phone` | String | **Mandatory** | Mobile number | Valid Indian standard or international format |
  | `age` | Integer | Optional | Customer age | Value must be > 0 and < 125 |
  | `gender` | String | Optional | Gender | `MALE`, `FEMALE`, `OTHER` |
  | `dob` | Date | Optional | Date of Birth | `YYYY-MM-DD` format |
  | `address` | String | Optional | Physical address | Max 500 chars |
  | `email` | String | Optional | Email address | Valid email string |
* **Success Response (`201 Created`):**
```json
{
  "success": true,
  "code": 201,
  "data": {
    "customer_id": "cust-uuid-8899",
    "full_name": "John Doe",
    "phone": "+919876543210",
    "created_at": "2026-07-13T10:20:00Z"
  }
}
```

---

## 5. Prescription Lifecycle & Versioning

### 5.1 Create Prescription
* **Endpoint:** `POST /pharmacy`
* **Request Body:**
  | Field | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `patient_id` | UUID | Optional | Patient identifier from HMS database | Mandatory if walk_in_customer is empty |
  | `customer_id` | UUID | Optional | Customer identifier for walk-ins | Mandatory if patient_id is empty |
  | `doctor_id` | UUID | Optional | Doctor uuid authorizing prescription | Optional if external prescribing |
  | `doctor_name` | String | Optional | External doctor name | Max 200 chars |
  | `context_type` | String | **Mandatory** | Visit category | `OPD_VISIT`, `IPD_ADMISSION`, `RETAIL` |
  | `context_id` | UUID | Optional | Visit or Admission ID context | |
  | `source_module` | String | **Mandatory** | Origin of prescription | `OPD`, `IPD`, `EMERGENCY`, `ICU`, `OT_SURGERY`, `DISCHARGE`, `WALK_IN` |
  | `priority` | String | Optional | Urgency level | `NORMAL`, `STAT`, `HIGH_PRIORITY` (default: `NORMAL`) |
  | `allergy_checked` | Boolean | Optional | Indicates safety check completion | Default: `false` |
  | `interaction_checked` | Boolean | Optional | Contra-indications validation | Default: `false` |
  | `items` | Array | **Mandatory** | Prescription items | 1 to 50 items |
  | `notes` | String | Optional | Clinician instructions | |
* **PrescriptionItem Object Structure:**
  * `medicine_id` (UUID, Optional): ID from medicines catalog. Mandatory if medicine_name is empty.
  * `medicine_name` (String, Optional): Drug name. Mandatory if medicine_id is empty.
  * `generic_name` (String, Optional): Active pharmaceutical ingredient.
  * `strength` (String, Optional): e.g. `500mg`, `10ml`.
  * `dosage_form` (String, Optional): `TABLET`, `SYRUP`, `INJECTION`, `CAPSULE`, etc.
  * `route` (String, Mandatory): `ORAL`, `IV`, `IM`, `SC`, `TOPICAL`, `INHALATION`.
  * `frequency` (String, Mandatory): Standard frequency tags (`OD`, `BD`, `TDS`, `QDS`, `SOS`, `STAT`, `HS`, `AC`, `PC`) or normalized string equivalent.
  * `duration_days` (Integer, Mandatory): Days between 1 and 365.
  * `morning_dose` (Boolean, Optional): Default: `false`.
  * `afternoon_dose` (Boolean, Optional): Default: `false`.
  * `evening_dose` (Boolean, Optional): Default: `false`.
  * `night_dose` (Boolean, Optional): Default: `false`.
  * `with_food` (Boolean, Optional): Default: `false`.
  * `instructions` (String, Optional): Dietary/admin directions. Max 500 chars.
* **Success Response (`201 Created`):**
```json
{
  "success": true,
  "code": 201,
  "data": {
    "prescription_id": "rx-uuid-4444",
    "prescription_number": "RX-2026-9001",
    "status": "SIGNED",
    "version_no": 1,
    "items": [
      {
        "id": "rx-item-uuid-6666",
        "medicine_name": "Amoxicillin 500mg",
        "quantity_prescribed": 15,
        "is_discontinued": false
      }
    ],
    "created_at": "2026-07-13T10:25:00Z"
  }
}
```

### 5.2 Retrieve Prescription Details
* **Endpoint:** `GET /pharmacy/{prescription_id}`
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "prescription_id": "rx-uuid-4444",
    "prescription_number": "RX-2026-9001",
    "patient": {
      "uhid": "PAT-2026-0089",
      "name": "Priyanka Nair",
      "age": 32,
      "gender": "FEMALE",
      "allergies": ["Penicillin"]
    },
    "doctor_name": "Dr. Tanvi Shrivastava",
    "department": "Orthopaedics",
    "source_module": "OPD",
    "status": "SIGNED",
    "version_no": 1,
    "items": [
      {
        "id": "rx-item-uuid-6666",
        "medicine_name": "Amoxicillin 500mg",
        "generic_name": "Amoxicillin",
        "route": "ORAL",
        "frequency": "TDS",
        "duration_days": 5,
        "quantity_prescribed": 15,
        "quantity_dispensed": 0,
        "quantity_remaining": 15,
        "stock_status": "AVAILABLE"
      }
    ]
  }
}
```

### 5.3 Update Prescription (New Version)
* **Endpoint:** `PUT /pharmacy/{prescription_id}`
* **Description:** Edits active prescription items. Replaces current active medicines list, increments version index by `1`, and moves previous configurations into an immutable history archive.
* **Request Body:**
  | Field | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `change_reason` | String | **Mandatory** | Audit trail note explaining changes | Min: 5, Max: 500 chars |
  | `allergy_checked` | Boolean | Optional | Update safety flag | |
  | `items` | Array | **Mandatory** | Full set of new prescription items | |
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "message": "Prescription version incremented successfully",
  "data": {
    "prescription_id": "rx-uuid-4444",
    "prescription_number": "RX-2026-9001",
    "version_no": 2,
    "change_reason": "Dose adjusted due to allergic sensitivity notes.",
    "items": [
      {
        "id": "rx-item-uuid-7777",
        "medicine_name": "Amoxicillin 250mg",
        "frequency": "BD",
        "duration_days": 5
      }
    ]
  }
}
```

### 5.4 Cancel Prescription
* **Endpoint:** `POST /pharmacy/{prescription_id}/cancel`
* **Request Body:**
  | Field | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `reason` | String | **Mandatory** | Rationale for cancellation | Enums: `DOCTOR_CANCELLED`, `DUPLICATE`, `WRONG_PRESCRIPTION`, `PATIENT_REFUSED`, `EXPIRED_PRESCRIPTION`, `OTHER` |
  | `remarks` | String | Optional | Specific audit notes | Max 500 chars |
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "message": "Prescription status marked as CANCELLED",
  "data": {
    "prescription_id": "rx-uuid-4444",
    "status": "CANCELLED",
    "cancelled_at": "2026-07-13T10:30:00Z"
  }
}
```

### 5.5 Get Prescription History Timeline
* **Endpoint:** `GET /pharmacy/{prescription_id}/history`
* **Purpose:** Returns structural logs of all events occurred on the prescription context.
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "event_type": "CREATED",
      "user_name": "Dr. Tanvi Shrivastava",
      "role": "DOCTOR",
      "timestamp": "2026-07-13T10:25:00Z",
      "remarks": "Initial entry"
    },
    {
      "event_type": "EDITED",
      "user_name": "Dr. Tanvi Shrivastava",
      "role": "DOCTOR",
      "timestamp": "2026-07-13T10:28:00Z",
      "remarks": "Version 2: Dose adjusted due to allergic sensitivity notes."
    }
  ]
}
```

### 5.6 Get All Prescription Versions
* **Endpoint:** `GET /pharmacy/{prescription_id}/versions`
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": [
    { "version_no": 1, "created_at": "2026-07-13T10:25:00Z", "change_reason": "Initial entry" },
    { "version_no": 2, "created_at": "2026-07-13T10:28:00Z", "change_reason": "Dose adjusted due to allergic sensitivity notes." }
  ]
}
```

---

## 6. Dispensing Cart Engine

To handle high volumes and avoid double-allocation race conditions, dispensing actions require a cart state. Active items placed in a cart reserve stock rows temporarily for **15 minutes** before auto-expiring.

### 6.1 Initialize Cart
* **Endpoint:** `POST /pharmacy/carts`
* **Request Body:**
  | Field | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `cart_type` | String | **Mandatory** | Cart origin type | `PRESCRIPTION`, `WALK_IN` |
  | `prescription_id` | UUID | Optional | Prescription source | Mandatory if cart_type is `PRESCRIPTION` |
  | `customer_id` | UUID | Optional | Walk-in customer uuid | Optional for `WALK_IN` mode |
* **Success Response (`201 Created`):**
```json
{
  "success": true,
  "code": 201,
  "data": {
    "cart_id": "cart-uuid-9999",
    "status": "INITIALIZED",
    "expires_at": "2026-07-13T10:55:00Z"
  }
}
```

### 6.2 Get Cart Details & Picking Instructions
* **Endpoint:** `GET /pharmacy/carts/{id}`
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "cart_id": "cart-uuid-9999",
    "status": "INITIALIZED",
    "items": [
      {
        "medicine_name": "Amoxicillin 500mg",
        "quantity_reserved": 15,
        "batch_number": "AMX-129",
        "picking_instructions": {
          "warehouse": "Main Store",
          "rack": "A-3",
          "shelf": "Level-2",
          "bin": "B-12"
        }
      }
    ],
    "financials": {
      "subtotal": 150.00,
      "discount": 15.00,
      "gst_amount": 16.20,
      "grand_total": 151.20
    }
  }
}
```

### 6.3 Add Item to Cart (Reserve Stock)
* **Endpoint:** `POST /pharmacy/carts/{id}/items`
* **Description:** Reserves specific medicine quantities. It performs **FEFO (First Expired First Out)** scheduling automatically unless a manual `batch_id` bypass is specified. 
* **Concurrency Safeguard:** Locks row inside transactions via `SELECT FOR UPDATE` on `pharmacy.inventory_batches`.
* **Request Body:**
  | Field | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `medicine_id` | UUID | **Mandatory** | Medicine catalogue reference | Must exist in catalogue |
  | `quantity` | Integer | **Mandatory** | Number of units to allocate | Must be > 0 |
  | `batch_id` | UUID | Optional | Manual override batch selector | |
* **Success Response (`201 Created`):**
```json
{
  "success": true,
  "code": 201,
  "message": "Stock reserved successfully",
  "data": {
    "item_id": "cart-item-uuid-223",
    "medicine_name": "Amoxicillin 500mg",
    "batch_number": "AMX-129",
    "quantity": 15
  }
}
```

### 6.4 Verify Cart (Clinical Validation Checks)
* **Endpoint:** `POST /pharmacy/carts/{id}/verify`
* **Purpose:** Runs critical validations prior to dispensing. All status outputs must be checked and clear.
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "cart_id": "cart-uuid-9999",
    "is_valid_to_dispense": true,
    "validations": {
      "allergy_check": { "status": "PASSED", "message": "No drug-allergy contra-indications flagged." },
      "drug_interaction": { "status": "PASSED", "message": "No negative medication interactions detected." },
      "duplicate_therapy": { "status": "PASSED", "message": "No active therapeutic duplication exists." },
      "expiry_check": { "status": "PASSED", "message": "All batches valid." },
      "controlled_drug_check": { "status": "PASSED", "message": "Controlled drug rules verified or approved by Dr." },
      "recall_check": { "status": "PASSED", "message": "No recalled items included." }
    }
  }
}
```

### 6.5 Checkout Cart (Finalize Dispense & Billing)
* **Endpoint:** `POST /pharmacy/carts/{id}/checkout`
* **Description:** Deducts active inventories permanently, compiles the invoice/bill, registers transaction payments, and switches the prescription dispensing state.
* **Request Body:**
  | Field | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `payment_mode` | String | **Mandatory** | Financial instrument | `CASH`, `CARD`, `UPI`, `NET_BANKING`, `INSURANCE_TPA`, `CREDIT` |
  | `payment_reference` | String | Optional | Bank transactional reference ID | Required if mode is UPI/Card/Net Banking |
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "message": "Checkout completed. Receipt generated.",
  "data": {
    "invoice_number": "BILL-2026-904",
    "billing_status": "PAID",
    "subtotal": 150.00,
    "gst_amount": 16.20,
    "grand_total": 151.20,
    "barcode_payload": "BILL2026904",
    "dispensed_items": [
      {
        "medicine_name": "Amoxicillin 500mg",
        "quantity": 15,
        "batch_number": "AMX-129"
      }
    ]
  }
}
```

### 6.6 Clean Expired Carts
* **Endpoint:** `POST /pharmacy/carts/cleanup`
* **Description:** Cron/system endpoint. Checks for active carts past their 15-minute expiration timeframe, releases their stock reservation blocks (adds back to `available_quantity`), and transitions cart statuses to `EXPIRED`.
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "cleaned_carts_count": 3,
    "released_items_count": 5
  }
}
```

---

## 7. Inventory & Medicine Catalogue

### 7.1 Search Medicines
* **Endpoint:** `GET /pharmacy/medicines`
* **Query Parameters:**
  | Parameter | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `q` | String | Optional | Search query matching Name, Brand, Barcode | Min 1 char |
  | `category` | String | Optional | Filter by category | e.g. `TABLET`, `SYRUP` |
  | `stock_status` | String | Optional | Filter by inventory status | `IN_STOCK`, `LOW_STOCK`, `OUT_OF_STOCK` |
  | `page` | Integer | Optional | Page index | Default: 1 |
  | `page_size` | Integer | Optional | Items per page | Default: 20 |
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "id": "med-uuid-001",
        "name": "Paracetamol 650mg",
        "brand": "Calpol",
        "generic_name": "Paracetamol",
        "category": "TABLET",
        "barcode": "890104300122",
        "stock_status": "IN_STOCK",
        "available_quantity": 450,
        "reserved_quantity": 10,
        "price": 15.50
      }
    ],
    "total": 1,
    "page": 1,
    "page_size": 20
  }
}
```

### 7.2 Get Batches for Medicine
* **Endpoint:** `GET /pharmacy/inventory/batches`
* **Query Parameters:**
  | Parameter | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `medicine_id` | UUID | **Mandatory** | Medicine catalogue identifier | Must exist in DB |
* **Description:** Returns batches sorted by FEFO (nearest expiry date first).
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "batch_id": "batch-uuid-099",
      "batch_number": "B-9023",
      "expiry_date": "2027-10-31",
      "stock_qty": 500,
      "status": "ACTIVE"
    }
  ]
}
```

---

## 8. Returns & Refund Override

### 8.1 Request Return / Refund
* **Endpoint:** `POST /pharmacy/returns`
* **Description:** Starts a return ticket. Return quantities are audited and capped to what was originally invoiced. If total refund amount exceeds **₹5,000**, status escalates to `PENDING_APPROVAL` (requires manager PIN override). Otherwise, it auto-approves.
* **Request Body:**
  | Field | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `invoice_id` | String | **Mandatory** | Original checkout bill number | Must be active invoice |
  | `items` | Array | **Mandatory** | Returns list | |
* **ReturnItem Structure:**
  * `medicine_id` (UUID, Mandatory)
  * `quantity` (Integer, Mandatory): Must be > 0.
  * `type` (String, Mandatory): `RETURN` or `EXCHANGE`.
  * `reason` (String, Mandatory): `DAMAGED`, `WRONG_DOSAGE`, `EXCESS`, `EXPIRED`, `OTHER`.
* **Success Response (`201 Created` - Auto-Approved):**
```json
{
  "success": true,
  "code": 201,
  "message": "Return approved. Stock updated successfully.",
  "data": {
    "return_id": "return-uuid-111",
    "status": "APPROVED",
    "total_refund_amount": 150.00
  }
}
```

### 8.2 Approve Return Request (Manager Override)
* **Endpoint:** `POST /pharmacy/returns/{id}/approve`
* **Required Permission:** `pharmacy:returns:override`
* **Request Body:**
  | Field | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `admin_pin` | String | **Mandatory** | Override PIN of Chief Pharmacist/Manager | Exactly 4 numeric digits |
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "message": "Return transaction approved. Stock inventory incremented."
}
```

---

## 9. Shift Operations

### 9.1 Submit Shift Handover
* **Endpoint:** `POST /pharmacy/shifts/handover`
* **Purpose:** Reconciles physical cash collected in shift drawer versus estimated dashboard billing reports. Status is marked `MISMATCHED` if values diverge.
* **Request Body:**
  | Field | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `shift_type` | String | **Mandatory** | Shift schedule segment | `MORNING`, `AFTERNOON`, `NIGHT` |
  | `incoming_pharmacist_id` | UUID | **Mandatory** | Pharmacist user receiving the keys | Valid staff user UUID |
  | `cash_drawer_counted` | Decimal | **Mandatory** | Counted drawer cash | Must be >= 0.00 |
  | `notes` | String | Optional | Operational reports | Max 1000 chars |
* **Success Response (`201 Created`):**
```json
{
  "success": true,
  "code": 201,
  "data": {
    "handover_id": "handover-uuid-777",
    "cash_drawer_expected": 12320.00,
    "cash_drawer_counted": 12320.00,
    "status": "MATCHED"
  }
}
```

### 9.2 Get Shift Handover History
* **Endpoint:** `GET /pharmacy/shifts/handover/history`
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "handover_id": "handover-uuid-777",
      "shift_date": "2026-07-13",
      "shift_type": "MORNING",
      "outgoing_pharmacist_name": "Sarah Connor",
      "incoming_pharmacist_name": "John Doe",
      "status": "MATCHED"
    }
  ]
}
```

---

## 10. Procurement Management

### 10.1 Create Supplier
* **Endpoint:** `POST /pharmacy/procurement/suppliers`
* **Request Body:**
  | Field | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `vendor_name` | String | **Mandatory** | Supplier entity name | Max 255 chars |
  | `vendor_code` | String | **Mandatory** | Unique corporate reference code | Max 50 chars, alphanumeric |
* **Success Response (`201 Created`):**
```json
{
  "success": true,
  "code": 201,
  "data": {
    "vendor_id": "vendor-uuid-555",
    "vendor_name": "Global Pharma Corp",
    "vendor_code": "VND-GLB-001"
  }
}
```

### 10.2 Create Purchase Requisition
* **Endpoint:** `POST /pharmacy/procurement/requisitions`
* **Request Body:**
  | Field | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `notes` | String | Optional | Request detail details | Max 500 chars |
  | `items` | Array | **Mandatory** | Requisition items | Min 1 item |
* **RequisitionItem Structure:**
  * `medicine_id` (UUID, Mandatory): Medicine katalog reference.
  * `quantity` (Integer, Mandatory): Must be > 0.
  * `estimated_unit_price` (Decimal, Optional): Target MRP/Unit cost projection.
* **Success Response (`201 Created`):**
```json
{
  "success": true,
  "code": 201,
  "data": {
    "requisition_id": "req-uuid-9922",
    "status": "PENDING_APPROVAL",
    "created_at": "2026-07-13T10:45:00Z"
  }
}
```

### 10.3 Approve Purchase Requisition
* **Endpoint:** `POST /pharmacy/procurement/requisitions/{id}/approve`
* **Request Body:**
  | Field | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `status` | String | **Mandatory** | Review decision outcome | `APPROVED`, `REJECTED` |
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "message": "Requisition status updated to APPROVED"
}
```

### 10.4 Create Purchase Order (PO)
* **Endpoint:** `POST /pharmacy/procurement/orders`
* **Request Body:**
  | Field | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `vendor_id` | UUID | **Mandatory** | Target supplier identifier | Must exist in DB |
  | `po_type` | String | **Mandatory** | Restock strategy | `ROUTINE_REPLENISHMENT`, `EMERGENCY_ORDER` |
  | `items` | Array | **Mandatory** | Purchase items list | |
* **POItem Structure:**
  * `medicine_id` (UUID, Mandatory)
  * `quantity` (Integer, Mandatory)
  * `unit_price` (Decimal, Mandatory)
  * `gst_percent` (Decimal, Mandatory): Tax tier (e.g. `12.00`, `18.00`).
* **Success Response (`201 Created`):**
```json
{
  "success": true,
  "code": 201,
  "data": {
    "po_id": "po-uuid-1122",
    "po_number": "PO-2026-104",
    "status": "ORDERED",
    "created_at": "2026-07-13T10:50:00Z"
  }
}
```

### 10.5 Create Goods Receipt Note (GRN)
* **Endpoint:** `POST /pharmacy/procurement/grns`
* **Request Body:**
  | Field | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `po_id` | UUID | **Mandatory** | Source Purchase Order | |
  | `vendor_id` | UUID | **Mandatory** | Executing Supplier | |
  | `items` | Array | **Mandatory** | Received goods listing | |
* **GRNItem Structure:**
  * `medicine_id` (UUID, Mandatory)
  * `received_quantity` (Integer, Mandatory)
  * `expected_quantity` (Integer, Mandatory)
  * `unit_price` (Decimal, Mandatory)
  * `batch_number` (String, Mandatory): Manufacturer batch string tag.
  * `expiry_date` (Date, Mandatory): `YYYY-MM-DD` (Must be in future).
* **Success Response (`201 Created`):**
```json
{
  "success": true,
  "code": 201,
  "data": {
    "grn_id": "grn-uuid-9900",
    "status": "DRAFT",
    "created_at": "2026-07-13T10:52:00Z"
  }
}
```

### 10.6 Verify GRN (Commit to Active Inventory)
* **Endpoint:** `POST /pharmacy/procurement/grns/{id}/verify`
* **Description:** Finalizes the GRN verification step. Commits draft values into the live inventory databases, initializing batch records and incrementing available quantities.
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "message": "GRN verified and committed. Stock updated."
}
```
