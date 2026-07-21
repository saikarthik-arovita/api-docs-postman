# Laboratory Service — Complete API Documentation

> **Generated from live source code and actual database responses.**  
> **Base URL**: `https://api.arovita.com/diagnostic-orders`  
> **Service**: Laboratory Management Module  
> **Last Updated**: 2026-07-16

---

## Table of Contents

1. [Authentication & Headers](#1-authentication--headers)
2. [Response Envelope](#2-response-envelope)
3. [API Endpoints — GET](#3-api-endpoints--get)
   - [Orders](#31-orders)
   - [Lookup / Dropdowns](#32-lookup--dropdowns)
   - [Dashboard & Analytics](#33-dashboard--analytics)
   - [Inventory](#34-inventory)
   - [Vendors](#35-vendors)
   - [Procurement](#36-procurement)
   - [Billing](#37-billing)
   - [Staff](#38-staff)
   - [Roles (RBAC)](#39-roles-rbac)
   - [Config (Templates & Machines)](#310-config-templates--machines)
   - [Audit & Compliance](#311-audit--compliance)
4. [API Endpoints — POST / PATCH / DELETE](#4-api-endpoints--post--patch--delete)
   - [Create Order](#41-create-diagnostic-order)
   - [Order Actions](#42-order-actions)
   - [Bulk Order Actions](#43-bulk-order-actions)
   - [Create Lab Test (Catalog)](#44-create-lab-test-catalog)
   - [Create Inventory Item](#45-create-inventory-item)
   - [Create Vendor](#46-create-vendor)
   - [Create Purchase Order](#47-create-purchase-order)
   - [Create Role](#48-create-role)
   - [Create Config (Report Template)](#49-create-config-report-template)
5. [Database Enum Reference](#5-database-enum-reference)
6. [Error Responses](#6-error-responses)

---

## 1. Authentication & Headers

All requests require a valid AWS Cognito JWT Bearer token and a tenant header.

| Header | Required | Description |
| :--- | :---: | :--- |
| `Authorization` | ✅ | `Bearer <JWT_TOKEN>` |
| `X-Tenant-Id` | ✅ | Branch/Tenant UUID |
| `Content-Type` | ✅ | `application/json` |

---

## 2. Response Envelope

All API responses follow this standard envelope:

### Success Response
```json
{
  "success": true,
  "code": 200,
  "data": { },
  "message": "optional success message"
}
```

### Error Response
```json
{
  "success": false,
  "code": 400,
  "message": "Error description",
  "errors": []
}
```

---

## 3. API Endpoints — GET

### 3.1 Orders

#### `GET /diagnostic-orders/orders`

List all lab orders (worklist). Supports pagination and filtering.

**Query Parameters:**

| Param | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `page` | integer | `1` | Page number |
| `limit` | integer | `10` | Items per page |
| `status` | string | — | Filter by status (`NEW`, `COLLECTED`, `PROCESSING`, etc.) |
| `priority` | string | — | Filter by priority (`ROUTINE`, `URGENT`, `STAT`) |
| `search` | string | — | Search by patient name, UHID, or test name |

**Response:**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "order_id": "2e737ce4-d61b-4ed6-8795-b38a902e3608",
        "order_item_id": "04440985-c6d9-421c-99be-7c8a91c481bf",
        "patient": {
          "id": "8404356a-9d8d-494b-bd29-ddc76b0fb58b",
          "name": "Aditya",
          "uhid": "PAT-2026-0045",
          "age": null,
          "gender": "MALE",
          "phone": "+91951383663"
        },
        "test": {
          "name": "24-Hour Urine Protein",
          "code": "URN-003"
        },
        "barcode": "BC-10019",
        "collector": null,
        "priority": "ROUTINE",
        "created_at": "2026-07-13T14:03:48.018865+05:30",
        "current_status": "NEW"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 10,
      "total": 72,
      "totalPages": 8,
      "hasNext": true,
      "hasPrevious": false
    }
  }
}
```

---

#### `GET /diagnostic-orders/orders/{id}`

Get a single lab order by UUID.

**Path Parameters:**

| Param | Type | Required | Description |
| :--- | :--- | :---: | :--- |
| `id` | UUID | ✅ | Lab order UUID |

**Response:**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "order_id": "uuid",
    "order_item_id": "uuid",
    "patient": {
      "id": "uuid",
      "name": "string",
      "uhid": "string",
      "age": "number|null",
      "gender": "string",
      "phone": "string"
    },
    "test": {
      "name": "string",
      "code": "string"
    },
    "barcode": "string",
    "collector": "string|null",
    "priority": "ROUTINE|URGENT|STAT",
    "created_at": "ISO 8601",
    "current_status": "string"
  }
}
```

---

### 3.2 Lookup / Dropdowns

#### `GET /diagnostic-orders/lookup`

Unified lookup endpoint. Use the `type` query parameter to fetch different data catalogs.

| `type` value | Description |
| :--- | :--- |
| `tests` | Lab test catalog |
| `packages` | Test packages |
| `machines` | Lab machines |
| `departments` | Staff departments |
| `shifts` | Work shifts |
| `permissions` | Available permissions catalog |
| `printers` | Configured printers |
| `report-templates` | Report templates |
| `dropdowns` *(default)* | Registration form dropdowns |

---

#### `GET /diagnostic-orders/lookup?type=tests`

**Query Parameters:** `search`, `department`, `status`, `category`, `page`, `limit`

**Response:**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "testId": "52949dc3-e8b3-4abc-8d5d-c95b00337f6c",
        "code": "URN-003",
        "testName": "24-Hour Urine Protein",
        "department": "Nephrology",
        "referenceRange": "<150",
        "sampleType": "URINE",
        "containerType": "Vacutainer",
        "tat": "24 hrs",
        "price": 0.0,
        "status": "Active",
        "lastUpdated": "2026-06-23T11:25:21.283249+05:30"
      }
    ],
    "total": 263
  }
}
```

---

#### `GET /diagnostic-orders/lookup?type=packages`

**Response:**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "packageId": "uuid",
        "packageName": "string",
        "tests": ["..."],
        "price": 0.0,
        "status": "ACTIVE"
      }
    ],
    "total": 2
  }
}
```

---

#### `GET /diagnostic-orders/lookup?type=dropdowns`

Returns registration form dropdown values.

**Response:**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "visit_types": ["Walk-in", "OPD Referral", "IPD Referral", "Emergency"],
    "priorities": ["Routine", "Urgent", "Stat"],
    "sample_types": ["Whole Blood", "Serum", "Plasma", "Urine", "Swab"],
    "departments": [
      { "id": "46d19155-09a1-462e-b256-ce2a5741b091", "name": "General Medicine" },
      { "id": "cab7cc4e-3ec7-49d9-a393-bbdb93bf58c2", "name": "Pediatrics" }
    ]
  }
}
```

---

#### `GET /diagnostic-orders/lookup?type=departments`

**Response:**
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "departmentId": "0151fa54-59eb-4066-8755-0025bd9e3961",
      "departmentName": "Dietetics",
      "capacity": 15,
      "occupiedStaff": 0,
      "availableSlots": 15
    }
  ]
}
```

---

#### `GET /diagnostic-orders/lookup?type=permissions`

**Response:**
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "permission_id": "STF-001",
      "permission_name": "staff:create",
      "module": "staff",
      "action": "create",
      "description": "Create new staff accounts"
    }
  ]
}
```

---

#### `GET /diagnostic-orders/lookup?type=printers`

**Response:**
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "printerId": "5ec5b6dd-359f-43e8-9b4b-e91341986100",
      "printerName": "Zebra GK420t",
      "printerIp": "192.168.1.150",
      "connectionType": "NETWORK",
      "status": "Online"
    }
  ]
}
```

---

#### `GET /diagnostic-orders/lookup?type=report-templates`

**Response:**
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "templateId": "d766545b-c6b0-4abc-aa17-49cb0be25e08",
      "templateName": "Biochemistry Report Template",
      "category": "Biochemistry",
      "status": "DRAFT",
      "isDefault": true,
      "lastUpdated": "2026-07-16T10:52:03.987313+05:30"
    }
  ]
}
```

---

### 3.3 Dashboard & Analytics

#### `GET /diagnostic-orders/dashboard`

Returns lab analytics KPIs, volume trends, revenue trends, category distribution.

**Response:**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "kpis": {
      "tests_processed": { "count": 10, "change_pct": 233.3 },
      "samples_collected": { "count": 12, "change_pct": 0.0 },
      "lab_revenue": { "amount": 0.0, "change_pct": 9.3 },
      "avg_tat": { "hours": 0.3, "change_pct": 0.0 },
      "pending_reports": 58,
      "critical_results": 1
    },
    "volume_trend": [
      { "date": "2026-06-01", "count": 19 },
      { "date": "2026-07-01", "count": 53 }
    ],
    "revenue_trend": [
      { "date": "2026-07-01", "amount": 172540.0 }
    ],
    "category_distribution": [
      { "category": "General", "count": 52, "percentage": 72.2 },
      { "category": "Immunology", "count": 1, "percentage": 1.4 },
      { "category": "Haematology", "count": 4, "percentage": 5.6 },
      { "category": "Biochemistry", "count": 2, "percentage": 2.8 },
      { "category": "Microbiology", "count": 3, "percentage": 4.2 }
    ],
    "avg_tat": 0.3,
    "pending_reports": 58,
    "critical_results": 1
  }
}
```

---

### 3.4 Inventory

#### `GET /diagnostic-orders/inventory`

List lab inventory items with enriched stock data.

**Query Parameters:**

| Param | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `page` | integer | `1` | Page number |
| `limit` | integer | `10` | Items per page |
| `category` | string | — | Filter by category enum |
| `search` | string | — | Search by item name/code |
| `status` | string | — | Filter by stock status |
| `stockLevel` | string | — | Filter: `low`, `out`, `normal` |
| `sortBy` | string | `item_name` | Sort field |
| `sortOrder` | string | `ASC` | `ASC` or `DESC` |

**Response:**
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "id": "3221d3cd-367f-4c95-9e3d-e921e9bc7029",
      "item_code": "TEST-ITEM-FFC1EF",
      "item_name": "Verification Reagent Solution",
      "category": "REAGENT",
      "sub_category": "Testing Chemistry",
      "description": "Verification tests fluid",
      "current_stock": 25.0,
      "reorder_level": 5.0,
      "minimum_stock": 2.0,
      "maximum_stock": 50.0,
      "uom": "BOTTLE",
      "batch_number": "B123-VERIFY",
      "expiry_date": "2027-12-31",
      "supplier_name": null,
      "computed_status": "IN_STOCK"
    }
  ]
}
```

---

#### `GET /diagnostic-orders/inventory/{id}`

Get single inventory item detail by UUID.

---

### 3.5 Vendors

#### `GET /diagnostic-orders/vendors`

List registered lab vendors.

**Query Parameters:** `search`, `status`, `category`, `rating`, `risk`, `page`, `limit`, `sort_by`, `sort_order`

**Response:**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "id": "77f8a7ee-abaf-45dc-8441-9d2d4cd43f2c",
        "tenant_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
        "vendor_name": "MedLife Diagnostics Supplies Pvt. Ltd.",
        "vendor_code": "MDS-1001",
        "vendor_type": "DISTRIBUTOR",
        "vendor_category": null,
        "business_type": null,
        "primary_contact": null,
        "mobile": null,
        "official_email": "sales@medlifediagnostics.com",
        "gst_number": null,
        "pan_number": null,
        "drug_license_number": null,
        "risk_level": "LOW",
        "vendor_status": "PENDING_VERIFICATION",
        "total_orders": 0,
        "quality_rating": null,
        "is_active": true,
        "created_at": "2026-07-16T16:48:19.512708+05:30",
        "updated_at": "2026-07-16T16:48:19.512708+05:30"
      }
    ],
    "limit": 10,
    "offset": 0
  }
}
```

---

#### `GET /diagnostic-orders/vendors/{id}`

Get vendor detail. Supports `?include=contracts,analytics,financial,stock,documents,purchaseOrders,performance` to fetch related data.

---

### 3.6 Procurement

#### `GET /diagnostic-orders/procurement`

List purchase orders. Use `?type=grn` for GRNs or `?type=invoice` for vendor invoices.

**Query Parameters:**

| Param | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `type` | string | `po` | `po` / `grn` / `invoice` |
| `search` | string | — | Search text |
| `status` | string | — | Filter by status |
| `page` | integer | `1` | Page |
| `limit` | integer | `10` | Per page |

**Response (type=po):**
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "id": "4e4b4b51-408e-44b2-ac6e-1322be3d82f1",
      "po_number": "PO-LAB-46D603",
      "vendor_id": "f197a875-efe9-432e-ac71-6356abad2efd",
      "status": "ORDERED",
      "priority": "HIGH",
      "lab_section": "Microbiology",
      "requested_by": "8b7f50c4-fa3a-4aef-ab27-f42b7fb0e54f",
      "approved_by": null,
      "estimated_value": 1000.00,
      "gst_amount": 180.00,
      "grand_total": 1180.00,
      "created_at": "2026-07-14T12:46:50.788703+05:30"
    }
  ]
}
```

**Response (type=grn):**
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "id": "a52e974d-8e8d-48a6-9d60-d49777726b09",
      "grn_number": "GRN-LAB-00001",
      "po_id": "4f1b9a00-14df-4b92-9260-91639141b4de",
      "vendor_id": null,
      "status": "ACCEPTED",
      "received_by": "8b7f50c4-fa3a-4aef-ab27-f42b7fb0e54f",
      "received_at": "2026-06-29T15:26:38.027908+05:30"
    }
  ]
}
```

**Response (type=invoice):**
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "id": "93e36d22-5727-44c5-8b45-ec6c3b5953d2",
      "invoice_number": "INV-12345",
      "vendor_id": null,
      "po_id": "4f1b9a00-14df-4b92-9260-91639141b4de",
      "grn_id": "a52e974d-8e8d-48a6-9d60-d49777726b09",
      "total_amount": 1770.00,
      "match_status": "MATCHED",
      "payment_status": "PAID",
      "created_at": "2026-06-29T15:30:00.989274+05:30"
    }
  ]
}
```

---

### 3.7 Billing

#### `GET /diagnostic-orders/billing/invoices`

List patient billing invoices.

**Query Parameters:** `search`, `status`, `paymentMethod`, `insurance`, `startDate`, `endDate`, `page`, `limit`, `sort_by`, `sort_order`, `format` (set `format=csv` to export)

**Response:**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "invoice_id": "989e0219-446d-4b99-929a-8857e9dbf6a2",
        "invoice_number": "INV-2026-92233",
        "patient_name": "E2E Simulation Patient",
        "patient_uhid": "PAT-2026-0004",
        "patient_age": 35,
        "patient_gender": "MALE",
        "patient_phone": "+919924826164",
        "rx_id": null,
        "rx_number": "Walk-in",
        "payment_method": "CASH",
        "amount": 0.0,
        "status": "PAID",
        "invoice_date": "2026-07-11"
      }
    ],
    "total": 1920
  }
}
```

---

#### `GET /diagnostic-orders/billing/invoices/{id}`

Get a single billing invoice by UUID.

---

### 3.8 Staff

#### `GET /diagnostic-orders/staff`

List laboratory staff.

**Query Parameters:** `search`, `designation`, `status`, `shift`, `department`, `page`, `limit`

**Response:**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "employeeId": "cab7cc4e-3ec7-49d9-a393-bbdb93bf58c2",
        "employeeCode": "RCP-ESS388",
        "fullName": "Kiran Patil",
        "designation": "Technician",
        "department": "Diagnostics",
        "shift": "N/A",
        "employmentType": "Full Time",
        "status": "Off Duty",
        "phone": "+919123900025",
        "email": "kiran.patil.b2@hospital.com",
        "profileImage": "https://images.unsplash.com/...",
        "isAssigned": false
      }
    ],
    "total": 94
  }
}
```

---

#### `GET /diagnostic-orders/staff/{id}`

Get staff profile by UUID.

---

### 3.9 Roles (RBAC)

#### `GET /diagnostic-orders/roles`

List all access control roles with assigned user counts and permissions.

**Response:**
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "roleId": "FIN-003",
      "roleName": "ACCOUNTANT",
      "category": "FINANCIAL",
      "description": "Financial reporting",
      "isSuperAdmin": false,
      "status": "ACTIVE",
      "maxUsers": null,
      "sessionTimeout": 30,
      "mfaRequired": false,
      "assignedUsersCount": 0,
      "permissions": ["billing:view", "billing:export", "audit:view"]
    }
  ]
}
```

