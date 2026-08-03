# Admin Inventory & Procurement API Specification (Frontend Integration Guide)

This document is the **master integration reference guide for Frontend (FE) Developers** implementing the **Inventory & Supply Chain Dashboard** in the HMS application. 

It covers all 4 primary sub-tabs (**Inventory Overview**, **Procurement**, **Vendor Management**, and **Asset Management**), as well as side drawers, modals, filter panels, document previews, and action workflows.

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

## 2. Screen 1: Inventory Overview Tab

### 2.1 Overview KPI Summary Cards (`GET /admin/inventory/warehouse/overview`)
Populates the 4 top summary cards on the Inventory Overview screen.

* **Method**: `GET`
* **URL Path**: `/admin/inventory/warehouse/overview`
* **Handler**: `inventory_svc.get_warehouse_overview()`

#### Query Parameters
| Parameter | Type | Required | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| `as_of_date` | `string (YYYY-MM-DD)` | Optional | Today | Snapshot cutoff date for metrics calculation. |

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
      "comparison_period": "vs last month",
      "trend": "UP"
    },
    "total_stock_items": {
      "count": 98340,
      "formatted": "98,340",
      "percentage_change": 3.2,
      "comparison_period": "vs last month",
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

### 2.2 Warehouse Inventory Table & Filter Drawer (`GET /admin/inventory`)
Fetches paginated stock records matching search terms, department filters, warehouse locations, movement statuses, and batch expiry date ranges.

* **Method**: `GET`
* **URL Path**: `/admin/inventory`
* **Handler**: `inventory_svc.list_inventory()`

#### Query Parameters (All Optional)
| Parameter | Type | Allowed Values | Description |
| :--- | :--- | :--- | :--- |
| `search` | `string` | Free text | Matches Item Name, SKU Code (e.g. `012456`), or Order ID (`OD-102456`). |
| `item_type` | `string` | `MEDICAL`, `NON_MEDICAL` | Filter by medical vs non-medical item classification. |
| `category` | `string` | `OT Consumables`, `Pharmacy`, `Laboratory`, `House Keeping`, `Radiology` | Item category filter. |
| `department_id` | `string (UUID)` | UUID | Filter by assigned department location. |
| `warehouse_location` | `string` | e.g. `Central Storage` | Storage location string filter. |
| `movement_status` | `string` | `STABLE`, `CRITICAL`, `FAST_MOVING`, `SLOW_MOVING` | Badge status filter. Can pass multiple comma-separated. |
| `status_level` | `string` | `HEALTHY`, `LOW_STOCK`, `CRITICAL` | Stock health status. |
| `expiry_date_from` | `string (YYYY-MM-DD)` | Date | Batch expiry start date range. |
| `expiry_date_to` | `string (YYYY-MM-DD)` | Date | Batch expiry end date range. |
| `page` | `integer` | Minimum: `1` (Default: `1`) | Pagination page number. |
| `page_size` | `integer` | 1–100 (Default: `10`) | Items limit per page. |

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
        "department": {
          "id": "dept-01",
          "name": "Pharmacy"
        },
        "warehouse_location": {
          "id": "loc-c1",
          "name": "Central Storage"
        },
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
      },
      {
        "id": "item-99282-b91",
        "order_id": "OD-102456",
        "sku_code": "SKU: 012457",
        "item_name": "Surgical Gloves",
        "category": "OT Consumables",
        "department": {
          "id": "dept-02",
          "name": "Laboratory"
        },
        "warehouse_location": {
          "id": "loc-c1",
          "name": "Central Storage"
        },
        "stock": {
          "available_units": 1240,
          "reserved_units": 220,
          "total_units": 1460,
          "unit_of_measure": "Units"
        },
        "batch": {
          "batch_number": "BG-2026-12",
          "expiry_date": "2026-06-30",
          "formatted_expiry": "Jun 2026"
        },
        "movement_status": "CRITICAL"
      }
    ],
    "meta": {
      "total": 124,
      "page": 1,
      "page_size": 10,
      "total_pages": 13
    }
  }
}
```

---

### 2.3 Item Details Drawer & Activity Log (`GET /admin/inventory/{item_id}`)
Fetches single item attributes and recent audit/reservation activities for the right side drawer.

* **Method**: `GET`
* **URL Path**: `/admin/inventory/{item_id}`
* **Handler**: `inventory_svc.get_inventory_item()`

#### Path Parameters
* `item_id` (`string`, Mandatory): Item UUID (e.g. `item-99281-a82`).

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
      },
      {
        "id": "act-103",
        "title": "Stock Batch Verified and Tagged at Central Storage",
        "performed_by": "Mark Spencer",
        "formatted_timestamp": "Oct 03, 2023 • 08:30 AM",
        "activity_type": "VERIFICATION"
      }
    ]
  }
}
```

