# Master Admin Inventory & Procurement API Documentation (Frontend Integration Guide)

This document is the **master integration reference guide for Frontend (FE) Developers** implementing the **Inventory & Supply Chain Dashboard** in the HMS application.

It covers all 4 primary sub-tabs (**Inventory Overview**, **Procurement**, **Vendor Management**, and **Asset Management**), as well as header modal actions (`+ Create Item`, `+ Add Stock`, `+ Transfer Stock`, `+ Create PO`, `+ Create Vendor`), side drawers, modals, filter panels, document previews, and action workflows.

---

## 1. Global API & Technical Environment Configuration

* **Microservice Name**: `hms-admin` (`services/admin`)
* **Base URL**: `https://vo585bjv4a.execute-api.ap-south-1.amazonaws.com/dev`
* **Protocol**: HTTPS / REST JSON
* **Content-Type**: `application/json`
* **Authentication Header**: `Authorization: Bearer <COGNITO_JWT_ACCESS_TOKEN>`
* **Tenant & Branch Headers**:
  * `X-Tenant-ID`: `0fbfdbef-9873-4824-b1fd-6fb827d3ba57` (UUID)
  * `X-Branch-ID`: `f1cb936c-1d4f-47da-8de6-a48ba04a4a81` (UUID)
* **Required RBAC Permissions**:
  * System Admin: `ADM-001` or `SYS-001`
  * Store Manager / Purchase Agent: `PHM-004`

---

## 2. Header Modal Actions (Top Bar Buttons)

### 2.1 `+ Add Item` / `+ Create Item` (`POST /admin/inventory/medical`)
Creates a new medical or non-medical inventory master record.

* **Method**: `POST`
* **URL Path**: `/admin/inventory/medical`
* **Handler**: `inventory_svc.create_medical_inventory()`

#### Request Body Schema
| Field | Type | Mandatory | Constraints | Description |
| :--- | :--- | :--- | :--- | :--- |
| `item_name` | `string` | Mandatory | Max 200 chars | Item commercial name (e.g. `Surgical Gloves`). |
| `sku_code` | `string` | Mandatory | Max 60 chars | SKU Code (e.g. `SKU-012456`). |
| `category` | `string` | Mandatory | Max 60 chars | Category (e.g. `OT Consumables`, `Pharmacy`). |
| `medicine_type` | `string` | Optional | Max 60 chars | `TABLET`, `CAPSULE`, `INJECTION`, `CONSUMABLE`. |
| `unit` | `string` | Mandatory | Max 20 chars | Unit of measure (`Units`, `Boxes`, `Pairs`, `Pcs`). |
| `reorder_level` | `integer` | Mandatory | Min: 0 | Safety reorder threshold (e.g. `500`). |
| `opening_balance` | `integer` | Optional | Min: 0 | Initial stock balance. |
| `quantity_received` | `integer` | Optional | Min: 0 | Quantity received in initial batch. |
| `batch_number` | `string` | Optional | Max 60 chars | Batch number (e.g. `BG-2026-12`). |
| `expiry_date` | `string` | Optional | YYYY-MM-DD | Batch expiry date (e.g. `2028-01-31`). |

```json
{
  "item_name": "Surgical Gloves",
  "sku_code": "SKU-012456",
  "category": "OT Consumables",
  "medicine_type": "CONSUMABLE",
  "unit": "Units",
  "reorder_level": 500,
  "opening_balance": 1000,
  "quantity_received": 460,
  "batch_number": "BG-2026-12",
  "expiry_date": "2028-01-31"
}
```

