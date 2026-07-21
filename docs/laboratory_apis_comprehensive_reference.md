# Laboratory Module - Comprehensive GET API Reference

This document provides a detailed catalog of all `GET` API endpoints across the Laboratory module (consisting of 9 key functional areas), outlining path variables, query parameters, expected responses, and the specific controller/service files where they are implemented.

---

## 1. Dashboard Overview

### A. GET Dashboard Analytics & Stats
* **URL**: `GET /diagnostic-orders/lab/dashboard`
* **Purpose**: Fetches high-level metrics (KPI counters, volume/revenue trends, and category distribution) to populate the Manager/Tech Overview screens.
* **Query Parameters**: None
* **Implementation File**:
  * Route: `services/lab/app/routes/lab/analytics.py` -> `get_lab_analytics_dashboard`
  * Service: `services/lab/app/core/lab_analytics_service.py` -> `AnalyticsService.get_dashboard`
* **Response Body (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "kpis": {
      "avg_tat": {"hours": 2.4},
      "pending_reports": 37,
      "critical_results": 4
    },
    "volume_trend": [
      {"month": "Jul", "count": 142}
    ],
    "revenue_trend": [
      {"month": "Jul", "amount": 84200.0}
    ],
    "category_distribution": [
      {"category": "Biochemistry", "percentage": 42.5}
    ]
  }
}
```

### B. GET Technician Dashboard List
* **URL**: `GET /diagnostic-orders/lab/dashboard/technician`
* **Purpose**: Lists patients with active orders waiting for sample collection or test processing, grouped by department and specimen.
* **Query Parameters**: None
* **Implementation File**:
  * Route: `services/lab/app/routes/lab/registration.py` -> `get_technician_dashboard`
  * Service: `services/lab/app/core/lab/registration_service.py` -> `get_technician_dashboard`
* **Response Body (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "order_id": "3cc7db49-490e-4c6a-897e-809d45aeb093",
      "lab_unique_id": "LABID-29226",
      "uhid": "PAT-2026-0005",
      "patient_name": "Rajesh Sharma",
      "age": 42,
      "gender": "MALE",
      "mobile_number": "+919953847134",
      "departments": [
        {
          "department_name": "Hematology",
          "sample_type": "BLOOD",
          "sample_status": "PENDING",
          "sample_barcode": "BC-10049",
          "tests": [
            {
              "test_id": "e5e82d23-96f8-4981-b068-52a12dc10de4",
              "test_name": "Complete Blood Count",
              "status": "PENDING"
            }
          ]
        }
      ]
    }
  ]
}
```

### C. GET Unified Metadata Lookup Facade
* **URL**: `GET /diagnostic-orders/lookup` (Alias: `GET /lab/lookup`)
* **Purpose**: Acts as a unified gateway query facade. Depending on the value of the `type` query parameter, it dynamically routes the lookup query to the corresponding master catalog list.
* **Query Parameters**:
  * `type` (optional, default: `dropdowns`): Determines the target master catalog to load.
* **Lookup Redirect Mappings**:
  * `?type=dropdowns`: General Form Metadata (Visit Types, Urgency, Specimens) -> redirects to `initialize_dropdowns`
  * `?type=tests`: Lab Catalog Tests List -> redirects to `list_lab_tests`
  * `?type=packages`: Test Packages Catalog List -> redirects to `list_test_packages`
  * `?type=machines`: Connected Laboratory Analyzers -> redirects to `list_machines`
  * `?type=printers`: Hospital Report/Barcode Printers -> redirects to `list_printers`
  * `?type=report-templates`: PDF Report Design Templates -> redirects to `list_report_templates`
  * `?type=departments`: Hospital Referral Departments List -> redirects to `list_lab_departments`
  * `?type=shifts`: Lab Staff Shift Schedules -> redirects to `list_lab_shifts`
  * `?type=permissions`: Laboratory RBAC Permissions list -> redirects to `list_lab_access_permissions`
* **Implementation File**:
  * Route: `services/lab/app/routes/lab/registration.py` -> `get_lookup_data`
  * Service: Routes dynamically to other sub-services based on query.