---

## 3. Screen 2: Procurement Tab

### 3.1 Procurement Dashboard KPI Insights (`GET /admin/procurement/insights`)
Fetches top 4 KPI cards on the **Procurement** tab screen.

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
      "total_purchase_orders": {
        "count": 1284,
        "formatted": "1,284",
        "label": "Orders placed",
        "percentage_change": 12.4,
        "trend": "UP"
      },
      "procurement_value": {
        "amount": 4836500.00,
        "formatted": "₹48,36,500",
        "currency": "INR",
        "label": "Total procurement spend",
        "percentage_change": 18.5,
        "trend": "UP"
      },
      "pending_purchase_orders": {
        "count": 87,
        "formatted": "87",
        "label": "Requires attention",
        "overdue_count": 14,
        "on_track_count": 73,
        "status_severity": "WARNING"
      },
      "active_suppliers": {
        "count": 134,
        "formatted": "134",
        "label": "Approved vendors",
        "percentage_change": 4.0,
        "trend": "UP"
      }
    }
  }
}
```

---

### 3.2 Purchase Orders List & Filters (`GET /admin/procurement/orders`)
Fetches paginated purchase order rows matching search inputs, status badges, vendor selection, and date pickers.

* **Method**: `GET`
* **URL Path**: `/admin/procurement/orders`
* **Handler**: `procurement_svc.list_purchase_orders()`

#### Query Parameters
| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `search` | `string` | Optional | Search string for PO ID e.g. `PO-2045`, Item e.g. `Lidocaine`, or Vendor. |
| `status` | `string` | Optional | Filter by approval status: `Pending`, `Processing`, `Approved`, `Rejected`. |
| `vendor_id` | `string (UUID)` | Optional | Filter by vendor UUID. |
| `invoice_status` | `string` | Optional | `Awaiting Submission`, `Pending GRN`, `Matched`. |
| `date_from` | `string (YYYY-MM-DD)` | Optional | Start date filter. |
| `date_to` | `string (YYYY-MM-DD)` | Optional | End date filter. |
| `page` | `integer` | Optional (Default: `1`) | Page offset. |
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
      },
      {
        "id": "po-2046-uuid-02",
        "po_number": "PO-2046",
        "item_name": "Lidocaine Packet 5cl",
        "vendor_name": "MediPlus Pharma",
        "vendor_category": "Primary Medical Dist.",
        "category": "Procedure Medication",
        "quantity_units": 1240,
        "reserved_units": 220,
        "delivery_eta": "2 days",
        "invoice_status": "Pending GRN",
        "approval_status": "Processing",
        "total_amount": 45000.00,
        "created_at": "2026-04-18T14:30:00.000Z"
      },
      {
        "id": "po-2047-uuid-03",
        "po_number": "PO-2047",
        "item_name": "Lidocaine Packet 5cl",
        "vendor_name": "MediPlus Pharma",
        "vendor_category": "Primary Medical Dist.",
        "category": "Procedure Medication",
        "quantity_units": 1240,
        "reserved_units": 220,
        "delivery_eta": "Delivered",
        "invoice_status": "Matched",
        "approval_status": "Approved",
        "total_amount": 45000.00,
        "created_at": "2026-04-15T09:15:00.000Z"
      }
    ],
    "meta": {
      "total": 124,
      "page": 1,
      "page_size": 10,
      "total_pages": 13
    }
  }
}
```