#### Response Body (`201 Created`)
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "item-99281-a82",
    "item_name": "Surgical Gloves",
    "sku_code": "SKU-012456",
    "category": "OT Consumables",
    "unit": "Units",
    "current_stock": 1460,
    "reorder_level": 500,
    "status_level": "HEALTHY",
    "is_active": true,
    "created_at": "2026-04-20T10:00:00.000Z"
  }
}
```

---

### 2.2 `+ Transfer Stock` / Department Requisition (`POST /admin/inventory/transfers`)
Transfers stock between warehouses or departments (enforces FEFO logic).

* **Method**: `POST`
* **URL Path**: `/admin/inventory/transfers`
* **Handler**: `inventory_svc.create_transfer()`

#### Request Body Schema
| Field | Type | Mandatory | Description |
| :--- | :--- | :--- | :--- |
| `item_id` | `string (UUID)` | Mandatory | Target item UUID. |
| `batch_id` | `string (UUID)` | Optional | Specific batch UUID. |
| `source_department_id` | `string (UUID)` | Mandatory | Source department/warehouse UUID. |
| `destination_department_id` | `string (UUID)` | Mandatory | Destination department UUID. |
| `quantity` | `integer` | Mandatory | Quantity to transfer (Min: 1). |
| `transfer_reason` | `string` | Mandatory | Justification note. |

```json
{
  "item_id": "a901f41b-4fef-48a1-b84e-8664fd37a912",
  "batch_id": "b101-aa-bb",
  "source_department_id": "0fbfdbef-9873-4824-b1fd-6fb827d3ba57",
  "destination_department_id": "f1cb936c-1d4f-47da-8de6-a48ba04a4a81",
  "quantity": 120,
  "transfer_reason": "Emergency OT Ward Stock Replenishment"
}
```

#### Response Body (`201 Created`)
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "t1-aa30b4b7-0da8-4b9d-8643-4fdffefabe9a",
    "item_id": "a901f41b-4fef-48a1-b84e-8664fd37a912",
    "quantity": 120,
    "status": "COMPLETED",
    "created_at": "2026-04-20T10:30:00.000Z"
  }
}
```

---

## 3. Screen 1: Inventory Overview Tab

### 3.1 Overview KPI Summary Cards (`GET /admin/inventory/warehouse/overview`)
* **Method**: `GET`
* **URL Path**: `/admin/inventory/warehouse/overview`
* **Handler**: `inventory_svc.get_warehouse_overview()`

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "total_inventory_value": {
      "amount": 2486450.00,
      "currency": "INR",
      "formatted": "₹24,86,450",
      "percentage_change": 12.5,
      "trend": "UP"
    },
    "total_stock_items": {
      "count": 98340,
      "formatted": "98,340",
      "percentage_change": 3.2,
      "trend": "UP"
    },
    "low_stock_items": {
      "count": 26,
      "critical_count": 14,
      "subtitle": "14 Critical items",
      "status_severity": "WARNING"
    },
    "near_expiry_items": {
      "count": 142,
      "action_required_count": 34,
      "subtitle": "34 Action required",
      "status_severity": "CRITICAL"
    }
  }
}
```

---

### 3.2 Warehouse Inventory Table & Filter Drawer (`GET /admin/inventory`)
* **Method**: `GET`
* **URL Path**: `/admin/inventory`
* **Handler**: `inventory_svc.list_inventory()`

#### Query Parameters
| Parameter | Type | Required | Allowed Enums / Values | Description |
| :--- | :--- | :--- | :--- | :--- |
| `search` | `string` | Optional | Free text | Matches Item Name, SKU Code (`012456`), or Order ID (`OD-102456`). |
| `item_type` | `string` | Optional | `MEDICAL`, `NON_MEDICAL` | Classification filter. |
| `category` | `string` | Optional | `OT Consumables`, `Pharmacy`, `Laboratory`, `House Keeping`, `Radiology` | Category filter. |
| `movement_status` | `string` | Optional | `STABLE`, `CRITICAL`, `FAST_MOVING`, `SLOW_MOVING` | Movement status filter. |
| `status_level` | `string` | Optional | `HEALTHY`, `LOW_STOCK`, `CRITICAL` | Stock health status level. |
| `page` | `integer` | Optional | Min: `1` (Default: `1`) | Pagination page number. |
| `page_size` | `integer` | Optional | 1–100 (Default: `10`) | Page size limit. |

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "id": "item-99281-a82",
        "order_id": "OD-102456",
        "sku_code": "SKU: 012456",
        "item_name": "Surgical Gloves",
        "category": "OT Consumables",
        "department": { "id": "dept-01", "name": "Pharmacy" },
        "warehouse_location": { "id": "loc-c1", "name": "Central Storage" },
        "stock": {
          "available_units": 1240,
          "reserved_units": 220,
          "total_units": 1460,
          "unit_of_measure": "Units"
        },
        "batch": {
          "batch_number": "BG-2026-12",
          "expiry_date": "2028-01-31",
          "formatted_expiry": "Jan 2028"
        },
        "movement_status": "STABLE"
      }
    ],
    "meta": { "total": 124, "page": 1, "page_size": 10, "total_pages": 13 }
  }
}
```