* **Response Body (200 OK for type=dropdowns)**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "visit_types": ["Whole Blood", "Serum", "Plasma", "Urine", "Swab"],
    "priorities": ["Routine", "Urgent", "Stat"],
    "sample_types": ["Whole Blood", "Serum", "Plasma", "Urine", "Swab"],
    "departments": [
      {
        "id": "46d19155-09a1-462e-b256-ce2a5741b091",
        "name": "General Medicine"
      }
    ]
  }
}
```

---

## 2. Inventory Portal

### A. List All Inventory Items
* **URL**: `GET /diagnostic-orders/inventory`
* **Purpose**: Paginated list of lab reagents, tubes, gloves, and other consumables.
* **Query Parameters**:
  * `search` (optional): Matches the `item_name` or description (case-insensitive substring match). E.g. `?search=Glucose`.
  * `category` (optional): Filters by item category (e.g. `Consumables`, `Reagents`).
  * `supplier` (optional): Filters by vendor/supplier name (e.g. `Global`).
  * `status` (optional): Filters stock levels (e.g. `Healthy`, `Reorder`, `Critical`).
  * `expiry` (optional): Filters item batches by shelf-life status (e.g. `Expired`, `Near Expiry`).
  * `page` (optional, default: `1`): Current page offset.
  * `limit` (optional, default: `10`): Records per page.
  * `sortBy` (optional, default: `item_name`): Sorts list (`item_name`, `category`, `current_stock`).
  * `sortOrder` (optional, default: `ASC`): Sort order (`ASC` or `DESC`).
* **Real Database Examples for Testing**:
  * Try searching for: `?search=Glucose`, `?search=Reagent`, or `?search=Verification` (since `"EDTA"` placeholder doesn't exist).
* **Implementation File**:
  * Route: `services/lab/app/routes/lab/manager.py` -> `list_lab_inventory`
  * Service: `services/lab/app/core/lab_inventory_service.py` -> `InventoryService.list_items`
* **Response Body (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "id": "192d502b-564a-4321-89b7-8243f61311c6",
        "item_name": "Glucose Reagent Kit",
        "category": "Reagents",
        "current_stock": 12,
        "reorder_level": 5,
        "status": "Healthy"
      }
    ],
    "total": 1
  }
}
```

### B. Get Single Inventory Item Details
* **URL**: `GET /diagnostic-orders/inventory/{id}`
* **Path Parameters**:
  * `id` (required, UUID of inventory item)
* **Real Database IDs for Testing**:
  * `192d502b-564a-4321-89b7-8243f61311c6` *(for item: "heyy")*
  * `3221d3cd-367f-4c95-9e3d-e921e9bc7029` *(for item: "Verification Reagent Solution")*
* **Implementation File**:
  * Route: `services/lab/app/routes/lab/manager.py` -> `get_lab_inventory_item`
  * Service: `services/lab/app/core/lab_inventory_service.py` -> `InventoryService.get_item_details`
* **Response Body (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "192d502b-564a-4321-89b7-8243f61311c6",
    "item_name": "heyy",
    "category": "Reagents",
    "current_stock": 12,
    "unit": "pcs",
    "supplier_name": "Fisher Scientific"
  }
}
```

### C. Get Inventory Alerts / Low Stock
* **URL**: `GET /diagnostic-orders/lab/inventory/statistics`
* **Purpose**: Fetches alert metrics and items under critical reorder levels.
* **Query Parameters**: None
* **Implementation File**:
  * Route: `services/lab/app/routes/lab/manager.py` -> `get_lab_inventory_statistics`
  * Service: `services/lab/app/core/lab_inventory_service.py` -> `InventoryService.get_statistics`
* **Response Body (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "low_stock_count": 3,
    "critical_items": [
      {
        "id": "abc-123",
        "item_name": "CRP Reagent Kit",
        "current_stock": 5
      }
    ]
  }
}
```

---

## 3. Vendor Management

### A. List All Vendors
* **URL**: `GET /diagnostic-orders/vendors`
* **Purpose**: Retrieves all partner lab equipment and reagent suppliers.
* **Query Parameters**:
  * `search` (optional)
* **Implementation File**:
  * Route: `services/lab/app/routes/lab/vendors.py` -> `list_vendors`
  * Service: `services/lab/app/core/lab_vendor_service.py` -> `VendorService.list_vendors`
