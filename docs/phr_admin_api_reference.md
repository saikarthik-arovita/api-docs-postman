# Enterprise Pharmacy (PHR Admin) API Reference

This document provides a comprehensive API specification for the **Enterprise Pharmacy (PHR Admin)** module as shown in the 14 UI design screens.

> **Base URL**: `{{pharmacy_base_url}}` (or `{{admin_base_url}}` for global settings/roles)  
> **Auth**: `Authorization: Bearer <access_token>`  
> **Tenant**: `x-tenant-id: <branch_id>` (required header)

---

## 1. Dashboard & Analytics (Screens 1, 3, 4, 11)

### `GET /pharmacy/dashboard/summary`
Returns the overview metadata of the pharmacy dashboard.
* **Query Params**:
  * `tab` (string, optional) — filter category (e.g. `all`, `ipd`, `opd`)
* **Response `200 OK`**:
  ```json
  {
    "success": true,
    "data": {
      "today_dispensed": 142,
      "pending_prescriptions": 12,
      "low_stock_alerts": 3,
      "revenue_today": 45000.00
    }
  }
  ```

### `GET /pharmacy/dashboard/kpis`
Returns the core KPI metrics shown on the pharmacy landing page.
* **Query Params**:
  * `tab` (string, optional) — filter category (e.g. `all`, `ipd`, `opd`, `walkin`)
* **Response `200 OK`**:
  ```json
  {
    "success": true,
    "data": {
      "active_prescriptions": 24,
      "ipd": 8,
      "opd": 12,
      "walk_in": 4
    }
  }
  ```

### `GET /pharmacy/dashboard/alerts`
Returns active operational/stock alerts requiring immediate attention.
* **Response `200 OK`**:
  ```json
  {
    "success": true,
    "data": [
      {
        "alert_id": "alert-uuid",
        "title": "Low Stock: Amlodipine 5mg",
        "severity": "CRITICAL",
        "message": "Stock level (5 units) is below reorder level (50 units).",
        "timestamp": "2026-07-16T12:00:00Z"
      }
    ]
  }
  ```

### `GET /pharmacy/dashboard/activity`
Returns the chronological activity feed/audit trail.
* **Query Params**:
  * `limit` (int, optional, default: 10, max: 100)
  * `offset` (int, optional, default: 0)
* **Response `200 OK`**:
  ```json
  {
    "success": true,
    "data": [
      {
        "id": "activity-uuid",
        "action": "PRESCRIPTION_DISPENSED",
        "actor_id": "user-uuid",
        "entity_type": "PRESCRIPTION",
        "entity_id": "presc-uuid",
        "timestamp": "2026-07-16T15:20:00Z",
        "status": "SUCCESS",
        "new_data": { "bill_no": "BILL-2026-0421", "amount": 1250.0 }
      }
    ]
  }
  ```

---

## 2. Prescription Management (Screen 2)

### `GET /pharmacy/prescriptions/queue`
Retrieves prescriptions in the queue. Powers the main Prescription Grid.
* **Query Params**:
  * `status` (string, optional) — `'SIGNED'`, `'DISPENSED'`, `'CANCELLED'`, or `'DRAFT'`
  * `search` (string, optional) — search keyword for patient name, phone, or UHID
* **Response `200 OK`**:
  ```json
  {
    "success": true,
    "data": [
      {
        "id": "presc-uuid",
        "version_no": 1,
        "prescription_type": "REGULAR",
        "encounter_context": "OPD-1024",
        "status": "SIGNED",
        "created_at": "2026-07-16T14:15:00Z",
        "patient_id": "patient-uuid",
        "patient_name": "Suresh Raina",
        "uhid": "UHID-10042",
        "doctor_id": "doctor-uuid",
        "doctor_name": "Dr. Ramesh Verma",
        "source_module": "OPD"
      }
    ]
  }
  ```

### `POST /pharmacy/prescriptions/{id}/dispense`
Dispenses medicines against a prescription.
* **Path Params**:
  * `id` — Prescription ID
* **Request Body**:
  ```json
  {
    "notes": "Dispensed full course.",
    "items": [
      {
        "medicine_id": "med-uuid",
        "batch_id": "batch-uuid",
        "quantity_dispensed": 10
      }
    ]
  }
  ```
* **Response `200 OK`**:
  ```json
  {
    "success": true,
    "data": {
      "dispense_id": "dispense-uuid",
      "prescription_id": "presc-uuid",
      "bill_no": "BILL-2026-0425",
      "status": "DISPENSED"
    }
  }
  ```

---

## 3. Patient & Customer Registry (Screen 9)

### `POST /pharmacy/customers`
Registers a walk-in customer (patient not admitted/enrolled in the hospital system).
* **Request Body**:
  ```json
  {
    "full_name": "Aman Gupta",
    "phone": "9988776655",
    "email": "aman@gmail.com",
    "gender": "MALE",
    "age": 34
  }
  ```