---

### 3.3 Item Details Drawer & Activity Log (`GET /admin/inventory/{item_id}`)
* **Method**: `GET`
* **URL Path**: `/admin/inventory/{item_id}`
* **Handler**: `inventory_svc.get_inventory_item()`

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "item-99281-a82",
    "order_id": "OD-102456",
    "sku_code": "SKU: 012456",
    "item_name": "Surgical Gloves",
    "category": "OT Consumables",
    "movement_status": "STABLE",
    "department": "Pharmacy",
    "warehouse_location": "Central Storage",
    "batch_number": "BG-2026-12",
    "available_units": 1240,
    "reserved_units": 220,
    "expiry_date": "2028-01-31",
    "formatted_expiry": "Jan 2028",
    "recent_activity": [
      {
        "id": "act-101",
        "title": "Physical stock audit completed",
        "performed_by": "Frank James",
        "formatted_timestamp": "Oct 04, 2023 • 10:17 AM",
        "activity_type": "AUDIT"
      },
      {
        "id": "act-102",
        "title": "120 Units reserved for Emergency OT",
        "performed_by": "Dr. Robert Chen",
        "formatted_timestamp": "Oct 04, 2023 • 09:42 PM",
        "activity_type": "RESERVATION"
      }
    ]
  }
}
```

---

## 4. Screen 2: Procurement Tab

### 4.1 Procurement Dashboard KPI Insights (`GET /admin/procurement/insights`)
* **Method**: `GET`
* **URL Path**: `/admin/procurement/insights`
* **Handler**: `procurement_svc.get_procurement_insights()`

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "metrics": {
      "total_purchase_orders": { "count": 1284, "formatted": "1,284", "percentage_change": 12.4, "trend": "UP" },
      "procurement_value": { "amount": 4836500.00, "formatted": "₹48,36,500", "currency": "INR", "percentage_change": 18.5, "trend": "UP" },
      "pending_purchase_orders": { "count": 87, "formatted": "87", "overdue_count": 14, "on_track_count": 73, "status_severity": "WARNING" },
      "active_suppliers": { "count": 134, "formatted": "134", "percentage_change": 4.0, "trend": "UP" }
    }
  }
}
```

---

### 4.2 Purchase Orders List & Filters (`GET /admin/procurement/orders`)
* **Method**: `GET`
* **URL Path**: `/admin/procurement/orders`
* **Handler**: `procurement_svc.list_purchase_orders()`

#### Query Parameters
| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `search` | `string` | Optional | Search string for PO ID (`PO-2045`), Item (`Lidocaine`), or Vendor. |
| `status` | `string` | Optional | `Pending`, `Processing`, `Approved`, `Rejected`. |
| `vendor_id` | `string (UUID)` | Optional | Supplier UUID. |
| `page` | `integer` | Optional (Default: `1`) | Page number. |
| `page_size` | `integer` | Optional (Default: `10`) | Limit per page. |

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "id": "po-2045-uuid-01",
        "po_number": "PO-2045",
        "item_name": "Lidocaine Packet 5cl",
        "vendor_name": "MediPlus Pharma",
        "vendor_category": "Primary Medical Dist.",
        "category": "Procedure Medication",
        "quantity_units": 1240,
        "reserved_units": 220,
        "delivery_eta": "Tomorrow",
        "invoice_status": "Awaiting Submission",
        "approval_status": "Pending",
        "total_amount": 45000.00,
        "created_at": "2026-04-19T10:00:00.000Z"
      }
    ],
    "meta": { "total": 124, "page": 1, "page_size": 10, "total_pages": 13 }
  }
}
```

---

### 4.3 Purchase Order Details Modal (`GET /admin/procurement/orders/{po_id}`)
* **Method**: `GET`
* **URL Path**: `/admin/procurement/orders/{po_id}`
* **Handler**: `procurement_svc.get_purchase_order()`

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "po-2045-uuid-01",
    "po_number": "PO-2045",
    "item_name": "Lidocaine Packet 5cl",
    "vendor_name": "MediPlus Pharma",
    "category": "Procedure Medication",
    "quantity_units": 1240,
    "reserved_units": 220,
    "delivery_eta": "Tomorrow",
    "invoice_status": "Awaiting Submission",
    "approval_status": "PENDING"
  }
}
```

