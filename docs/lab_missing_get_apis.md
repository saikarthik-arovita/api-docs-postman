# HMS — Laboratory Service GET APIs Reference Documentation
Version: 1.0.0
Workspace Target: HMS Diagnostics Microservice (Lab Module)

This document provides complete, production-ready specifications for all GET endpoints required for your dashboards. Each endpoint contains the HTTP method, request headers, query/path parameters, and an untruncated success JSON response.

---

## Global Headers
All GET requests require the following request headers:
```http
Authorization: Bearer <cognito_jwt_token>
Content-Type: application/json
Accept: application/json
```

---

## 1. Dashboard & Analytics Endpoints

### 1.1 GET /dashboard
* **Description**: Retrieves overview metrics count (pending, processing, and completed counts) for the laboratory dashboard.
* **Request URL**: `GET /diagnostic-orders/dashboard`
* **Required Permission**: None (Authenticated only)
* **Success Response (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "pending_orders": 8,
    "in_progress_orders": 12,
    "completed_orders": 45,
    "cancelled_orders": 3,
    "pending_samples": 4,
    "sample_collected": 6,
    "in_progress": 8,
    "completed": 45,
    "cancelled": 3,
    "total_results": 45,
    "abnormal_results": 7,
    "critical_results": 2,
    "pending_validation": 5
  }
}
```

---

### 1.2 GET /dashboard/kpis
* **Description**: Returns quick key performance indicators (KPIs) for the operational pipeline.
* **Request URL**: `GET /diagnostic-orders/dashboard`
* **Success Response (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "tat_compliance_rate": "94.2%",
    "average_processing_time_hours": 2.5,
    "critical_alerts_pending": 2,
    "reagent_alerts_triggered": 0
  }
}
```

---

### 1.3 GET /dashboard/recent-registrations
* **Description**: Returns the last 5 registered laboratory patient admission records.
* **Request URL**: `GET /diagnostic-orders/orders?limit=5`
* **Success Response (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "order_id": "6a9e1088-7576-463a-ae62-386e97802b26",
      "patient_name": "Rajesh Sharma",
      "patient_mrn": "UHID-9821",
      "test_name": "Fasting Blood Sugar",
      "priority": "ROUTINE",
      "created_at": "2026-07-16T12:00:00Z"
    }
  ]
}
```

---

### 1.4 GET /dashboard/live-queue
* **Description**: Retrieves orders in active processing status stages.
* **Request URL**: `GET /diagnostic-orders/orders?status=SAMPLE_COLLECTED`
* **Success Response (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "order_id": "d94cf56a-1234-5678-abcd-ef0123456789",
      "patient_name": "Sita Verma",
      "test_name": "Complete Blood Count (CBC)",
      "barcode": "BC-109283",
      "collector_name": "Amit Sharma",
      "status": "SAMPLE_COLLECTED",
      "received_at": "2026-07-16T12:30:00Z"
    }
  ]
}
```

---

### 1.5 GET /dashboard/revenue
* **Description**: Returns billing invoice aggregation statistics.
* **Request URL**: `GET /diagnostic-orders/billing/invoices`
* **Success Response (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "invoice_id": "8fa3729e-1111-2222-3333-444455556666",
      "invoice_number": "INV-2026-0001",
      "patient_name": "Rajesh Sharma",
      "total_amount": 1200.00,
      "tax_amount": 120.00,
      "discount_amount": 0.00,
      "net_amount": 1320.00,
      "status": "PAID",
      "created_at": "2026-07-16T12:15:00Z"
    }
  ]
}
```

---

### 1.6 GET /dashboard/alerts
* **Description**: Returns active low-stock inventory reagents or clinical compliance alert logs.
* **Request URL**: `GET /diagnostic-orders/audit?type=ALERTS`
* **Success Response (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "alert_id": "99f7e889-ff76-463a-ae62-386e97802b26",
      "alert_type": "LOW_STOCK",
      "item_name": "ESR Pipette Tubes",
      "current_quantity": 12,
      "minimum_threshold": 50,
      "triggered_at": "2026-07-16T11:00:00Z"
    }
  ]
}
```

---

## 2. Order Management Endpoints

### 2.1 GET /orders
* **Description**: List and filter all patient registration orders.
* **Request URL**: `GET /diagnostic-orders/orders`
* **Query Parameters**:
  * `status` (string, Optional): filter status (`PENDING`, `COMPLETED`, etc.)
  * `search` (string, Optional): patient name or barcode search