* **Response Body (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "vendor_id": "v1004",
      "vendor_name": "Fisher Scientific",
      "contact_person": "James Wilson",
      "phone": "+14083391823",
      "status": "ACTIVE"
    }
  ]
}
```

### B. Get Single Vendor Profile
* **URL**: `GET /diagnostic-orders/vendors/{id}`
* **Path Parameters**:
  * `id` (required, UUID of vendor record)
* **Implementation File**:
  * Route: `services/lab/app/routes/lab/vendors.py` -> `get_vendor_profile`
  * Service: `services/lab/app/core/lab_vendor_service.py` -> `VendorService.get_vendor_profile`
* **Response Body (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "vendor_id": "v1004",
    "vendor_name": "Fisher Scientific",
    "contact_email": "sales@fisher.com",
    "address": "San Jose, CA"
  }
}
```

---

## 4. Purchase & Procurement

### A. List Procurement Records / Purchase Orders
* **URL**: `GET /diagnostic-orders/procurement`
* **Purpose**: Tracks procurement purchase orders for reagents and equipment.
* **Query Parameters**:
  * `status` (optional): Filters purchase orders by current lifecycle stage. Supported statuses include:
    * `DRAFT` (Initial requisition created)
    * `PENDING` (Awaiting approval)
    * `APPROVED` (Approved but not sent)
    * `ORDERED` (PO sent to vendor)
    * `RECEIVED` (All items delivered and checked in)
    * `CANCELLED` (Requisition cancelled)
  * `type` (optional, default: `po`): Set `type=grn` to fetch Goods Receipt Notes or `type=invoice` to list vendor invoices.
* **Implementation File**:
  * Route: `services/lab/app/routes/lab/manager.py` -> `list_procurement_records` (which routes to `list_lab_pos`)
  * Service: `services/lab/app/core/lab_inventory_service.py` -> `list_pos`
* **Response Body (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "po_id": "PO-2026-0045",
      "vendor_name": "Fisher Scientific",
      "order_date": "2026-07-10",
      "total_amount": 12500.0,
      "status": "APPROVED"
    }
  ]
}
```

### B. Get Single Procurement Record
* **URL**: `GET /diagnostic-orders/procurement/{id}`
* **Path Parameters**:
  * `id` (required, UUID of purchase order)
* **Implementation File**:
  * Route: `services/lab/app/routes/lab/procurement.py` -> `get_procurement_record`
  * Service: `services/lab/app/core/lab_procurement_service.py` -> `ProcurementService.get_order_details`
* **Response Body (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "po_id": "PO-2026-0045",
    "items": [
      {"item_name": "EDTA Tubes", "quantity": 100, "unit_price": 5.0}
    ]
  }
}
```

### C. Get Procurement/Vendor Stats
* **URL**: `GET /diagnostic-orders/lab/sample-management/statistics`
* **Purpose**: Pulls overall procurement metrics and fulfillment times.
* **Query Parameters**: None
* **Implementation File**:
  * Route: `services/lab/app/routes/lab/manager.py` -> `get_sample_management_statistics`
  * Service: `services/lab/app/core/lab/manager_service.py` -> `get_sample_management_statistics`
* **Response Body (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "total_vendors": 12,
    "pending_deliveries": 3
  }
}
```

---

## 5. Billing & Payments

### A. List All Billing Invoices
* **URL**: `GET /diagnostic-orders/billing/invoices`
* **Purpose**: List patient invoice transactions.
* **Query Parameters**:
  * `status` (optional, e.g. PAID, UNPAID)
* **Implementation File**:
  * Route: `services/lab/app/routes/lab/billing.py` -> `list_billing_invoices`
  * Service: `services/lab/app/core/lab_billing_service.py` -> `BillingService.list_invoices`
* **Response Body (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "invoice_number": "INV-LAB-8902",
      "patient_name": "E2E Simulation Patient",
      "net_amount": 368.0,
      "status": "PAID"
    }
  ]
}
```

### B. Get Single Billing Invoice
* **URL**: `GET /diagnostic-orders/billing/invoices/{id}`
* **Path Parameters**:
  * `id` (required, UUID of invoice)
* **Implementation File**:
  * Route: `services/lab/app/routes/lab/billing.py` -> `get_billing_invoice`
  * Service: `services/lab/app/core/lab_billing_service.py` -> `BillingService.get_invoice_details`