---

### 4.4 Approve / Reject PO Status (`PATCH /admin/procurement/orders/{po_id}/status`)
* **Method**: `PATCH`
* **URL Path**: `/admin/procurement/orders/{po_id}/status`
* **Handler**: `procurement_svc.update_po_status()`

#### Request Body
```json
{
  "status": "APPROVED",
  "notes": "Approved by Rahul Sharma (Admin) for immediate dispatch."
}
```

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "message": "Purchase order status updated successfully",
  "data": {
    "id": "po-2045-uuid-01",
    "po_number": "PO-2045",
    "approval_status": "APPROVED",
    "updated_at": "2026-04-20T11:50:00.000Z"
  }
}
```

---

## 5. Screen 3: Vendor Management Tab

### 5.1 Vendor Summary KPI Cards (`GET /admin/procurement/vendors/metrics`)
* **Method**: `GET`
* **URL Path**: `/admin/procurement/vendors/metrics`
* **Handler**: `procurement_svc.get_vendor_metrics()`

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "total_vendors": { "count": 148, "formatted": "148", "percentage_change": 4.0 },
    "active_vendors": { "count": 124, "formatted": "124" },
    "pending_approvals": { "count": 12, "formatted": "12", "status_severity": "WARNING" },
    "expiring_contracts": { "count": 9, "formatted": "9", "status_severity": "ALERT" },
    "delayed_deliveries": { "count": 17, "formatted": "17" },
    "avg_vendor_rating": { "score": 4.5, "formatted": "4.5" }
  }
}
```

---

### 5.2 Vendor Listing Table (`GET /admin/procurement/vendors`)
* **Method**: `GET`
* **URL Path**: `/admin/procurement/vendors`
* **Handler**: `procurement_svc.list_vendors()`

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "id": "v-9912-medline",
        "vendor_code": "VND-1020",
        "vendor_name": "MedLine Pharma Pvt Ltd",
        "gstin": "27AAACM1234F1Z5",
        "category": "Pharma Supplier",
        "contact_person": "Rahul Desai",
        "contact_phone": "+91 98765 43210",
        "performance": {
          "on_time_delivery_rate": 98.0,
          "formatted_on_time": "98% On-Time",
          "rating": 5.0,
          "last_delivery": "Last delivery 2 days ago"
        },
        "status": "Contract Expiring"
      }
    ],
    "meta": { "total": 124, "page": 1, "page_size": 10, "total_pages": 13 }
  }
}
```

---

### 5.3 Vendor 360-Degree Profile Drawer (`GET /admin/procurement/vendors/{vendor_id}`)
* **Method**: `GET`
* **URL Path**: `/admin/procurement/vendors/{vendor_id}`
* **Handler**: `procurement_svc.get_vendor()`

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "v-9912-medline",
    "vendor_code": "VND-1020",
    "vendor_name": "MedLine Pharma Pvt Ltd",
    "status": "ACTIVE",
    "badges": ["Active", "Preferred", "GST Verified"],
    "ai_recommendation": "Preferred vendor for Emergency OT Supplies. Lowest cost & 98% on-time delivery rate.",
    "basic_information": {
      "vendor_type": "Pharma Supplier",
      "gst_number": "27AAACM1234F1Z5",
      "pan_number": "AAACM1234D",
      "drug_license_no": "DL-MH-20Z-123456",
      "registered_address": "Unit 48, HealthTech Park, Andheri East, Mumbai, Maharashtra 400093"
    },
    "procurement_commercials": {
      "standard_lead_time": "2-3 Days",
      "credit_period": "45 Days",
      "payment_terms": "Net 45",
      "delivery_location": "Mumbai Central Store"
    },
    "performance_analytics": {
      "po_fulfillment_rate": 98.0,
      "discrepancy_rate": 1.2,
      "avg_lead_time_days": 1.8,
      "delayed_orders_count": "2 / 112",
      "compliance_rate": 100.0
    },
    "financial_overview": {
      "total_procured_ytd": 12400000.00,
      "formatted_total_procured": "₹ 1.24 Cr",
      "outstanding_payable": 450000.00,
      "formatted_outstanding_payable": "₹ 4,50,000"
    },
    "compliance_documents": [
      { "id": "doc-01", "name": "GST_Certificate_2023.pdf", "download_url": "/api/v1/documents/doc-01" },
      { "id": "doc-02", "name": "Drug_License_Renewed.pdf", "download_url": "/api/v1/documents/doc-02" },
      { "id": "doc-03", "name": "Vendor_Agreement_v2.pdf", "download_url": "/api/v1/documents/doc-03" }
    ]
  }
}
```