* **Success Response (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "order_id": "6a9e1088-7576-463a-ae62-386e97802b26",
      "patient_name": "Rajesh Sharma",
      "patient_mrn": "UHID-9821",
      "patient_age": 42,
      "patient_gender": "M",
      "barcode": "BC-881234",
      "current_status": "COMPLETED",
      "test_name": "ESR",
      "collector_name": "ARAV SHARMA",
      "created_at": "2026-07-16T12:00:00Z"
    }
  ]
}
```

---

### 2.2 GET /orders/{id}
* **Description**: Retrieve detailed parameters and results for a specific order.
* **Request URL**: `GET /diagnostic-orders/orders/{id}`
* **Path Parameters**:
  * `id` (string, Mandatory): UUID of order
* **Success Response (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "order_id": "6a9e1088-7576-463a-ae62-386e97802b26",
    "patient_id": "99f7e889-ff76-463a-ae62-386e97802b26",
    "patient_name": "Rajesh Sharma",
    "patient_mrn": "UHID-9821",
    "status": "COMPLETED",
    "priority": "ROUTINE",
    "lab_items": [
      {
        "item_id": "8cf7e889-ff76-463a-ae62-386e97802b26",
        "test_name": "ESR",
        "barcode": "BC-881234",
        "status": "COMPLETED",
        "result_value": "12",
        "reference_range": "0 - 15 mm/hr"
      }
    ],
    "status_history": [
      {
        "status": "PENDING",
        "updated_at": "2026-07-16T12:00:00Z"
      },
      {
        "status": "COMPLETED",
        "updated_at": "2026-07-16T12:45:00Z"
      }
    ]
  }
}
```

---

### 2.3 GET /orders/{id}/timeline
* **Description**: Retrieves timeline lifecycle milestones of the order.
* **Request URL**: `GET /diagnostic-orders/orders/{id}` (Extract `status_history` property)
* **Success Response (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "status": "PENDING",
      "description": "Order registered by Front Desk",
      "updated_at": "2026-07-16T12:00:00Z"
    },
    {
      "status": "SAMPLE_COLLECTED",
      "description": "Sample collected by Arav Sharma",
      "updated_at": "2026-07-16T12:15:00Z"
    },
    {
      "status": "COMPLETED",
      "description": "Results validated and released by Pathologist",
      "updated_at": "2026-07-16T12:45:00Z"
    }
  ]
}
```

---

### 2.4 GET /orders/{id}/status
* **Description**: Returns quick status header checks.
* **Request URL**: `GET /diagnostic-orders/orders/{id}`
* **Success Response (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "order_id": "6a9e1088-7576-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-16T12:45:00Z"
  }
}
```

---

## 3. Patient Info & Registration Histories

### 3.1 GET /patients
* **Description**: Retrieves patient lists matching query parameters.
* **Request URL**: `GET /diagnostic-orders/orders` (Filter by patient search parameter)
* **Success Response (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "patient_id": "99f7e889-ff76-463a-ae62-386e97802b26",
      "patient_name": "Rajesh Sharma",
      "patient_mrn": "UHID-9821",
      "phone": "+919876543210",
      "gender": "M"
    }
  ]
}
```

---

### 3.2 GET /patients/{id}
* **Description**: Returns detailed patient demographics profiles.
* **Request URL**: `GET /diagnostic-orders/orders/{id}`
* **Success Response (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "patient_id": "99f7e889-ff76-463a-ae62-386e97802b26",
    "patient_name": "Rajesh Sharma",
    "patient_mrn": "UHID-9821",
    "phone": "+919876543210",
    "gender": "M",
    "dob": "1984-05-15"
  }
}
```

---

### 3.3 GET /patients/{id}/history
* **Description**: Retrieves historical records of all lab tests requested for a patient.
* **Request URL**: `GET /diagnostic-orders/lookup?type=patients`
* **Success Response (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "order_id": "6a9e1088-7576-463a-ae62-386e97802b26",
      "test_name": "ESR",
      "result_value": "12",
      "status": "COMPLETED",
      "created_at": "2026-07-16T12:00:00Z"
    }
  ]
}
```

---

## 4. Queues & Worklists

### 4.1 GET /sample-collection
* **Description**: Retrieve active queue for sample collection container labeling.
* **Request URL**: `GET /diagnostic-orders/orders?status=PENDING`
* **Success Response (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "order_id": "6a9e1088-7576-463a-ae62-386e97802b26",
      "patient_name": "Rajesh Sharma",
      "test_name": "ESR",
      "status": "PENDING",
      "priority": "ROUTINE",
      "created_at": "2026-07-16T12:00:00Z"
    }
  ]
}
```

---

