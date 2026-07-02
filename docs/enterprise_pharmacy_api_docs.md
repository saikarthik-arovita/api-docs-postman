# Enterprise Pharmacy Management System - API Documentation

This document covers the complete set of enterprise-grade endpoints, database integrations, and operations supported by the upgraded **Pharmacy Service**. 

The system operates in two core environments:
1. **Hospital Integrated mode**: Links to existing prescriptions, departments, and patients.
2. **Standalone Retail mode**: Allows walk-in customers, cash sales, and prescription-less transactions.

---

## 1. Global Conventions

### Base URL
```
https://<api-id>.execute-api.ap-south-1.amazonaws.com/<stage>
```

### Authorization Header
All requests must include a JWT Access Token:
```http
Authorization: Bearer <access_token>
```

---

## 2. API Endpoints Reference

### 2.1 Dashboard Overview

#### Get Dashboard Quick Stats
* **Endpoint**: `GET /pharmacy/dashboard/summary`
* **Response `200 OK`**:
```json
{
  "success": true,
  "data": {
    "pending_prescriptions": 15,
    "pending_dispensing": 10,
    "bills_generated_today": 32,
    "todays_sales": 48650.00,
    "cash_collected": 12320.00,
    "cash_breakdown": {
      "CASH": 12320.00,
      "CARD": 20330.00,
      "UPI": 16000.00,
      "CREDIT": 0.00
    }
  }
}
```

#### Get Critical Dashboard Alerts
* **Endpoint**: `GET /pharmacy/dashboard/alerts`
* **Description**: Returns alerts for out-of-stock items, low-stock warnings (below threshold), and batches expiring within 30 days.
* **Response `200 OK`**:
```json
{
  "success": true,
  "data": [
    {
      "id": "alert-uuid-1",
      "type": "OUT_OF_STOCK",
      "medicine_name": "Amoxicillin 500mg",
      "batch_number": "N/A",
      "current_stock": 0,
      "timestamp": "2026-06-25T10:00:00Z"
    },
    {
      "id": "alert-uuid-2",
      "type": "EXPIRY_WARNING",
      "medicine_name": "Cetirizine 10mg",
      "batch_number": "CTZ1023",
      "expiry_date": "2026-07-15",
      "timestamp": "2026-06-25T10:00:00Z"
    }
  ]
}
```

---

### 2.2 Inventory Access

#### Search & Audit Medicines
* **Endpoint**: `GET /pharmacy/medicines`
* **Query Parameters**:
  - `q` (Optional string): Match item name, barcode, or batch number.
  - `category` (Optional string): Filter by category (e.g. `TABLET`, `SYRUP`).
* **Response `200 OK`**:
```json
{
  "success": true,
  "data": [
    {
      "id": "item-uuid-123",
      "name": "Paracetamol 650mg",
      "category": "TABLET",
      "batch_number": "BAT-2026-99",
      "barcode": "890104300122",
      "price": 15.50,
      "stock_quantity": 450,
      "min_stock_level": 50,
      "expiry_date": "2027-12-31",
      "status": "IN_STOCK"
    }
  ]
}
```

---

### 2.3 Dispensing & Billing

#### Dispense OTC / Walk-in Customer Medications
* **Endpoint**: `POST /pharmacy/dispense`
* **Description**: Dispense medications directly. If `walk_in_customer` details are provided, the system creates a customer record and attributes the bill to them.
* **Request Body**:
```json
{
  "walk_in_customer": {
    "full_name": "John Doe",
    "phone": "555-0199",
    "email": "johndoe@example.com"
  },
  "items": [
    {
      "medicine_id": "medicine-master-uuid",
      "quantity": 5
    }
  ],
  "payment_mode": "CASH",
  "notes": "OTC Walk-in sale"
}
```
* **Response `201 Created`**:
```json
{
  "success": true,
  "message": "Medications dispensed successfully",
  "data": {
    "dispense_id": "dispense-uuid",
    "bill_no": "BILL-20260625-9182",
    "total_amount": 56.00,
    "items": [
      {
        "medicine_id": "medicine-master-uuid",
        "medicine_name": "Mock Paracetamol 500mg",
        "quantity": 5,
        "unit_price": 10.00,
        "gst_percent": 12.00,
        "line_subtotal": 50.00,
        "line_gst": 6.00,
        "line_total": 56.00
      }
    ]
  }
}
```