---

### 5.4 Create / Onboard New Vendor (`POST /admin/procurement/vendors`)
* **Method**: `POST`
* **URL Path**: `/admin/procurement/vendors`
* **Handler**: `procurement_svc.create_vendor()`

#### Request Body
```json
{
  "vendor_name": "MedLine Pharma Pvt Ltd",
  "category": "Pharma Supplier",
  "gstin": "27AAACM1234F1Z5",
  "pan_number": "AAACM1234D",
  "drug_license_no": "DL-MH-20Z-123456",
  "contact_person": "Rahul Desai",
  "contact_phone": "+91 98765 43210",
  "official_email": "rahul.desai@medlinepharma.com",
  "registered_address": "Unit 48, HealthTech Park, Andheri East, Mumbai, Maharashtra 400093",
  "credit_period_days": 45,
  "lead_time_days": 3
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

## 6. Screen 4: Asset Management Tab

### 6.1 Asset Dashboard Overview & KPI Metrics (`GET /admin/inventory/assets/dashboard`)
* **Method**: `GET`
* **URL Path**: `/admin/inventory/assets/dashboard`
* **Handler**: `dashboard_svc.get_asset_dashboard()`

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "total_assets": 2847,
    "operational_assets": 2341,
    "under_maintenance": 312,
    "out_of_service": 94,
    "maintenance_due": 187,
    "maintenance_alerts": [
      { "equipment": "MRI Scanner Bay 2", "message": "Overdue maintenance by 3 days", "severity": "HIGH" },
      { "equipment": "Sterilization Unit A4", "message": "Calibration required", "severity": "MEDIUM" },
      { "equipment": "Emergency Generator", "message": "Annual inspection due in 5 days", "severity": "LOW" }
    ],
    "upcoming_maintenance": [
      { "asset_name": "MRI Scanner #3", "department": "Radiology", "next_maintenance": "Oct 12, 2026", "status": "Pending" },
      { "asset_name": "ICU Ventilator Unit #12", "department": "ICU", "next_maintenance": "Oct 14, 2026", "status": "Scheduled" }
    ],
    "critical_equipment_status": [
      { "name": "ICU Ventilators", "online": 48, "total": 50, "status": "ONLINE" },
      { "name": "OR Monitors", "online": 24, "total": 24, "status": "ONLINE" }
    ],
    "assets_by_department": [
      { "department": "Radiology", "count": 420 },
      { "department": "Surgery", "count": 380 }
    ],
    "recent_activity": [
      { "activity": "MRI Scanner A3 — Maintenance Completed", "time_ago": "2 hours ago" }
    ]
  }
}
```

---