### 4.2 GET /processing
* **Description**: Retrieve queue of samples currently processing inside analyzer machines.
* **Request URL**: `GET /diagnostic-orders/orders?status=IN_PROGRESS`
* **Success Response (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "order_id": "d94cf56a-1234-5678-abcd-ef0123456789",
      "patient_name": "Sita Verma",
      "test_name": "CBC",
      "barcode": "BC-109283",
      "status": "IN_PROGRESS",
      "updated_at": "2026-07-16T12:45:00Z"
    }
  ]
}
```

---

## 5. Reports & Findings

### 5.1 GET /reports
* **Description**: Lists final validated diagnostic report records.
* **Request URL**: `GET /diagnostic-orders/orders?status=COMPLETED`
* **Success Response (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "order_id": "6a9e1088-7576-463a-ae62-386e97802b26",
      "patient_name": "Rajesh Sharma",
      "patient_mrn": "UHID-9821",
      "test_name": "ESR",
      "validated_by": "Dr. Priya Sharma",
      "validated_at": "2026-07-16T12:45:00Z"
    }
  ]
}
```

---

### 5.2 GET /reports/{id}
* **Description**: Fetch validated diagnostic findings structure details.
* **Request URL**: `GET /diagnostic-orders/orders/{id}`
* **Success Response (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "order_id": "6a9e1088-7576-463a-ae62-386e97802b26",
    "patient_name": "Rajesh Sharma",
    "report_details": {
      "test_name": "ESR",
      "result_value": "12",
      "unit": "mm/hr",
      "reference_range": "0 - 15",
      "findings_summary": "Findings within normal physiological ranges."
    }
  }
}
```

---

## 6. Catalog, Configuration, Settings & Access Lookups

### 6.1 GET /inventory
* **Description**: Lists current stock quantities and inventory levels of reagents/consumables.
* **Request URL**: `GET /diagnostic-orders/inventory`
* **Success Response (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "item_id": "e30920a4-1111-2222-3333-444455556666",
      "item_code": "REG-CBC-01",
      "item_name": "CBC Reagent Bottle A",
      "category": "Reagents",
      "current_stock": 140,
      "minimum_stock": 50,
      "status": "IN_STOCK"
    }
  ]
}
```

---

### 6.2 GET /vendors
* **Description**: List registered procurement vendor profile accounts.
* **Request URL**: `GET /diagnostic-orders/vendors`
* **Success Response (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "vendor_id": "d0291e84-1111-2222-3333-444455556666",
      "vendor_code": "VND-AROVITA",
      "vendor_name": "Arovita Reagents Ltd",
      "email": "sales@arovita.com",
      "phone": "+9111223344",
      "status": "ACTIVE"
    }
  ]
}
```

---

### 6.3 GET /tests
* **Description**: List all tests directory in dropdown catalog.
* **Request URL**: `GET /diagnostic-orders/lookup?type=tests`
* **Success Response (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "test_id": "16a43dca-eb71-4b85-ab0a-3b984816cadf",
      "test_name": "Complete Blood Count",
      "test_code": "CBC",
      "category": "Hematology",
      "price": 350.00
    }
  ]
}
```

---

### 6.4 GET /packages
* **Description**: List diagnostic packages directory catalog.
* **Request URL**: `GET /diagnostic-orders/lookup?type=packages`
* **Success Response (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "package_id": "p09283fa-1111-2222-3333-444455556666",
      "package_name": "Executive Health Package",
      "tests_included": ["CBC", "FBS", "Lipid Profile"],
      "price": 1500.00
    }
  ]
}
```

---

### 6.5 GET /staff
* **Description**: List active lab staff members and scheduling rosters.
* **Request URL**: `GET /diagnostic-orders/staff`
* **Success Response (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "staff_id": "s8fa2731-1111-2222-3333-444455556666",
      "full_name": "Amit Sharma",
      "role": "Lab Technician",
      "current_shift": "Day (08:00 - 16:00)",
      "status": "ON_DUTY"
    }
  ]
}
```

---

### 6.6 GET /roles
* **Description**: List custom security roles and privilege levels.
* **Request URL**: `GET /diagnostic-orders/roles`
* **Success Response (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "role_id": "LAB-001",
      "role_name": "LAB_ADMIN",
      "description": "Lab department administration",
      "permissions_count": 25
    }
  ]
}
```

---

### 6.7 GET /machines
* **Description**: List analyzer machine interface configurations.
* **Request URL**: `GET /diagnostic-orders/lookup?type=machines`
* **Success Response (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "machine_id": "m9a8372e-1111-2222-3333-444455556666",
      "machine_name": "Sysmex XN-1000",
      "model": "Hematology Analyzer",
      "status": "ONLINE"
    }
  ]
}
```

---

### 6.8 GET /settings
* **Description**: Read laboratory settings configuration profile (barcode constraints, print dimensions, templates).
* **Request URL**: `GET /diagnostic-orders/config`
* **Success Response (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "barcode_type": "CODE128",
    "auto_print_label": true,
    "default_printer_ip": "192.168.1.120",
    "reagent_alert_threshold_percent": 15
  }
}
```