---

#### `GET /diagnostic-orders/roles/{id}`

Get role details by role ID string.

---

### 3.10 Config (Templates & Machines)

#### `GET /diagnostic-orders/config`

List report templates (default) or machines.

**Query Parameters:**

| Param | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `type` | string | `templates` | `templates` or `machines` |

**Response (type=templates):**
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "templateId": "d766545b-c6b0-4abc-aa17-49cb0be25e08",
      "templateName": "Biochemistry Report Template",
      "category": "Biochemistry",
      "status": "DRAFT",
      "isDefault": true,
      "lastUpdated": "2026-07-16T10:52:03.987313+05:30"
    }
  ]
}
```

**Response (type=machines):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [],
    "total": 0
  }
}
```

---

#### `GET /diagnostic-orders/config/{id}`

Get a single config record (template or machine). Use `?type=machines` for machine details.

---

### 3.11 Audit & Compliance

#### `GET /diagnostic-orders/audit`

List audit logs. Use `type` query param to switch between log types.

**Query Parameters:**

| Param | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `type` | string | `logs` | `logs` / `waste` / `compliance` / `events` |
| `search` | string | — | Search text |
| `module` | string | — | Filter by module |
| `status` | string | — | Filter by status |
| `page` | integer | `1` | Page |
| `limit` | integer | `10` | Per page |