### 6.2 Asset Listing Table & Filter Drawer (`GET /admin/inventory/assets`)
* **Method**: `GET`
* **URL Path**: `/admin/inventory/assets`
* **Handler**: `inventory_svc.list_assets()`

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "id": "ast-2024-001-uuid",
        "asset_id": "AST-2024-001",
        "asset_name": "MRI Scanner - GE Signa Premier 3T",
        "category": "Medical Equipment",
        "location": "Radiology - Ground Floor",
        "purchase_date": "2021-01-12",
        "formatted_purchase_date": "Jan 12, 2021",
        "condition": "Good",
        "status": "Maintenance"
      }
    ],
    "meta": { "total": 124, "page": 1, "page_size": 10, "total_pages": 13 }
  }
}
```

---

### 6.3 Single Asset Details Drawer (`GET /admin/inventory/assets/{asset_id}`)
* **Method**: `GET`
* **URL Path**: `/admin/inventory/assets/{asset_id}`
* **Handler**: `inventory_svc.get_asset()`

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "ast-2024-0847-uuid",
    "asset_code": "AST-2024-0847",
    "asset_name": "MRI Scanner T3",
    "status": "Active",
    "summary": {
      "category": "Medical Equipment",
      "sub_category": "Imaging",
      "manufacturer": "Siemens Healthineers",
      "model": "MAGNETOM Vida",
      "serial_no": "SN-9982014-1117"
    },
    "location": {
      "building": "Building A",
      "floor": "Floor 2",
      "room": "Room 215",
      "department": "Radiology Department"
    },
    "purchase_info": {
      "purchase_date": "2021-01-12",
      "purchase_cost": 1450000.00,
      "formatted_cost": "₹ 1,450,000",
      "vendor_name": "Siemens Healthcare",
      "po_number": "PO-2024-0894"
    },
    "warranty": {
      "valid_till": "2029-03-31",
      "formatted_valid_till": "Mar 2029",
      "coverage": "Siemens Care (Comprehensive Coverage)"
    },
    "maintenance_timeline": {
      "last_serviced": "2024-01-15",
      "preventive_maintenance_due": "2024-04-20",
      "inspection_status": "Passed"
    }
  }
}
```

---

### 6.4 Schedule Asset Maintenance Form (`POST /admin/inventory/assets/{asset_id}/maintenance`)
* **Method**: `POST`
* **URL Path**: `/admin/inventory/assets/{asset_id}/maintenance`
* **Handler**: `inventory_svc.schedule_asset_maintenance()`

#### Request Body
```json
{
  "asset_code": "AST-2024-0847",
  "maintenance_type": "Preventive",
  "scheduled_date": "2025-02-28",
  "priority_status": "High Priority",
  "assigned_technician": "Mr. Marcus Brody (Lead Biotech Eng.)",
  "vendor_service_provider": "Medtronics Allied Engineering",
  "estimated_service_cost": 45000.00,
  "service_scope_notes": "Bi-annual pressure seal inspection, gas calibration, replacement of flow sensor filters. Complete oxygen pressure drop tests."
}
```

#### Response Body (`201 Created`)
```json
{
  "success": true,
  "code": 201,
  "message": "Asset maintenance task scheduled successfully",
  "data": {
    "maintenance_id": "maint-2025-0089",
    "asset_code": "AST-2024-0847",
    "asset_name": "Ventilator Unit A3",
    "scheduled_date": "2025-02-28",
    "status": "SCHEDULED",
    "created_at": "2026-04-20T11:55:00.000Z"
  }
}
```

---

### 6.5 Add New Asset Form (`POST /admin/inventory/assets`)
Registers a new medical device, equipment, or facility asset (from the **Add New Asset** modal screen).

* **Method**: `POST`
* **URL Path**: `/admin/inventory/assets`
* **Handler**: `inventory_svc.create_non_medical_inventory()`