---

### 3.3 Purchase Order Details Modal (`GET /admin/procurement/orders/{po_id}`)
Fetches single PO details for rendering in the bottom-right details modal.

* **Method**: `GET`
* **URL Path**: `/admin/procurement/orders/{po_id}`
* **Handler**: `procurement_svc.get_purchase_order()`

#### Path Parameters
* `po_id` (`string`, Mandatory): Purchase Order UUID.

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

### 3.4 Approve / Reject PO Status (`PATCH /admin/procurement/orders/{po_id}/status`)
Triggers the "Approve Order" button action in the Purchase Order modal.

* **Method**: `PATCH`
* **URL Path**: `/admin/procurement/orders/{po_id}/status`
* **Handler**: `procurement_svc.update_po_status()`

#### Request Body
| Field | Type | Mandatory | Allowed Values | Description |
| :--- | :--- | :--- | :--- | :--- |
| `status` | `string` | Yes | `APPROVED`, `REJECTED`, `PROCESSING` | Target approval status. |
| `notes` | `string` | Optional | Max 500 chars | Approval or rejection justification note. |

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

## 4. Screen 3: Vendor Management Tab

### 4.1 Vendor Summary KPI Cards (`GET /admin/procurement/vendors/metrics`)
Populates the 6 summary cards at the top of the **Vendor Management** screen.

* **Method**: `GET`
* **URL Path**: `/admin/procurement/vendors/metrics`
* **Handler**: `procurement_svc.get_vendor_metrics()`

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "total_vendors": {
      "count": 148,
      "formatted": "148",
      "percentage_change": 4.0
    },
    "active_vendors": {
      "count": 124,
      "formatted": "124"
    },
    "pending_approvals": {
      "count": 12,
      "formatted": "12",
      "status_severity": "WARNING"
    },
    "expiring_contracts": {
      "count": 9,
      "formatted": "9",
      "status_severity": "ALERT"
    },
    "delayed_deliveries": {
      "count": 17,
      "formatted": "17"
    },
    "avg_vendor_rating": {
      "score": 4.5,
      "formatted": "4.5"
    }
  }
}
```

---

### 4.2 Vendor Listing Table (`GET /admin/procurement/vendors`)
Retrieves paginated suppliers for the main table and populates vendor dropdowns.

* **Method**: `GET`
* **URL Path**: `/admin/procurement/vendors`
* **Handler**: `procurement_svc.list_vendors()`

#### Query Parameters
| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `search` | `string` | Optional | Search Vendor Name (`MedLine Pharma`), Code (`VND-1020`), or GSTIN (`27AAACM1234F1Z5`). |
| `status` | `string` | Optional | `Active`, `Pending Approval`, `Contract Expiring`, `Inactive`. |
| `category` | `string` | Optional | `Pharma Supplier`, `Surgical Vendor`, `Lab Consumables`. |
| `performance_score` | `string` | Optional | `HIGH` (>90%), `AVERAGE` (70-90%), `UNDERPERFORMING` (<70%). |
| `contract_date_from` | `string (YYYY-MM-DD)` | Optional | Contract date range start. |
| `contract_date_to` | `string (YYYY-MM-DD)` | Optional | Contract date range end. |
| `page` | `integer` | Optional (Default: `1`) | Page index. |
| `page_size` | `integer` | Optional (Default: `10`) | Limit per page. |

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
      },
      {
        "id": "v-9913-surgicare",
        "vendor_code": "VND-1021",
        "vendor_name": "SurgiCare Supplies",
        "gstin": "27AAACM2345G2Z6",
        "category": "Surgical Vendor",
        "contact_person": "Rahul Desai",
        "contact_phone": "+91 98765 43210",
        "performance": {
          "on_time_delivery_rate": 82.0,
          "formatted_on_time": "82% On-Time",
          "rating": 4.0,
          "last_delivery": "3 delayed deliveries this month"
        },
        "status": "Pending Approval"
      },
      {
        "id": "v-9914-healthcare",
        "vendor_code": "VND-1022",
        "vendor_name": "HealCare Diagnostics",
        "gstin": "07AAACM3456H3Z7",
        "category": "Lab Consumables",
        "contact_person": "Rahul Desai",
        "contact_phone": "+91 98765 43210",
        "performance": {
          "on_time_delivery_rate": 82.0,
          "formatted_on_time": "82% On-Time",
          "rating": 4.0,
          "last_delivery": "3 delayed deliveries this month"
        },
        "status": "Active"
      }
    ],
    "meta": {
      "total": 124,
      "page": 1,
      "page_size": 10,
      "total_pages": 13
    }
  }
}
```