**Response (type=logs):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "auditId": "b077d578-3afc-449a-a4d1-e995724efb27",
        "timestamp": "2026-07-16T16:46:36.537876+05:30",
        "user": "amit.verma@arovita.com",
        "action": "Test Created",
        "resource": "SYSTEM",
        "module": "CONFIG",
        "ipAddress": "127.0.0.1",
        "status": "SUCCESS",
        "beforeValue": null,
        "afterValue": {
          "code": "CBC001",
          "testId": "2d78223d-6d57-4bce-bc87-063738a4f140",
          "testName": "Complete Blood Count (CBC)"
        }
      }
    ],
    "total": 3445
  }
}
```

**Response (type=compliance):**
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "id": "f2543139-2c9e-4e32-8b16-05e289294787",
      "title": "Unauthorized Patient Access",
      "description": "Patient record accessed outside hours",
      "severity": "HIGH",
      "status": "OPEN",
      "reportedDate": "2026-07-14",
      "resolvedDate": null
    }
  ]
}
```

**Response (type=events):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "id": "ea0bf3f7-1de7-4b66-93b6-55695c4310a0",
        "eventType": "LOGIN_FAILURE",
        "severity": "CRITICAL",
        "description": "Failed login attempts exceeded limit",
        "source": "Auth Gateway",
        "timestamp": "2026-07-14T15:18:53.980927+05:30"
      }
    ],
    "total": 8
  }
}
```

---

#### `GET /diagnostic-orders/audit/{id}`

Get audit record detail by UUID. Use `?type=logs|waste|events` to specify the record type.

---

## 4. API Endpoints — POST / PATCH / DELETE

### 4.1 Create Diagnostic Order

`POST /diagnostic-orders/orders`

**Required Fields:**

| Field | Type | Required | Description |
| :--- | :--- | :---: | :--- |
| `patient_id` | UUID | ✅ | Patient UUID |
| `doctor_id` | UUID | ✅ | Referring doctor UUID |
| `order_type` | enum | ✅ | `LAB`, `RADIOLOGY`, `CARDIOLOGY`, `PATHOLOGY` |
| `context_type` | string | ✅ | e.g. `OPD`, `IPD`, `EMERGENCY` |
| `context_id` | UUID | ✅ | Visit/encounter UUID |
| `source_module` | string | ✅ | e.g. `OPD`, `WARD` |
| `encounter_type` | string | ✅ | e.g. `WALK_IN`, `REFERRAL` |
| `lab_items` | array | ✅* | Required when order_type is `LAB` or `PATHOLOGY` |
| `department_id` | UUID | — | Department UUID |
| `priority` | enum | — | `ROUTINE` (default), `URGENT`, `STAT` |
| `clinical_notes` | string | — | Clinical notes |

**`lab_items[]` object:**

| Field | Type | Required | Description |
| :--- | :--- | :---: | :--- |
| `test_id` | UUID | ✅* | Test UUID from catalog. Either `test_id` or `test_name` required. |
| `test_name` | string | ✅* | Test name. Either `test_id` or `test_name` required. |
| `notes` | string | — | Per-test notes |
| `fasting_required` | boolean | — | Default `false` |
| `sample_type` | string | — | Override sample type |

**Request Body Example:**
```json
{
  "patient_id": "8404356a-9d8d-494b-bd29-ddc76b0fb58b",
  "doctor_id": "cab7cc4e-3ec7-49d9-a393-bbdb93bf58c2",
  "order_type": "LAB",
  "priority": "ROUTINE",
  "context_type": "OPD",
  "context_id": "46d19155-09a1-462e-b256-ce2a5741b091",
  "source_module": "OPD",
  "encounter_type": "WALK_IN",
  "lab_items": [
    {
      "test_id": "52949dc3-e8b3-4abc-8d5d-c95b00337f6c",
      "notes": "Fasting sample",
      "fasting_required": true
    }
  ]
}
```

**Success Response:** `201 Created`

---

### 4.2 Order Actions

`POST /diagnostic-orders/orders/{id}/actions`

Execute workflow actions on an order item.

**Request Body:**
```json
{ "action": "<action_name>", ...additional_fields }
```

**Available Actions:**

| Action | Required Body Fields | Description |
| :--- | :--- | :--- |
| `accept` | — | Lab technician accepts the order |
| `reject` | `reason` (string) | Reject with reason |
| `sample-received` | `barcode` (string) | Mark sample as received |
| `collect-sample` | `barcode` (string) | Collect specimen |
| `process` | — | Start processing |
| `save-draft` | `draft_values` (object) | Save result draft |
| `submit-values` | `result_values` (object) | Submit final results |
| `mark-critical` | `parameter_ids` (array), `reason`, `pathologist_id` | Flag critical |
| `send-to-pathologist` | — | Send for pathologist review |
| `repeat-sample` | `reason` (string) | Request repeat sample |
| `acknowledge-critical` | `notes` (string) | Acknowledge critical log |

**Example — Collect Sample:**
```json
{
  "action": "collect-sample",
  "barcode": "BC-10019"
}
```

**Success Response:** `200 OK`
```json
{
  "success": true,
  "code": 200,
  "message": "Sample collected successfully",
  "data": { "...updated order data..." }
}
```

---

### 4.3 Bulk Order Actions

`POST /diagnostic-orders/orders/bulk/actions`

Execute an action on multiple order items at once.

**Request Body:**
```json
{
  "action": "accept",
  "ids": [
    "04440985-c6d9-421c-99be-7c8a91c481bf",
    "b2c3d4e5-f6a7-8901-2345-6789abcdef01"
  ]
}
```

**Response:**
```json
{
  "success": true,
  "code": 200,
  "message": "Bulk action 'accept' executed.",
  "data": {
    "success_count": 2,
    "failed_count": 0,
    "results": [...]
  }
}
```

---

### 4.4 Create Lab Test (Catalog)

`POST /diagnostic-orders/tests`

Create a new lab test in the catalog. Requires `config.write` permission.

**Required Fields:**

| Field | Type | Required | Description |
| :--- | :--- | :---: | :--- |
| `testName` | string | ✅ | Name of the investigation (min 2 chars) |
| `code` | string | ✅ | Unique test code configuration (min 2 chars) |

**Optional Fields:**

| Field | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `departmentId` | UUID string | `null` | Department UUID reference (must be valid UUID format) |
| `category` | string | `null` | Test category (e.g. Hematology, Biochemistry) |
| `referenceRange` | string | `null` | Standard reference range values |
| `unit` | string | `null` | Measurement units (e.g. mg/dL, g/L) |
| `sampleType` | string | `"BLOOD"` | Sample type required (e.g. BLOOD, URINE, CSF) |
| `tat` | integer | `24` | Expected turnaround time in hours |
| `price` | float | `0.00` | Base rate price |

**Request Example:**
```json
{
  "testName": "Thyroid Stimulating Hormone (TSH)",
  "code": "TSH001",
  "departmentId": "46d19155-09a1-462e-b256-ce2a5741b091",
  "category": "Hormonal",
  "referenceRange": "0.4 - 4.0",
  "unit": "uIU/mL",
  "sampleType": "SERUM",
  "tat": 12,
  "price": 350.00
}
```

**Success Response:** `201 Created`
```json
{
  "success": true,
  "code": 201,
  "data": {
    "testId": "c4d0922e-bf6e-44cb-bc71-331cb301d0fe",
    "testName": "Thyroid Stimulating Hormone (TSH)",
    "code": "TSH001",
    "departmentId": "46d19155-09a1-462e-b256-ce2a5741b091",
    "category": "Hormonal",
    "referenceRange": "0.4 - 4.0",
    "unit": "uIU/mL",
    "sampleType": "SERUM",
    "tat": 12,
    "price": 350.00,
    "status": "ACTIVE"
  }
}
```

---

### 4.5 Create Inventory Item

`POST /diagnostic-orders/inventory`

**Required Fields (only 3!):**

| Field | Type | Required | Description |
| :--- | :--- | :---: | :--- |
| `item_code` | string | ✅ | Unique item code |
| `item_name` | string | ✅ | Item display name |
| `category` | enum | ✅ | Must be uppercase. See [Enum Reference](#5-database-enum-reference) |

**Optional Fields (all have defaults):**

| Field | Type | Default | Allowed Values |
| :--- | :--- | :--- | :--- |
| `sub_category` | string | `null` | Free text |
| `description` | string | `null` | Free text |
| `cas_number` | string | `null` | CAS registry number |
| `molecular_formula` | string | `null` | e.g. `C6H12O6` |
| `concentration` | string | `null` | e.g. `500 mg/L` |
| `purity_grade` | string | `null` | e.g. `Analytical Grade` |
| `hazard_class` | enum | `NONE` | See [hazard_class enum](#5-database-enum-reference) |
| `msds_available` | boolean | `false` | |
| `requires_coc` | boolean | `false` | |
| `uom` | enum | `PIECE` | See [lab_uom enum](#5-database-enum-reference) |
| `pack_size` | string | `null` | e.g. `100 Tests` |
| `tests_per_unit` | integer | `null` | |
| `storage_condition` | enum | `ROOM_TEMPERATURE` | See [storage_condition enum](#5-database-enum-reference) |
| `storage_temperature_min` | float | `null` | °C |
| `storage_temperature_max` | float | `null` | °C |
| `is_light_sensitive` | boolean | `false` | |
| `is_moisture_sensitive` | boolean | `false` | |
| `shelf_life_months` | integer | `null` | |
| `brand` | string | `null` | |
| `manufacturer` | string | `null` | |
| `catalogue_number` | string | `null` | |
| `barcode` | string | `null` | |
| `preferred_vendor_id` | string | `null` | Vendor UUID |
| `mrp` | float | `null` | |
| `gst_percent` | float | `null` | |
| `lead_time_days` | integer | `7` | |
| `reorder_level` | integer | `0` | |
| `minimum_stock` | integer | `0` | |
| `maximum_stock` | integer | `null` | |
| `warehouse_id` | string | `null` | |
| `warehouse_name` | string | `null` | |
| `rack_shelf_bin` | string | `null` | |
| `assigned_department_id` | string | `null` | |
| `assigned_section` | string | `null` | |
| `linked_equipment_name` | string | `null` | |
| `linked_equipment_model` | string | `null` | |
| `is_controlled` | boolean | `false` | |
| `regulatory_approval` | string | `null` | |
| `recall_tracking` | boolean | `false` | |

**Minimal Request Example:**
```json
{
  "item_code": "REAG-GLU-001",
  "item_name": "Glucose Reagent Kit",
  "category": "REAGENT"
}
```

**Full Request Example:**
```json
{
  "item_code": "REAG-GLU-001",
  "item_name": "Glucose Reagent Kit",
  "category": "REAGENT",
  "sub_category": "Biochemistry",
  "description": "Enzymatic reagent kit for glucose estimation",
  "hazard_class": "NONE",
  "uom": "BOX",
  "storage_condition": "REFRIGERATED",
  "storage_temperature_min": 2,
  "storage_temperature_max": 8,
  "brand": "Abbott",
  "manufacturer": "Abbott Diagnostics",
  "mrp": 2850.50,
  "gst_percent": 18,
  "reorder_level": 10,
  "minimum_stock": 20
}
```

**Success Response:** `201 Created`
```json
{
  "success": true,
  "code": 201,
  "message": "Lab inventory item created successfully",
  "data": {
    "id": "692439fb-aaf1-49b0-88c4-83a0b22f540a",
    "item_code": "REAG-GLU-001",
    "item_name": "Glucose Reagent Kit",
    "category": "REAGENT",
    "sub_category": null,
    "hazard_class": "NONE",
    "uom": "BOX",
    "storage_condition": "REFRIGERATED",
    "lead_time_days": 7,
    "reorder_level": 0,
    "minimum_stock": 0,
    "is_controlled": false,
    "recall_tracking": false
  }
}
```

> ⚠️ **IMPORTANT:** Enum fields (`category`, `hazard_class`, `uom`, `storage_condition`) MUST be **UPPERCASE** values from the database enum. Passing `"Reagent"` instead of `"REAGENT"` will return: `invalid input value for database type/enum`.

---

### 4.6 Create Vendor

`POST /diagnostic-orders/vendors`

Registers a new lab vendor/supplier. Requires `diagnostics:test:manage` permission.

**Required Fields:**

| Field | Type | Required | Description |
| :--- | :--- | :---: | :--- |
| `vendor_name` | string | ✅ | Official vendor name |
| `official_email` | string | ✅ | Official contact email address |

**Optional Fields:**

| Field | Type | Default | Allowed Values / Description |
| :--- | :--- | :--- | :--- |
| `vendor_code` | string | `null` | Unique vendor identifier |
| `vendor_type` | string | `"DISTRIBUTOR"` | `"DISTRIBUTOR"`, `"IMPORTER"`, `"MANUFACTURER"`, `"RETAILER"`, `"WHOLESALER"` |
| `vendor_category` | string | `null` | Free text classification |
| `business_type` | string | `null` | Free text (e.g. LLC, Corp) |
| `year_established` | integer | `null` | Year established |
| `website_url` | string | `null` | Website address |
| `company_description` | string | `null` | Brief overview |
| `primary_contact` | string | `null` | Primary contact person name |
| `designation` | string | `null` | Contact person's designation |
| `mobile` | string | `null` | Mobile phone number |
| `alternate_mobile` | string | `null` | Alternate phone number |
| `support_email` | string | `null` | Customer support email |
| `landline` | string | `null` | Landline number |
| `whatsapp` | string | `null` | WhatsApp number |
| `office_address` | string | `null` | Street address |
| `city` | string | `null` | Office city |
| `state` | string | `null` | Office state |
| `pin_code` | string | `null` | Postal pin code |
| `gst_number` | string | `null` | GST tax registration number |
| `pan_number` | string | `null` | PAN tax registration number |
| `cin_number` | string | `null` | Corporate ID number |
| `drug_license_number`| string | `null` | Drug license certificate number |
| `drug_license_expiry`| string | `null` | ISO Date string format (YYYY-MM-DD) |
| `bank_name` | string | `null` | Bank name for payments |
| `account_holder` | string | `null` | Account holder name |
| `account_number` | string | `null` | Bank account number |
| `ifsc_code` | string | `null` | Bank IFSC routing code |
| `branch_name` | string | `null` | Bank branch name |
| `upi_id` | string | `null` | UPI payment ID |
| `payment_terms` | string | `null` | e.g. Net 30, COD |
| `credit_period_days`| integer | `30` | Allowed credit period in days |
| `preferred_payment_mode` | string| `null` | e.g. BANK_TRANSFER, UPI, CASH |
| `advance_payment_allowed` | boolean| `false` | Whether advance payments are supported |
| `delivery_coverage` | string | `null` | Delivery locations coverage |
| `avg_delivery_hours`| integer | `null` | Average delivery lead hours |
| `min_order_value` | float | `null` | Minimum purchase order value required |
| `emergency_supply` | boolean | `false` | Emergency delivery support flag |
| `cold_chain_supported` | boolean | `false` | Cold chain product support flag |
| `installation_support`| boolean | `false` | Equipment setup support flag |
| `amc_support` | boolean | `false` | Maintenance contract support flag |
| `nabl_iso_certified` | boolean | `false` | ISO validation flag |
| `gmp_certified` | boolean | `false` | Good Manufacturing Practice flag |
| `fda_approved` | boolean | `false` | FDA approved supplier flag |
| `biomedical_waste_compliance` | boolean | `false`| Waste protocols approval flag |
| `risk_level` | string | `"LOW"` | `"CRITICAL"`, `"HIGH"`, `"LOW"`, `"MEDIUM"` |
| `approval_authority`| string | `null` | Regulatory body approval details |
| `procurement_notes` | string | `null` | Internal procurement remarks |
| `vendor_status` | string | `"PENDING_VERIFICATION"`| `"ACTIVE"`, `"BLACKLISTED"`, `"PENDING_VERIFICATION"`, `"SUSPENDED"` |
| `is_active` | boolean | `true` | Vendor active status |

**Request Example:**
```json
{
  "vendor_name": "Bio Diagnostics Corp",
  "official_email": "orders@biodiagnostics.com",
  "vendor_type": "MANUFACTURER",
  "risk_level": "LOW",
  "credit_period_days": 45,
  "cold_chain_supported": true
}
```

**Success Response:** `201 Created`
```json
{
  "success": true,
  "code": 201,
  "message": "Vendor registered successfully",
  "data": {
    "id": "77f8a7ee-abaf-45dc-8441-9d2d4cd43f2c",
    "vendor_name": "Bio Diagnostics Corp",
    "vendor_code": "BLS-1002",
    "vendor_type": "MANUFACTURER",
    "official_email": "orders@biodiagnostics.com",
    "risk_level": "LOW",
    "vendor_status": "PENDING_VERIFICATION",
    "credit_period_days": 45,
    "cold_chain_supported": true,
    "is_active": true
  }
}
```

---

### 4.7 Create Purchase Order / GRN / Invoice

`POST /diagnostic-orders/procurement`

Unified endpoint to manage lab procurement. Use the `type` query parameter to determine which procurement record to create. Requires `diagnostics:test:manage` permission.

| `type` Query Parameter | Description |
| :--- | :--- |
| `po` *(default)* | Create a Purchase Order |
| `grn` | Create a Goods Received Note |
| `invoice` | Create a Vendor Invoice |

---

#### 4.7.1 Create Purchase Order (`?type=po`)

**Required Fields:**

| Field | Type | Required | Description |
| :--- | :--- | :---: | :--- |
| `items` | array | ✅ | List of Purchase Order items (min 1 item) |

*PO Line `items[]` details:*

| Field | Type | Required | Description |
| :--- | :--- | :---: | :--- |
| `item_name` | string | ✅ | Name of the catalog inventory item |
| `final_qty` | integer | ✅ | Quantity to order (must be > 0) |
| `unit_price` | float | ✅ | Unit price of the item (must be >= 0) |

**Optional Fields:**

| Field | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `vendor_id` | UUID string | `null` | Supplier UUID reference |
| `priority` | string | `"MEDIUM"`| Priority rating (e.g. LOW, MEDIUM, HIGH, URGENT) |
| `lab_section` | string | `null` | Section requesting the inventory (e.g. Hematology) |
| `store_location` | string | `null` | Intended store location |
| `delivery_address` | string | `null` | Office delivery address override |
| `expected_delivery` | string | `null` | ISO Date format (YYYY-MM-DD) |
| `notes` | string | `null` | Internal instructions |
| `items[].item_id` | UUID string | `null` | Catalog inventory item UUID |
| `items[].category` | string | `null` | e.g. REAGENT, CONSUMABLE |
| `items[].suggested_qty`| integer | `0` | Recommended restock quantity |
| `items[].uom` | string | `"PIECE"` | Unit of measurement |
| `items[].gst_percent` | float | `0` | GST tax percentage |

**Request Example:**
```json
{
  "vendor_id": "f197a875-efe9-432e-ac71-6356abad2efd",
  "priority": "HIGH",
  "lab_section": "Biochemistry",
  "notes": "Emergency restock of Glucose Reagents",
  "items": [
    {
      "item_id": "3221d3cd-367f-4c95-9e3d-e921e9bc7029",
      "item_name": "Glucose Reagent Kit",
      "final_qty": 10,
      "unit_price": 250.00,
      "gst_percent": 18.0
    }
  ]
}
```

**Success Response:** `201 Created`
```json
{
  "success": true,
  "code": 201,
  "message": "Purchase Order created successfully",
  "data": {
    "id": "4e4b4b51-408e-44b2-ac6e-1322be3d82f1",
    "po_number": "PO-LAB-46D603",
    "vendor_id": "f197a875-efe9-432e-ac71-6356abad2efd",
    "status": "DRAFT",
    "priority": "HIGH",
    "lab_section": "Biochemistry",
    "estimated_value": 2500.00,
    "gst_amount": 450.00,
    "grand_total": 2950.00,
    "created_at": "2026-07-16T17:30:00Z"
  }
}
```

---

#### 4.7.2 Create Goods Received Note (`?type=grn`)

**Required Fields:**

| Field | Type | Required | Description |
| :--- | :--- | :---: | :--- |
| `items` | array | ✅ | List of received items (min 1 item) |

*GRN Line `items[]` details:*

| Field | Type | Required | Description |
| :--- | :--- | :---: | :--- |
| `item_name` | string | ✅ | Name of the received item |
| `received_qty` | integer | ✅ | Quantity received from supplier (must be > 0) |

**Optional Fields:**

| Field | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `po_id` | UUID string | `null` | Reference purchase order UUID |
| `vendor_id` | UUID string | `null` | Supplier UUID reference |
| `notes` | string | `null` | Receiving notes |
| `items[].item_id` | UUID string | `null` | Catalog inventory item UUID |
| `items[].expected_qty` | integer | `null` | Quantity expected from PO |
| `items[].accepted_qty` | integer | `null` | Inspected quantity accepted |
| `items[].rejected_qty` | integer | `0` | Inspected quantity rejected |
| `items[].rejection_reason`| string | `null` | Reason for rejection |
| `items[].batch_number` | string | `null` | Manufacturing batch reference |
| `items[].lot_number` | string | `null` | Manufacturing lot reference |
| `items[].expiry_date` | string | `null` | Expiry Date (YYYY-MM-DD) |
| `items[].manufacturing_date`| string | `null` | Mfg Date (YYYY-MM-DD) |
| `items[].unit_price` | float | `null` | Item unit price |
| `items[].coa_received` | boolean | `false` | Certificate of Analysis status |
| `items[].qc_status` | string | `"PENDING"` | QC workflow status (e.g. PENDING, PASSED, FAILED) |

**Request Example:**
```json
{
  "po_id": "4e4b4b51-408e-44b2-ac6e-1322be3d82f1",
  "notes": "Received batch of Abbott kits",
  "items": [
    {
      "item_id": "3221d3cd-367f-4c95-9e3d-e921e9bc7029",
      "item_name": "Glucose Reagent Kit",
      "received_qty": 10,
      "batch_number": "B-ABB-992",
      "expiry_date": "2027-06-30"
    }
  ]
}
```

**Success Response:** `201 Created`
```json
{
  "success": true,
  "code": 201,
  "message": "Goods Received Note processed successfully",
  "data": {
    "id": "a52e974d-8e8d-48a6-9d60-d49777726b09",
    "grn_number": "GRN-LAB-00002",
    "po_id": "4e4b4b51-408e-44b2-ac6e-1322be3d82f1",
    "status": "PENDING_INSPECTION",
    "received_by": "8b7f50c4-fa3a-4aef-ab27-f42b7fb0e54f"
  }
}
```

---

#### 4.7.3 Create Vendor Invoice (`?type=invoice`)

**Required Fields:**

| Field | Type | Required | Description |
| :--- | :--- | :---: | :--- |
| `invoice_number` | string | ✅ | Vendor invoice reference identifier |
| `invoice_amount` | float | ✅ | Subtotal amount of the invoice (must be >= 0) |
| `total_amount` | float | ✅ | Grand total amount including tax (must be >= 0) |

**Optional Fields:**

| Field | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `vendor_id` | UUID string | `null` | Supplier UUID reference |
| `po_id` | UUID string | `null` | Associated Purchase Order UUID reference |
| `grn_id` | UUID string | `null` | Associated Goods Received Note UUID reference |
| `invoice_date` | string | `null` | Vendor invoice date (YYYY-MM-DD) |
| `gst_amount` | float | `0.00` | Tax component amount |

**Request Example:**
```json
{
  "invoice_number": "INV-ABB-9201",
  "po_id": "4e4b4b51-408e-44b2-ac6e-1322be3d82f1",
  "grn_id": "a52e974d-8e8d-48a6-9d60-d49777726b09",
  "invoice_amount": 2500.00,
  "gst_amount": 450.00,
  "total_amount": 2950.00
}
```

**Success Response:** `201 Created`
```json
{
  "success": true,
  "code": 201,
  "message": "Vendor Invoice created successfully",
  "data": {
    "id": "93e36d22-5727-44c5-8b45-ec6c3b5953d2",
    "invoice_number": "INV-ABB-9201",
    "total_amount": 2950.00,
    "match_status": "MATCHED",
    "payment_status": "UNPAID"
  }
}
```

---

### 4.8 Create Role

`POST /diagnostic-orders/roles`

Create a new access control role. Requires `roles.write` permission.

**Required Fields:**

| Field | Type | Required | Description |
| :--- | :--- | :---: | :--- |
| `roleName` | string | ✅ | Name of the role |
| `permissions` | array of strings| ✅ | Permitted action tags (e.g. `config.read`, `roles.view`) |

**Optional Fields:**

| Field | Type | Default | Allowed Values / Description |
| :--- | :--- | :--- | :--- |
| `description` | string | `null` | Role responsibilities |
| `status` | string | `"ACTIVE"` | `"ACTIVE"`, `"INACTIVE"` |
| `maxUsers` | integer | `null` | Max users allowed to hold role (must be > 0) |
| `sessionTimeout`| integer | `30` | Autologout timeout in minutes |
| `mfaRequired` | boolean | `false` | Multi-factor authentication flag |

**Request Example:**
```json
{
  "roleName": "LAB_ASSISTANT",
  "description": "Assistant for sample collection and labeling",
  "permissions": [
    "diagnostics:order:view",
    "diagnostics:lab:collect"
  ],
  "maxUsers": 10,
  "mfaRequired": false
}
```

**Success Response:** `201 Created`
```json
{
  "success": true,
  "code": 201,
  "data": {
    "roleId": "ROLE-LAB-AST",
    "roleName": "LAB_ASSISTANT",
    "description": "Assistant for sample collection and labeling",
    "status": "ACTIVE",
    "maxUsers": 10,
    "sessionTimeout": 30,
    "mfaRequired": false,
    "permissions": [
      "diagnostics:order:view",
      "diagnostics:lab:collect"
    ]
  }
}
```

---

### 4.9 Create Config (Report Template)

`POST /diagnostic-orders/config`

Create a new laboratory report template layout configuration. Requires `config.write` permission.

**Required Fields:**

| Field | Type | Required | Description |
| :--- | :--- | :---: | :--- |
| `templateName` | string | ✅ | Name of the template (min 2 chars) |
| `category` | string | ✅ | Test department/category (e.g. Biochemistry, Hematology) |
| `layoutJson` | object | ✅ | Dynamic JSON mapping layout structure for rendering |

**Request Example:**
```json
{
  "templateName": "Hematology Routine Report Template",
  "category": "Hematology",
  "layoutJson": {
    "header": {
      "showLogo": true,
      "hospitalName": "Arovita Labs"
    },
    "body": {
      "columns": ["Parameter", "Result", "Reference Range", "Unit"]
    },
    "footer": {
      "signatories": ["Dr. Amit Shah (MD, Pathologist)"]
    }
  }
}
```

**Success Response:** `201 Created`
```json
{
  "success": true,
  "code": 201,
  "data": {
    "templateId": "d766545b-c6b0-4abc-aa17-49cb0be25e08",
    "templateName": "Hematology Routine Report Template",
    "category": "Hematology",
    "status": "DRAFT",
    "isDefault": false,
    "lastUpdated": "2026-07-16T17:45:00Z"
  }
}
```

---

## 5. Database Enum Reference

> ⚠️ All enum values are **UPPERCASE**. Passing mixed-case values will cause: `invalid input value for database type/enum`.

### `category` — `catalogue.lab_item_category`

| Value | Description |
| :--- | :--- |
| `REAGENT` | Chemical reagents |
| `CHEMICAL` | Lab chemicals |
| `CONSUMABLE` | Disposable consumables |
| `STAINING_SUPPLY` | Staining kits & supplies |
| `CULTURE_MEDIA` | Microbiology culture media |
| `CALIBRATOR` | Calibration materials |
| `CONTROL` | QC control materials |
| `STANDARD` | Reference standards |
| `EQUIPMENT_SPARE` | Equipment spare parts |
| `GLASSWARE` | Lab glassware |
| `PLASTICWARE` | Lab plasticware |
| `SAFETY_SUPPLY` | PPE & safety equipment |
| `OTHER` | Other items |

### `hazard_class` — `catalogue.hazard_class`

| Value | Description |
| :--- | :--- |
| `NONE` | Non-hazardous |
| `FLAMMABLE` | Flammable material |
| `CORROSIVE` | Corrosive material |
| `TOXIC` | Toxic substance |
| `OXIDIZER` | Oxidizing agent |
| `BIOHAZARD` | Biological hazard |
| `RADIOACTIVE` | Radioactive material |
| `IRRITANT` | Irritant |
| `CARCINOGEN` | Carcinogenic substance |
| `EXPLOSIVE` | Explosive material |

### `uom` — `catalogue.lab_uom`

| Value | Description |
| :--- | :--- |
| `PIECE` | Individual piece |
| `BOX` | Box |
| `PACK` | Pack |
| `BOTTLE` | Bottle |
| `VIAL` | Vial |
| `AMPOULE` | Ampoule |
| `KIT` | Kit |
| `LITRE` | Litre |
| `MILLILITRE` | Millilitre |
| `GRAM` | Gram |
| `KILOGRAM` | Kilogram |
| `MILLIGRAM` | Milligram |
| `STRIP` | Strip |
| `ROLL` | Roll |
| `SET` | Set |
| `EACH` | Each |
| `PAIR` | Pair |
| `CARTRIDGE` | Cartridge |
| `CASSETTE` | Cassette |
| `TUBE` | Tube |
| `PLATE` | Plate |
| `SLIDE` | Slide |
| `OTHER` | Other |

### `storage_condition` — `catalogue.storage_condition`

| Value | Description |
| :--- | :--- |
| `ROOM_TEMPERATURE` | 15–25 °C |
| `REFRIGERATED` | 2–8 °C |
| `FROZEN` | -20 °C |
| `COLD_CHAIN` | Continuous cold chain |
| `HUMIDITY_CONTROLLED` | Humidity-controlled storage |
| `DUST_FREE` | Dust-free environment |

### `order_type` — Diagnostic Order Type

| Value |
| :--- |
| `LAB` |
| `RADIOLOGY` |
| `CARDIOLOGY` |
| `PATHOLOGY` |

### `priority` — Order Priority

| Value |
| :--- |
| `ROUTINE` |
| `URGENT` |
| `STAT` |

### `lab_po_status` — Purchase Order Status

| Value |
| :--- |
| `DRAFT` |
| `PENDING_APPROVAL` |
| `APPROVED` |
| `ORDERED` |
| `PARTIALLY_RECEIVED` |
| `RECEIVED` |
| `CANCELLED` |
| `CLOSED` |

### `lab_grn_status` — GRN Status

| Value |
| :--- |
| `PENDING_INSPECTION` |
| `QC_IN_PROGRESS` |
| `ACCEPTED` |
| `PARTIALLY_ACCEPTED` |
| `REJECTED` |
| `QUARANTINED` |

---

## 6. Error Responses

### 400 — Bad Request (Validation Error)
```json
{
  "success": false,
  "code": 400,
  "message": "Invalid request data: invalid input value for database type/enum",
  "errors": []
}
```

### 401 — Unauthorized
```json
{
  "success": false,
  "code": 401,
  "message": "Authentication required",
  "errors": []
}
```

### 403 — Forbidden
```json
{
  "success": false,
  "code": 403,
  "message": "Forbidden — 'waste.view' permission required",
  "errors": []
}
```

### 404 — Not Found
```json
{
  "success": false,
  "code": 404,
  "message": "Resource not found",
  "errors": []
}
```

### 500 — Internal Server Error
```json
{
  "success": false,
  "code": 500,
  "message": "Internal server error",
  "errors": []
}
```

---

## Complete Route Map

| Method | Path | Handler | Permission |
| :--- | :--- | :--- | :--- |
| `GET` | `/orders` | `list_lab_orders` | `diagnostics:order:view` |
| `POST` | `/orders` | `create_registration` | `diagnostics:order:view` |
| `GET` | `/orders/{id}` | `get_lab_order` | `diagnostics:order:view` |
| `PATCH` | `/orders/{id}` | `update_order` | — |
| `PATCH` | `/orders/{id}/status` | `update_order_status` | — |
| `POST` | `/orders/{id}/actions` | `execute_order_action` | varies by action |
| `POST` | `/orders/bulk/actions` | `execute_orders_bulk_action` | varies by action |
| `GET` | `/lookup` | `get_lookup_data` | varies by type |
| `GET` | `/dashboard` | `get_lab_analytics_dashboard` | `diagnostics:analytics:view` |
| `POST` | `/tests` | `create_lab_test` | `config.write` |
| `GET` | `/tests/{id}` | `get_catalog_test` | `config.read` |
| `PATCH` | `/tests/{id}` | `update_catalog_test` | `config.write` |
| `DELETE` | `/tests/{id}` | `delete_catalog_test` | `config.write` |
| `GET` | `/inventory` | `list_lab_inventory` | `diagnostics:order:view` |
| `POST` | `/inventory` | `create_lab_item` | `diagnostics:test:manage` |
| `GET` | `/inventory/{id}` | `get_inventory_item` | `diagnostics:order:view` |
| `PATCH` | `/inventory/{id}` | `update_inventory_item` | `diagnostics:test:manage` |
| `POST` | `/inventory/{id}/actions` | `execute_inventory_action` | varies |
| `GET` | `/vendors` | `list_lab_vendors` | `diagnostics:order:view` |
| `POST` | `/vendors` | `create_lab_vendor` | `diagnostics:test:manage` |
| `GET` | `/vendors/{id}` | `get_vendor_profile` | `diagnostics:order:view` |
| `PATCH` | `/vendors/{id}` | `update_vendor_profile` | `diagnostics:test:manage` |
| `DELETE` | `/vendors/{id}` | `delete_vendor_profile` | `diagnostics:test:manage` |
| `POST` | `/vendors/{id}/actions` | `execute_vendor_action` | varies |
| `GET` | `/procurement` | `list_procurement_records` | varies |
| `POST` | `/procurement` | `create_procurement_record` | `diagnostics:test:manage` |
| `GET` | `/procurement/{id}` | `get_procurement_record` | varies |
| `PATCH` | `/procurement/{id}` | `update_procurement_record` | `diagnostics:test:manage` |
| `POST` | `/procurement/{id}/actions` | `execute_procurement_action` | — |
| `GET` | `/billing/invoices` | `list_lab_invoices` | `diagnostics:billing:view` |
| `GET` | `/billing/invoices/{id}` | `get_billing_invoice` | `diagnostics:billing:view` |
| `PATCH` | `/billing/invoices/{id}` | `update_billing_invoice` | `diagnostics:billing:manage` |
| `POST` | `/billing/invoices/{id}/actions` | `execute_billing_invoice_action` | varies |
| `GET` | `/staff` | `list_lab_staff` | `staff.read` |
| `GET` | `/staff/{id}` | `get_staff_profile` | `staff.read` |
| `PATCH` | `/staff/{id}` | `update_staff_profile` | — |
| `POST` | `/staff/{id}/actions` | `execute_staff_action` | varies |
| `GET` | `/roles` | `list_lab_access_roles` | `roles.read` |
| `POST` | `/roles` | `create_lab_access_role` | `roles.write` |
| `GET` | `/roles/{id}` | `get_role_details` | `roles.read` |
| `PATCH` | `/roles/{id}` | `update_role_details` | `roles.write` |
| `DELETE` | `/roles/{id}` | `delete_role` | `roles.write` |
| `POST` | `/roles/{id}/actions` | `execute_role_action` | `roles.write` |
| `GET` | `/config` | `list_config_settings` | `config.read` |
| `POST` | `/config` | `create_config_settings` | `config.write` |
| `GET` | `/config/{id}` | `get_config_settings` | `config.read` |
| `PATCH` | `/config/{id}` | `update_config_settings` | `config.write` |
| `DELETE` | `/config/{id}` | `delete_config_settings` | `config.write` |
| `POST` | `/config/{id}/actions` | `execute_config_action` | `config.write` |
| `GET` | `/audit` | `list_audit_records` | `audit.read` |
| `GET` | `/audit/{id}` | `get_audit_details` | `audit.read` |
| `POST` | `/audit/actions` | `execute_audit_action` | varies |
