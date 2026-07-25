# Admin Inventory & Procurement API Documentation

This document provides specifications for the **Inventory Control, Supplier Management, Requisitioning, and Procurement Workflows** within the `hms-admin` microservice.

---

## Global Environment & Configuration
* **Base URL**: `https://vo585bjv4a.execute-api.ap-south-1.amazonaws.com/dev`
* **Authentication**: Bearer Token (Cognito / HMS JWT Access Token)

---

## 1. Inventory Management

### 1.1 GET List Medical Inventory
Retrieve a list of medical items (medicines, consumables) in stock.

* **Method**: `GET`
* **URL**: `/admin/inventory/medical`
* **Query Parameters**:
  * `page` (*Optional*, Default: `1`): Current page offset.
  * `page_size` (*Optional*, Default: `20`): Items limit.
  * `search` (*Optional*): Search term matched against medicine name.
  * `low_stock` (*Optional*): `true` to filter items below safety stock.

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "id": "aa30b4b7-0da8-4b9d-8643-4fdffefabe9a",
        "name": "Paracetamol 500mg",
        "sku_code": "MED-PAR-500",
        "batch_count": 3,
        "total_quantity": 450,
        "safety_stock": 100,
        "unit": "Tablet",
        "is_active": true
      }
    ],
    "total": 120
  }
}
```

---

### 1.2 POST Add Medical Inventory
Add a new medical medicine item to the inventory master database.

* **Method**: `POST`
* **URL**: `/admin/inventory/medical`

#### Request Body
```json
{
  "name": "Amoxicillin Capsule 250mg",
  "sku_code": "MED-AMX-250",
  "safety_stock": 150,
  "unit": "Capsule",
  "description": "Broad-spectrum antibiotic"
}
```

#### Response Body (`201 Created`)
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "bb30b4b7-0da8-4b9d-8643-4fdffefabe9b",
    "name": "Amoxicillin Capsule 250mg",
    "sku_code": "MED-AMX-250",
    "safety_stock": 150,
    "unit": "Capsule",
    "is_active": true,
    "created_at": "2026-07-25T11:55:00.000Z"
  }
}
```

---

### 1.3 GET List Stock Transfers
Retrieve a list of inter-departmental inventory transfers.

* **Method**: `GET`
* **URL**: `/admin/inventory/transfers`
* **Query Parameters**:
  * `status` (*Optional*): Filter by transfer status (`PENDING`, `APPROVED`, `REJECTED`).

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "id": "t1-aa30b4b7-0da8-4b9d-8643-4fdffefabe9a",
        "source_department_id": "0fbfdbef-9873-4824-b1fd-6fb827d3ba57",
        "source_department_name": "Main Store",
        "destination_department_id": "f1cb936c-1d4f-47da-8de6-a48ba04a4a81",
        "destination_department_name": "Emergency Ward",
        "item_name": "Paracetamol 500mg",
        "quantity": 100,
        "status": "PENDING",
        "created_at": "2026-07-25T11:45:00.000Z"
      }
    ],
    "total": 5
  }
}
```

---

### 1.4 POST Approve Stock Transfer
Commit stock adjustment for a pending transfer.

* **Method**: `POST`
* **URL**: `/admin/inventory/transfers/{transfer_id}/approve`

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "message": "Stock transfer approved and inventory moved successfully",
  "data": {
    "id": "t1-aa30b4b7-0da8-4b9d-8643-4fdffefabe9a",
    "status": "APPROVED"
  }
}
```

---

## 2. Procurement Workflows

### 2.1 GET List Purchase Orders
Retrieve a paginated list of purchase orders.

* **Method**: `GET`
* **URL**: `/admin/procurement/orders`
* **Query Parameters**:
  * `page` (*Optional*, Default: `1`): Current page.
  * `page_size` (*Optional*, Default: `10`): Items limit.
  * `search` (*Optional*): Search string.
  * `status` (*Optional*): Filter by PO status (`PENDING`, `APPROVED`, `REJECTED`).

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "id": "po-ffeee23d-9a29-454f-9f46-192f1aaab285",
        "po_number": "PO-2026-0001",
        "supplier_name": "MedLife Pharma",
        "order_date": "2026-07-15",
        "expected_delivery": "2026-07-22",
        "total_amount": 54000.00,
        "status": "APPROVED"
      }
    ],
    "total": 12
  }
}
```

---

### 2.2 POST Create Purchase Order
Initiate a new purchase order.

* **Method**: `POST`
* **URL**: `/admin/procurement/orders`

#### Request Body
```json
{
  "supplier_id": "a901f41b-4fef-48a1-b84e-8664fd37a912",
  "expected_delivery": "2026-08-01",
  "items": [
    {
      "medicine_id": "aa30b4b7-0da8-4b9d-8643-4fdffefabe9a",
      "quantity": 500,
      "unit_price": 10.00
    }
  ]
}
```

#### Response Body (`201 Created`)
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "po-ffeee23d-9a29-454f-9f46-192f1aaab285",
    "po_number": "PO-2026-0002",
    "supplier_id": "a901f41b-4fef-48a1-b84e-8664fd37a912",
    "order_date": "2026-07-25",
    "expected_delivery": "2026-08-01",
    "total_amount": 5000.00,
    "status": "PENDING"
  }
}
```

---

### 2.3 POST Receive GRN (Goods Received Note)
Record the arrival of ordered items at the store.

* **Method**: `POST`
* **URL**: `/admin/procurement/orders/{po_id}/grns`

#### Request Body
```json
{
  "delivery_challan_number": "CHL-9988",
  "received_items": [
    {
      "medicine_id": "aa30b4b7-0da8-4b9d-8643-4fdffefabe9a",
      "received_quantity": 500,
      "accepted_quantity": 490,
      "rejected_quantity": 10,
      "rejection_reason": "Damaged strip packages"
    }
  ]
}
```

#### Response Body (`201 Created`)
```json
{
  "success": true,
  "code": 201,
  "data": {
    "grn_id": "grn-9988-aa30b4b7-0da8-4de6a48ba",
    "po_id": "po-ffeee23d-9a29-454f-9f46-192f1aaab285",
    "status": "PENDING_VERIFICATION",
    "created_at": "2026-07-25T12:00:00.000Z"
  }
}
```