* **Response `201 Created`**:
  ```json
  {
    "success": true,
    "data": {
      "id": "walkin-customer-uuid",
      "full_name": "Aman Gupta",
      "phone": "9988776655",
      "uhid": "WALKIN-G-00124"
    }
  }
  ```

### `GET /pharmacy/patients`
Search registry patients and walk-in customers.
* **Query Params**:
  * `q` (string, optional) — search name, phone, or UHID
  * `walkin_only` (bool, optional) — set to `true` to only fetch walk-in customers
* **Response `200 OK`**:
  ```json
  {
    "success": true,
    "data": [
      {
        "id": "patient-uuid",
        "full_name": "Aman Gupta",
        "phone": "9988776655",
        "uhid": "WALKIN-G-00124",
        "is_walkin": true
      }
    ]
  }
  ```

---

## 4. Inventory & Warehouse Management (Screens 5, 8, 11)

### `GET /pharmacy/inventory/items`
Lists inventory items with stock levels.
* **Query Params**:
  * `search` (string, optional) — search item name or code
  * `status` (string, optional) — `'HEALTHY'`, `'LOW_STOCK'`, `'OUT_OF_STOCK'`
* **Response `200 OK`**:
  ```json
  {
    "success": true,
    "data": [
      {
        "id": "item-uuid",
        "item_name": "Amlodipine 5mg",
        "item_code": "MED-AML-005",
        "current_stock": 250,
        "minimum_stock": 50,
        "mrp": 12.50,
        "status_level": "HEALTHY"
      }
    ]
  }
  ```

### `GET /pharmacy/inventory/batches`
Fetches active stock batches of a specific medicine.
* **Query Params**:
  * `medicine_id` (string, required)
* **Response `200 OK`**:
  ```json
  {
    "success": true,
    "data": [
      {
        "batch_id": "batch-uuid",
        "batch_no": "B-AML-102",
        "expiry_date": "2027-08-31",
        "quantity": 100,
        "mrp": 12.50
      }
    ]
  }
  ```

### `POST /pharmacy/inventory/transfer`
Logs stock transfers between floor warehouses/racks.
* **Request Body**:
  ```json
  {
    "source_warehouse_id": "wh-source-uuid",
    "dest_warehouse_id": "wh-dest-uuid",
    "medicine_id": "med-uuid",
    "batch_id": "batch-uuid",
    "quantity": 50
  }
  ```
* **Response `200 OK`**:
  ```json
  {
    "success": true,
    "message": "Stock transfer recorded successfully."
  }
  ```

---

## 5. Procurement & Vendor Directory (Screens 6, 8, 12)

### `POST /pharmacy/procurement/suppliers`
Adds a new supplier/vendor.
* **Request Body**:
  ```json
  {
    "name": "Cipla Pharmaceuticals",
    "contact_name": "Rajesh Shah",
    "phone": "9876543210",
    "email": "contact@cipla.com",
    "address": "Mumbai, India",
    "tax_number": "GSTIN12345"
  }
  ```
* **Response `201 Created`**:
  ```json
  {
    "success": true,
    "data": {
      "id": "vendor-uuid",
      "name": "Cipla Pharmaceuticals",
      "status": "ACTIVE"
    }
  }
  ```

### `POST /pharmacy/procurement/orders`
Creates a Purchase Order (PO).
* **Request Body**:
  ```json
  {
    "vendor_id": "vendor-uuid",
    "delivery_expected_by": "2026-08-15",
    "items": [
      {
        "medicine_id": "med-uuid",
        "quantity": 1000,
        "unit_cost": 8.50
      }
    ]
  }
  ```
* **Response `201 Created`**:
  ```json
  {
    "success": true,
    "data": {
      "po_id": "po-uuid",
      "po_number": "PO-2026-0012",
      "status": "PENDING"
    }
  }
  ```

### `POST /pharmacy/procurement/grns`
Generates a Goods Receipt Note (GRN) on delivery.
* **Request Body**:
  ```json
  {
    "po_id": "po-uuid",
    "invoice_no": "INV-10024",
    "items_received": [
      {
        "medicine_id": "med-uuid",
        "quantity_received": 1000,
        "batch_no": "B-NEW-01",
        "expiry_date": "2028-01-31"
      }
    ]
  }
  ```
* **Response `201 Created`**:
  ```json
  {
    "success": true,
    "data": {
      "grn_id": "grn-uuid",
      "grn_number": "GRN-2026-0042",
      "status": "VERIFIED"
    }
  }
  ```

---

## 6. Billing & Returns (Screen 10)

### `GET /pharmacy/billing/invoices`
Lists all pharmacy invoices/billing records.
* **Query Params**:
  * `patient_id` (string, optional)
  * `search` (string, optional) — invoice number or patient details