#### Request Body Schema
| Field | Type | Mandatory | Constraints | Description |
| :--- | :--- | :--- | :--- | :--- |
| `asset_name` | `string` | Mandatory | Max 200 chars | Commercial name (e.g. `MRI Scanner - GE Signa Premier 3T`). |
| `category` | `string` | Mandatory | Max 60 chars | `Medical Equipment`, `IT Equipment`, `Office Furniture`, `Lab Equipment`. |
| `sub_category` | `string` | Optional | Max 60 chars | e.g. `Imaging`, `Ventilation`, `Monitoring`. |
| `manufacturer` | `string` | Optional | Max 120 chars | Manufacturer name (e.g. `Siemens Healthineers`, `GE`). |
| `model` | `string` | Optional | Max 120 chars | Model string (e.g. `MAGNETOM Vida`). |
| `serial_number` | `string` | Mandatory | Max 60 chars | Serial number (e.g. `SN-9982014-1117`). |
| `assigned_department` | `string (UUID)` | Mandatory | UUID | Department UUID (e.g. `Radiology Department`). |
| `building` | `string` | Optional | Max 60 chars | e.g. `Building A`. |
| `floor_wing` | `string` | Optional | Max 60 chars | e.g. `Floor 2, Ground Floor`. |
| `room_number` | `string` | Optional | Max 60 chars | e.g. `Room 215`. |
| `asset_location_code` | `string` | Optional | Max 60 chars | Location code (e.g. `LOC-RAD-215`). |
| `purchase_date` | `string` | Optional | YYYY-MM-DD | Date of purchase (e.g. `2021-01-12`). |
| `purchase_cost` | `number` | Optional | Min: 0 | Cost in INR (e.g. `1450000.00`). |
| `preferred_vendor_id` | `string (UUID)` | Optional | UUID | Vendor UUID (e.g. `MedTech Solutions`). |
| `service_contract_no` | `string` | Optional | Max 60 chars | PO / Contract Number (e.g. `PO-2024-0894`). |
| `warranty_period` | `string` | Optional | YYYY-MM-DD | Warranty expiry date (e.g. `2029-03-31`). |
| `maintenance_required` | `boolean` | Optional | Default: `true` | Requires scheduled maintenance sweeps. |

```json
{
  "asset_name": "MRI Scanner - GE Signa Premier 3T",
  "category": "Medical Equipment",
  "sub_category": "Imaging",
  "manufacturer": "Siemens Healthineers",
  "model": "MAGNETOM Vida",
  "serial_number": "SN-9982014-1117",
  "assigned_department": "0fbfdbef-9873-4824-b1fd-6fb827d3ba57",
  "building": "Building A",
  "floor_wing": "Floor 2",
  "room_number": "Room 215",
  "asset_location_code": "LOC-RAD-215",
  "purchase_date": "2021-01-12",
  "purchase_cost": 1450000.00,
  "service_contract_no": "PO-2024-0894",
  "warranty_period": "2029-03-31",
  "maintenance_required": true
}
```

#### Response Body (`201 Created`)
```json
{
  "success": true,
  "code": 201,
  "message": "Asset created successfully",
  "data": {
    "id": "ast-2024-0847-uuid",
    "asset_code": "AST-2024-0847",
    "asset_name": "MRI Scanner - GE Signa Premier 3T",
    "category": "Medical Equipment",
    "status": "Active",
    "created_at": "2026-04-20T11:58:00.000Z"
  }
}
```

---

## 7. Standard API Error Codes Matrix

| HTTP Status Code | Error Response Schema (`code`, `message`, `details`) | Cause / Trigger | Frontend Remediation |
| :--- | :--- | :--- | :--- |
| `400 Bad Request` | `{"success": false, "code": 400, "message": "Invalid date range or format"}` | Date picker range end < start or malformed query string. | Show form field error. |
| `401 Unauthorized` | `{"success": false, "code": 401, "message": "Token expired or missing"}` | Missing or invalid Cognito JWT Bearer token in header. | Redirect user to Login. |
| `403 Forbidden` | `{"success": false, "code": 403, "message": "Permission denied: Inventory access required"}` | Role lacks `ADM-001` or `PHM-004` permission. | Show permission alert message. |
| `404 Not Found` | `{"success": false, "code": 404, "message": "Item, Vendor, or Asset not found"}` | Invalid `item_id`, `vendor_id`, `po_id`, or `asset_id`. | Display 404 state empty drawer. |
| `422 Unprocessable Entity` | `{"success": false, "code": 422, "message": "Insufficient stock available"}` | Transfer quantity exceeds `available_units`. | Highlight quantity field warning. |