* **Response Body (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "invoice_number": "INV-LAB-8902",
    "total_amount": 350.0,
    "tax_amount": 18.0,
    "payment_mode": "UPI"
  }
}
```

### C. Get Lab Order Billing Detail
* **URL**: `GET /diagnostic-orders/lab/registrations/{id}/billing`
* **Path Parameters**:
  * `id` (required, UUID of registration/order)
* **Implementation File**:
  * Route: `services/lab/app/routes/lab/registration.py` -> `get_registration_billing`
  * Service: `services/lab/app/core/lab/registration_service.py` -> uses `_lab_repo.get_billing_record`
* **Response Body (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "e22af11c-89a1-4322-bc08-f82db6dcfa2a",
    "invoice_number": "INV-LAB-8902",
    "status": "PAID",
    "total_amount": 350.0,
    "tax_amount": 18.0,
    "paid_amount": 368.0,
    "outstanding": 0.0
  }
}
```

---

## 6. Analytics & MIS

### A. Get Lab Test Processing Statistics
* **URL**: `GET /diagnostic-orders/lab/test-processing/statistics`
* **Purpose**: Fetches active analyzer throughput metrics.
* **Query Parameters**: None
* **Implementation File**:
  * Route: `services/lab/app/routes/lab/manager.py` -> `get_test_processing_statistics`
  * Service: `services/lab/app/core/lab/manager_service.py` -> `get_test_processing_statistics`
* **Response Body (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "total_completed_today": 34,
    "active_in_progress": 3
  }
}
```

### B. Get Result Submission Statistics
* **URL**: `GET /diagnostic-orders/lab/result-submission/statistics`
* **Purpose**: Tracks validation review queue parameters.
* **Query Parameters**: None
* **Implementation File**:
  * Route: `services/lab/app/routes/lab/manager.py` -> `get_result_submission_statistics`
  * Service: `services/lab/app/core/lab/manager_service.py` -> `get_result_submission_statistics`
* **Response Body (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "pending_pathologist_review": 10,
    "critical_alerts": 0
  }
}
```

### C. Get Pathologist Validation Stats
* **URL**: `GET /diagnostic-orders/lab/results/statistics`
* **Purpose**: Retrieves review queues, approved volumes, and critical flag counts for the validation view.
* **Query Parameters**: None
* **Implementation File**:
  * Route: `services/lab/app/routes/lab/pathologist.py` -> `get_pathologist_statistics`
  * Service: `services/lab/app/core/lab/pathologist_service.py` -> `get_pathologist_statistics`
* **Response Body (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "reports_generated": 1,
    "pending_verification": 10,
    "critical_results": 0,
    "approved_today": 0
  }
}
```

### D. Get Reports Delivery Statistics
* **URL**: `GET /lab/results/statistics`
* **Purpose**: Measures diagnostic report release compliance times.
* **Query Parameters**: None
* **Implementation File**:
  * Route: `services/lab/app/routes/lab/pathologist.py` -> `get_reports_delivery_statistics`
  * Service: `services/lab/app/core/lab/pathologist_service.py` -> `get_reports_delivery_statistics`
* **Response Body (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "validated_reports": 45,
    "released_reports": 45,
    "compliance_rate": 94.2
  }
}
```

---

## 7. Audit & Compliance

### A. List Audit Logs
* **URL**: `GET /diagnostic-orders/audit`
* **Purpose**: Access log entries of user interactions, modifications, and system events.
* **Query Parameters**:
  * `page` (optional)
  * `limit` (optional)
* **Implementation File**:
  * Route: `services/lab/app/routes/lab/audit.py` -> `list_audit_records`
  * Service: `services/lab/app/core/lab_audit_service.py` -> `LabAuditService.list_logs`
* **Response Body (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "id": "aud-1029",
      "user_name": "Riya Jain",
      "action": "RESULT_SUBMITTED",
      "timestamp": "2026-07-17T23:48:40Z"
    }
  ]
}
```

### B. Get Single Audit Details
* **URL**: `GET /diagnostic-orders/audit/{id}`
* **Path Parameters**:
  * `id` (required, UUID of audit log)
* **Implementation File**:
  * Route: `services/lab/app/routes/lab/audit.py` -> `get_audit_details`
  * Service: `services/lab/app/core/lab_audit_service.py` -> `LabAuditService.get_log_details`
* **Response Body (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "aud-1029",
    "user_name": "Riya Jain",
    "action": "RESULT_SUBMITTED",
    "old_values": null,
    "new_values": {"status": "READY_FOR_REVIEW"}
  }
}
```

---

## 8. Staff Management