* **Response `200 OK`**:
  ```json
  {
    "success": true,
    "data": [
      {
        "invoice_id": "inv-uuid",
        "bill_no": "BILL-2026-0042",
        "patient_name": "Ravi Kumar",
        "total_amount": 1250.00,
        "discount_amount": 50.00,
        "gst_amount": 100.00,
        "payment_mode": "UPI",
        "created_at": "2026-07-16T10:00:00Z"
      }
    ]
  }
  ```

### `GET /pharmacy/returns`
Lists and searches return/exchange requests.
* **Query Params**:
  * `q` (string, optional) — search keyword
  * `status` (string, optional) — `'PENDING_APPROVAL'`, `'APPROVED'`, `'REJECTED'`
* **Response `200 OK`**:
  ```json
  {
    "success": true,
    "data": [
      {
        "id": "return-uuid",
        "bill_no": "BILL-2026-0042",
        "patient_name": "Ravi Kumar",
        "medicine_name": "Amlodipine 5mg",
        "return_quantity": 5,
        "refund_amount": 62.50,
        "status": "PENDING_APPROVAL"
      }
    ]
  }
  ```

---

## 7. Reports & Analytics (Screen 13)

### `GET /pharmacy/reports`
Lists reports available in the reports library along with their current statistics.
* **Response `200 OK`**:
  ```json
  {
    "success": true,
    "data": [
      {
        "report_type": "DAILY_PHARMACY",
        "title": "Daily Pharmacy Report",
        "description": "Consolidated summary of pharmacy operations",
        "metrics": {
          "sales": 45000.0,
          "prescriptions": 142,
          "transactions": 125
        },
        "last_generated": "2026-07-16T17:30:00Z"
      },
      {
        "report_type": "INVENTORY",
        "title": "Inventory Report",
        "description": "Stock levels, valuation, and warehouse summary",
        "metrics": {
          "current_stock": 25000,
          "low_stock_items": 4,
          "inv_value": 312500.0
        },
        "last_generated": "2026-07-16T17:30:00Z"
      }
    ]
  }
  ```

### `POST /pharmacy/reports/generate`
Triggers generation of a report.
* **Request Body**:
  ```json
  {
    "report_type": "DAILY_PHARMACY"  // or "INVENTORY", "SALES", "PRESCRIPTION"
  }
  ```
* **Response `200 OK`**:
  ```json
  {
    "success": true,
    "data": {
      "report_type": "DAILY_PHARMACY",
      "status": "GENERATED",
      "generated_at": "2026-07-16T17:35:00Z",
      "metrics": {
        "sales": 45120.0,
        "prescriptions": 143,
        "transactions": 126
      },
      "download_url": "/pharmacy/reports/download/daily-pharmacy"
    }
  }
  ```

---

## 8. Roles & Permissions Configuration (Screen 14)

> **Base URL**: `/admin` (identity/access control endpoints)

### `GET /admin/roles`
Lists all global system user roles.
* **Query Params**:
  * `role_category` (string, optional) — filter category (e.g. `'PHARMACY'`, `'IT'`)
* **Response `200 OK`**:
  ```json
  {
    "success": true,
    "data": [
      {
        "roleId": "PHA-01",
        "roleName": "Pharmacist",
        "description": "Handles dispensing and stock checks",
        "roleStatus": "Active"
      }
    ]
  }
  ```

### `GET /admin/roles/{role_id}/permissions`
Retrieves granular permission lists (grouped by functional categories) for the selected role.
* **Path Params**:
  * `role_id` — Role ID
* **Response `200 OK`**:
  ```json
  {
    "success": true,
    "data": {
      "roleId": "PHA-01",
      "categories": [
        {
          "categoryName": "Pharmacy",
          "permissions": [
            {
              "permissionId": "PHA-001",
              "permissionName": "pharmacy:dispense",
              "description": "Dispense medicines against prescription",
              "enabled": true,
              "criticalPermission": false
            }
          ]
        }
      ]
    }
  }
  ```

### `PATCH /admin/roles/{role_id}/permissions`
Updates the permissions checklist for a role in draft state.
* **Request Body**:
  ```json
  {
    "permissionIds": ["PHA-001", "PHA-002", "INV-001"]
  }
  ```
* **Response `200 OK`**:
  ```json
  {
    "success": true,
    "message": "Permissions updated in draft state."
  }
  ```

### `POST /admin/roles/{role_id}/publish`
Publishes and activates the draft permission configuration.
* **Response `200 OK`**:
  ```json
  {
    "success": true,
    "data": {
      "roleId": "PHA-01",
      "roleName": "Pharmacist"
    }
  }
  ```

### `POST /admin/roles/{role_id}/discard`
Discards the current draft configurations.
* **Response `200 OK`**:
  ```json
  {
    "success": true,
    "message": "Draft permission modifications discarded."
  }
  ```