---

### 4.3 Vendor 360-Degree Profile Drawer (`GET /admin/procurement/vendors/{vendor_id}`)
Retrieves full details for rendering the right-side vendor profile drawer.

* **Method**: `GET`
* **URL Path**: `/admin/procurement/vendors/{vendor_id}`
* **Handler**: `procurement_svc.get_vendor()`

#### Path Parameters
* `vendor_id` (`string`, Mandatory): Vendor UUID (e.g. `v-9912-medline`).

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
    "active_contracts": [
      {
        "contract_id": "CT-2023-0045",
        "title": "Critical OT Supplies SLA",
        "category": "Critical OT Medication",
        "valid_till": "2027-12-31",
        "auto_renewal": true,
        "status": "ACTIVE_SLA"
      }
    ],
    "stock_details": {
      "current_stock": 1240,
      "reorder_level": 150,
      "lead_time": "2-3 Days",
      "last_received": "2026-01-12",
      "stock_status": "IN STOCK",
      "stock_value": 245000.00,
      "formatted_stock_value": "₹ 2,45,000"
    },
    "quantity_received_breakdown": [
      { "item_name": "Paracetamol 500mg", "unit": "Tabs", "ordered_qty": 200, "received_qty": 200, "pending_qty": 0 },
      { "item_name": "Amoxicillin 250mg", "unit": "Caps", "ordered_qty": 150, "received_qty": 150, "pending_qty": 0 },
      { "item_name": "IV Saline 500ml", "unit": "Btl", "ordered_qty": 50, "received_qty": 48, "pending_qty": 2 },
      { "item_name": "Gloves L", "unit": "Pairs", "ordered_qty": 300, "received_qty": 300, "pending_qty": 0 },
      { "item_name": "Syringes 5ml", "unit": "Pcs", "ordered_qty": 120, "received_qty": 110, "pending_qty": 10 }
    ],
    "compliance_documents": [
      { "id": "doc-01", "name": "GST_Certificate_2023.pdf", "download_url": "/api/v1/documents/doc-01" },
      { "id": "doc-02", "name": "Drug_License_Renewed.pdf", "download_url": "/api/v1/documents/doc-02" },
      { "id": "doc-03", "name": "Vendor_Agreement_v2.pdf", "download_url": "/api/v1/documents/doc-03" }
    ]
  }
}
```

---

### 4.4 Create / Onboard New Vendor (`POST /admin/procurement/vendors`)
Registers a new supplier when the user clicks `+ Create Vendor`.

* **Method**: `POST`
* **URL Path**: `/admin/procurement/vendors`
* **Handler**: `procurement_svc.create_vendor()`

#### Request Body
| Field | Type | Mandatory | Constraints | Description |
| :--- | :--- | :--- | :--- | :--- |
| `vendor_name` | `string` | Mandatory | Max 200 chars | Legal company name. |
| `category` | `string` | Mandatory | e.g. `Pharma Supplier` | Category classification. |
| `gstin` | `string` | Mandatory | 15 chars (e.g. `27AAACM1234F1Z5`) | GST identification number. |
| `pan_number` | `string` | Mandatory | 10 chars (e.g. `AAACM1234D`) | PAN number. |
| `drug_license_no` | `string` | Optional | Max 60 chars | Drug License Registration number. |
| `contact_person` | `string` | Mandatory | Max 100 chars | Primary contact person name. |
| `contact_phone` | `string` | Mandatory | Max 20 chars | Primary mobile/phone number. |
| `official_email` | `string` | Mandatory | Max 200 chars | Email address. |
| `registered_address` | `string` | Mandatory | Max 500 chars | Full registered address. |
| `credit_period_days` | `integer` | Mandatory | Min: 0 (e.g. `45`) | Payment credit days. |
| `lead_time_days` | `integer` | Mandatory | Min: 0 (e.g. `3`) | Standard delivery lead time days. |

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
    "id": "v-9912-medline",
    "vendor_code": "VND-1020",
    "vendor_name": "MedLine Pharma Pvt Ltd",
    "status": "PENDING_APPROVAL",
    "created_at": "2026-04-20T11:52:00.000Z"
  }
}
```