### A. List Staff Profile Directory
* **URL**: `GET /diagnostic-orders/staff`
* **Purpose**: Directory of laboratory phlebotomists, technicians, managers, and pathologists.
* **Query Parameters**: None
* **Implementation File**:
  * Route: `services/lab/app/routes/lab/staff.py` -> `list_staff_profiles`
  * Service: `services/lab/app/core/lab_staff_service.py` -> `StaffService.list_staff`
* **Response Body (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "staff_id": "st-449",
      "full_name": "Riya Jain",
      "role": "Technician",
      "status": "ACTIVE"
    }
  ]
}
```

### B. Get Single Staff Profile
* **URL**: `GET /diagnostic-orders/staff/{id}`
* **Path Parameters**:
  * `id` (required, UUID of staff user record)
* **Implementation File**:
  * Route: `services/lab/app/routes/lab/staff.py` -> `get_staff_profile`
  * Service: `services/lab/app/core/lab_staff_service.py` -> `StaffService.get_staff_profile`
* **Response Body (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "staff_id": "st-449",
    "full_name": "Riya Jain",
    "email": "riya.jain@ljbhospital.com",
    "phone": "+919902341902"
  }
}
```

### C. List Technician Bench Assignments
* **URL**: `GET /diagnostic-orders/lab/bench-assignments`
* **Purpose**: Gets active task bench assignments for technicians.
* **Query Parameters**: None
* **Implementation File**:
  * Route: `services/lab/app/routes/lab/manager.py` -> `list_bench_assignments`
  * Service: `services/lab/app/core/lab/manager_service.py` -> `list_bench_assignments`
* **Response Body (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "bench_id": "BENCH-02",
      "department_name": "Hematology",
      "assigned_technician": "Riya Jain"
    }
  ]
}
```

---

## 9. System Configuration

### A. List Configuration Settings
* **URL**: `GET /diagnostic-orders/config`
* **Purpose**: List lab configuration properties.
* **Query Parameters**: None
* **Implementation File**:
  * Route: `services/lab/app/routes/lab/config.py` -> `list_config_settings`
  * Service: `services/lab/app/core/lab_config_service.py` -> `ConfigService.list_settings`
* **Response Body (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "setting_key": "LAB_REPORT_HEADER_URL",
      "value": "https://s3.amazonaws.com/ljb/header.png"
    }
  ]
}
```

### B. Get Single Config Setting
* **URL**: `GET /diagnostic-orders/config/{id}`
* **Path Parameters**:
  * `id` (required, key/UUID of config setting)
* **Response Body (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "setting_key": "LAB_REPORT_HEADER_URL",
    "value": "https://s3.amazonaws.com/ljb/header.png"
  }
}
```

### C. List User Access Roles
* **URL**: `GET /diagnostic-orders/roles`
* **Purpose**: Access role groups for permission assignment.
* **Query Parameters**: None
* **Response Body (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "role_id": "LAB-005",
      "role_name": "PATHOLOGIST",
      "is_super_admin": false
    }
  ]
}
```

### D. Get Role Details
* **URL**: `GET /diagnostic-orders/roles/{id}`
* **Path Parameters**:
  * `id` (required, ID of role)
* **Response Body (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "role_id": "LAB-005",
    "role_name": "PATHOLOGIST",
    "permissions": ["diagnostics:order:view", "diagnostics:lab:result:validate"]
  }
}
```

### E. List Available Lab Catalog Tests
* **URL**: `GET /diagnostic-orders/lab/tests`
* **Purpose**: Test catalog for checkout (e.g. Complete Blood Count, Lipid Profile).
* **Query Parameters**: None
* **Implementation File**:
  * Route: `services/lab/app/routes/lab/registration.py` -> `list_lab_tests`
  * Service: `services/lab/app/core/lab/registration_service.py` -> `list_lab_tests`
* **Response Body (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "test_id": "e5e82d23-96f8-4981-b068-52a12dc10de4",
      "test_name": "Blood Culture & Sensitivity",
      "specimen_type": "BLOOD",
      "price": 350.0
    }
  ]
}
```

### F. Get Lab Test Catalog Detail
* **URL**: `GET /diagnostic-orders/tests/{id}`
* **Path Parameters**:
  * `id` (required, UUID of test in catalog)
* **Response Body (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "e5e82d23-96f8-4981-b068-52a12dc10de4",
    "test_name": "Blood Culture & Sensitivity",
    "specimen_type": "BLOOD",
    "price": 350.0,
    "normal_range": "No growth",
    "unit": "Report"
  }
}
```