#### Dispense Hospital Prescription
* **Endpoint**: `POST /pharmacy/prescriptions/{prescription_id}/dispense`
* **Description**: Validates that the prescription exists and is signed. Allocates inventory batches automatically using **First Expired First Out (FEFO)** scheduling if no explicit `batch_id` is supplied.
* **Request Body**:
```json
{
  "patient_id": "patient-uuid-123",
  "items": [
    {
      "medicine_id": "medicine-master-uuid",
      "quantity": 2,
      "batch_id": "specific-batch-uuid"
    }
  ],
  "payment_mode": "UPI",
  "notes": "Prescription dispensing"
}
```

---

### 2.4 Returns & Exchanges

#### Request Return / Refund
* **Endpoint**: `POST /pharmacy/returns`
* **Description**: Create return requests for dispensed items.
  - Return quantities are capped to what was originally invoiced.
  - If the refund value exceeds ₹5,000, status is set to `PENDING_APPROVAL` (requires manager PIN). Otherwise, it auto-approves and updates stock.
* **Request Body**:
```json
{
  "invoice_id": "invoice-uuid-or-bill-no",
  "items": [
    {
      "medicine_id": "medicine-uuid",
      "quantity": 2,
      "type": "RETURN",
      "reason": "Excess"
    }
  ]
}
```
* **Response `200 OK` (Auto-Approved)**:
```json
{
  "success": true,
  "message": "Return approved. Stock updated successfully.",
  "data": {
    "status": "APPROVED",
    "total_refund_amount": 22.40,
    "returns": [
      {
        "id": "return-uuid",
        "invoice_id": "invoice-uuid",
        "status": "APPROVED",
        "total_refund_amount": 22.40
      }
    ]
  }
}
```

#### Approve Return Request (Manager Override PIN)
* **Endpoint**: `POST /pharmacy/returns/{id}/approve`
* **Description**: Overrides high-value returns. Validates PIN and updates inventory stock.
* **Request Body**:
```json
{
  "admin_pin": "1234"
}
```

---

### 2.5 Patient Assistance

#### Schedule Refill Reminder
* **Endpoint**: `POST /pharmacy/patients/{patient_id}/timeline/reminders`
* **Request Body**:
```json
{
  "reminder_date": "2026-07-25",
  "frequency_days": 30,
  "medicine_name": "Paracetamol 500mg",
  "message_template": "Reminder to collect your monthly refill"
}
```

---

### 2.6 Daily Operations

#### Submit Shift Handover
* **Endpoint**: `POST /pharmacy/shifts/handover`
* **Description**: Reconciles physical cash against expected daily billing sales.
* **Request Body**:
```json
{
  "shift_type": "MORNING",
  "incoming_pharmacist_id": "incoming-user-uuid",
  "cash_drawer_counted": 56.00,
  "notes": "Shift ended. Reconciled register."
}
```
* **Response `201 Created`**:
```json
{
  "success": true,
  "data": {
    "id": "handover-uuid",
    "cash_drawer_expected": 56.00,
    "status": "MATCHED"
  }
}
```

#### Get Shift Handover History
* **Endpoint**: `GET /pharmacy/shifts/handover/history`

---

---

### 2.7 Dispensing Cart Engine

#### Create a Dispensing Cart
* **Endpoint**: `POST /pharmacy/carts`
* **Description**: Initializes a new dispensing cart, optionally linked to a hospital patient, prescription, or walk-in customer. Reserved stock automatically expires after 15 minutes.
* **Request Body**:
```json
{
  "cart_type": "WALK_IN",
  "walk_in_customer": {
    "full_name": "Alice Smith",
    "phone": "999-999-9999",
    "email": "alice@example.com"
  }
}
```