---

## 5. Screen 4: Workflows, GRN & Invoice Reconciliation

### 5.1 POST Receive Goods Received Note (GRN) (`POST /admin/procurement/grn`)
Records arrival of delivered items and updates stock balances.

* **Method**: `POST`
* **URL Path**: `/admin/procurement/grn`
* **Handler**: `procurement_svc.receive_grn()`

#### Request Body
```json
{
  "po_id": "po-2045-uuid-01",
  "vendor_id": "v-9912-medline",
  "notes": "Verified 1,240 units received in good condition.",
  "items": [
    {
      "item_name": "Lidocaine Packet 5cl",
      "batch_number": "BG-2026-88",
      "quantity_received": 1240,
      "accepted_quantity": 1240,
      "rejected_quantity": 0,
      "expiry_date": "2028-06-30"
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
    "grn_id": "grn-2026-0091",
    "po_id": "po-2045-uuid-01",
    "status": "COMPLETED",
    "created_at": "2026-04-20T12:00:00.000Z"
  }
}
```

---

### 5.2 POST Reconcile Invoice (`POST /admin/procurement/invoices/{invoice_id}/reconcile`)
Matches received vendor invoice against PO & GRN records.

* **Method**: `POST`
* **URL Path**: `/admin/procurement/invoices/{invoice_id}/reconcile`
* **Handler**: `procurement_svc.reconcile_invoice()`

#### Request Body
```json
{
  "match_status": "MATCHED",
  "mismatch_reason": null
}
```

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "message": "Invoice reconciled successfully",
  "data": {
    "invoice_id": "inv-2026-9912",
    "status": "RECONCILED"
  }
}
```

---

## 6. Standard API Error Codes Matrix

| HTTP Status Code | Error Response Schema (`code`, `message`, `details`) | Cause / Trigger | Frontend Remediation |
| :--- | :--- | :--- | :--- |
| `400 Bad Request` | `{"success": false, "code": 400, "message": "Invalid date range or format"}` | Date picker range end < start or malformed query string. | Show form field error. |
| `401 Unauthorized` | `{"success": false, "code": 401, "message": "Token expired or missing"}` | Missing or invalid Cognito JWT Bearer token in header. | Redirect user to Login. |
| `403 Forbidden` | `{"success": false, "code": 403, "message": "Permission denied: Inventory access required"}` | Role lacks `ADM-001` or `PHM-004` permission. | Show permission alert message. |
| `404 Not Found` | `{"success": false, "code": 404, "message": "Item or Vendor not found"}` | Invalid `item_id`, `vendor_id`, or `po_id`. | Display 404 state empty drawer. |
| `422 Unprocessable Entity` | `{"success": false, "code": 422, "message": "Insufficient stock available"}` | Transfer quantity exceeds `available_units`. | Highlight quantity field warning. |