#### Get Cart Details
* **Endpoint**: `GET /pharmacy/carts/{id}`
* **Description**: Returns cart header, reserved items, pricing, and picking instructions (warehouse location, rack/shelf/bin details).

#### Add Item to Cart (Reserve Stock)
* **Endpoint**: `POST /pharmacy/carts/{id}/items`
* **Description**: Adds medicine to the cart and reserves the specified quantity. Allocates batches using FEFO by default, or allows a manual batch override.
* **Request Body**:
```json
{
  "medicine_id": "medicine-master-uuid",
  "quantity": 2,
  "batch_id": "optional-batch-override-uuid"
}
```

#### Verify Cart (Clinical Checks)
* **Endpoint**: `POST /pharmacy/carts/{id}/verify`
* **Description**: Runs clinical checks (allergies, duplicate therapy, drug interaction, expired batches) on cart items.

#### Checkout Cart (Billing & Dispensing)
* **Endpoint**: `POST /pharmacy/carts/{id}/checkout`
* **Description**: Finalizes checkout, processes payment, creates billing invoice, actualizes stock deductions, and dispenses the medications.
* **Request Body**:
```json
{
  "payment_mode": "UPI"
}
```

#### Clean Expired Carts
* **Endpoint**: `POST /pharmacy/carts/cleanup`
* **Description**: Checks for active carts past their expiry time, releases their stock reservations (updating `reserved_quantity`), and marks the carts as `EXPIRED`.

---

### 2.8 Procurement Lifecycle

#### Create Supplier/Vendor
* **Endpoint**: `POST /pharmacy/procurement/suppliers`
* **Request Body**:
```json
{
  "vendor_name": "Global Pharma Corp",
  "vendor_code": "VND-GLB-001"
}
```

#### Create Purchase Requisition
* **Endpoint**: `POST /pharmacy/procurement/requisitions`
* **Request Body**:
```json
{
  "notes": "Restocking Paracetamol",
  "items": [
    {
      "item_id": "inventory-item-uuid",
      "item_name": "Paracetamol 500mg",
      "quantity": 50,
      "estimated_unit_price": 10.00
    }
  ]
}
```

#### Approve Purchase Requisition
* **Endpoint**: `POST /pharmacy/procurement/requisitions/{id}/approve`
* **Request Body**:
```json
{
  "status": "APPROVED"
}
```

#### Create Purchase Order (PO)
* **Endpoint**: `POST /pharmacy/procurement/orders`
* **Request Body**:
```json
{
  "vendor_id": "vendor-uuid",
  "po_type": "ROUTINE_REPLENISHMENT",
  "items": [
    {
      "item_id": "inventory-item-uuid",
      "item_name": "Paracetamol 500mg",
      "final_qty": 50,
      "unit_price": 8.00,
      "gst_percent": 12.00
    }
  ]
}
```

#### Create Goods Receipt Note (GRN)
* **Endpoint**: `POST /pharmacy/procurement/grns`
* **Request Body**:
```json
{
  "po_id": "po-uuid",
  "vendor_id": "vendor-uuid",
  "items": [
    {
      "item_id": "inventory-item-uuid",
      "item_name": "Paracetamol 500mg",
      "received_qty": 50,
      "expected_qty": 50,
      "unit_price": 8.00,
      "batch_number": "BATCH-PROC-001",
      "expiry_date": "2027-06-25"
    }
  ]
}
```

#### Verify GRN (Commit to Stock)
* **Endpoint**: `POST /pharmacy/procurement/grns/{id}/verify`
* **Description**: Commits received goods to inventory stock, creating batches and updating quantities in the system.

---

## 3. Concurrency Safeguards
To prevent race conditions during heavy dispensing periods, database operations run inside explicit transactions utilizing PostgreSQL **Row-Level locking** (`SELECT FOR UPDATE`) on target rows of `pharmacy.inventory_items` and `pharmacy.inventory_batches`.

