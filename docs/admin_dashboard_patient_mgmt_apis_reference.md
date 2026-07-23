# Admin Dashboard API Reference Documentation

This document provides a comprehensive API reference for all Admin Dashboard endpoints. These endpoints aggregate multi-module metric data to populate the KPI cards, tables, and graphs in the Admin Dashboard.

---

## 1. Global Authentication, Headers & Permissions

* **Authorization**: All dashboard endpoints require a valid Cognito JWT Access Token passed in the `Authorization` header:
  * `Authorization: Bearer <jwt_access_token>`
* **RBAC Permission ID**: `SYS-001` (System Admin access or Super Admin bypass) is required to view these dashboards.
* **Tenant Scoping**: All queries are tenant-scoped automatically by resolving the caller's active `branch_id` from their session in the DB.

### 1.1 Authentication Workflow (Obtaining Token)

To obtain the `access_token` for authorization, perform a login request using the following schema.

* **Endpoint**: `POST /auth/login`
* **Request Headers**: `Content-Type: application/json`
* **Sample Request Body**:
```json
{
  "username": "SYS-ADMIN01",
  "password": "ArovitaAdmin2026!"
}
```

* **Sample Response Body**:
```json
{
  "success": true,
  "code": 200,
  "message": "Login successful",
  "data": {
    "tokens": {
      "access_token": "eyJraWQiOiJ0dW...",
      "refresh_token": "eyJjdHkiOiJKV...",
      "id_token": "eyJraWQiOiJya...",
      "token_type": "Bearer",
      "expires_in": 3600
    },
    "user": {
      "user_id": "8b7f50c4-fa3a-4aef-ab27-f42b7fb0e54f",
      "employee_id": "SYS-ADMIN01",
      "full_name": "System Administrator",
      "email": "sysadmin@sai-ljb.com",
      "phone": "+917892707563",
      "role": "Senior Auditor A564",
      "role_category": "LAB",
      "tenant_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
      "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
      "branch_name": "Sai LJB - Branch 1",
      "department_id": null,
      "department_name": null,
      "specialization_id": null,
      "specialization_name": null,
      "registration_number": null,
      "qualification": null,
      "experience_years": null,
      "profile_photo": null,
      "is_active": true,
      "login_enabled": true,
      "cognito_provisioned": false,
      "gender": null,
      "date_of_birth": null,
      "last_login_at": "2026-07-20 06:37:43.924059+00:00",
      "joining_date": "2026-06-10",
      "created_at": "2026-06-10 06:10:49.057693+00:00"
    },
    "permissions": [],
    "session_id": "36dd2421-22b8-45c4-bca2-dc48994a1ab1"
  }
}
```

---

## 2. Common Query Filters (`_get_dashboard_filters`)

Most of the time-range/date-based endpoints share a helper query parameter parser that accepts:

| Parameter | Type | Default | Description |
| :--- | :---: | :---: | :--- |
| `fromDate` / `from_date` | String | *30 days ago* | Start date in `YYYY-MM-DD` format. |
| `toDate` / `to_date` | String | *Today* | End date in `YYYY-MM-DD` format. |
| `timeRange` / `time_range` | String | *None* | Shortcuts: `today`, `this-week`, `week`, `this-month`, `month`, `last-30-days`, `last-90-days`. Overrides custom date filters. |
| `hospitalId` / `hospital_id` | String (UUID) | *None* | Scopes the overview to an entire hospital organization. Only available to Super Admins. |
| `branchId` / `branch_id` | String (UUID) | *Caller's Tenant* | Scopes the overview to a specific branch. Defaults to the caller's branch. |
| `departmentId` / `department_id` | String (UUID) | *None* | Scopes calculations to a specific department. |

---

## 3. Endpoints Detail

### 0. Main Dashboard Aggregator (Unified Screen Loader)
* **Endpoint**: `GET /admin/dashboard/main`
* **Query Parameters**: Shared Common Filters (Section 2)
* **Description**: Returns all metric components required to populate the main admin dashboard screen design in a single HTTP request (aggregating overview cards, hospital performance trends, resource utilization, recent reports, stats/bar-charts, department financials, and report templates).
* **Sample Response**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "overview": {
      "patients": {
        "value": 2003,
        "change_pct": 3414.0
      },
      "emergency_cases": {
        "value": 6,
        "change_pct": 100.0
      },
      "bed_occupancy": {
        "value": 9.8,
        "change_pct": 0.0
      },
      "procedures": {
        "value": 25,
        "change_pct": 50.0
      },
      "revenue": {
        "value": 145000.00,
        "change_pct": 12.4
      },
      "pending_payments": {
        "value": 45000.00,
        "change_pct": 4.1
      },
      "conversion_rate": {
        "value": 64.0,
        "change_pct": -2.3
      }
    },
    "hospital_performance": {
      "trends": [
        {
          "date": "2026-07-23",
          "admissions": 10,
          "visits": 55,
          "revenue": 14500.0
        }
      ]
    },
    "resource_utilization": {
      "active_ventilators": 4,
      "occupied_beds": 10,
      "staff_on_duty": 15,
      "icu_occupancy_pct": 45.5
    },
    "recent_reports": {
      "items": [
        {
          "id": "rep-001",
          "report_name": "Finance Summary",
          "department": "Finance",
          "generated_by": "System Admin",
          "status": "Ready"
        }
      ],
      "meta": {
        "total": 1,
        "page": 1,
        "page_size": 10,
        "total_pages": 1
      }
    },
    "statistics": {
      "hospital_occupancy": [
        {
          "label": "General Ward",
          "value": 75.0
        }
      ],
      "expense_breakdown": [
        {
          "label": "Pharmacy Procurement",
          "value": 120000.0
        }
      ]
    },
    "department_performance": {
      "departments": [
        {
          "department_name": "Pharmacy",
          "revenue": 15000.0,
          "expenses": 12000.0,
          "net_profit": 3000.0,
          "margin_pct": 20.0
        }
      ]
    },
    "report_library": {
      "templates": [
        {
          "report_type": "FINANCIAL",
          "name": "Monthly Revenue Analysis",
          "category": "Finance",
          "status": "Active"
        }
      ],
      "total": 1
    }
  }
}
```

---

### 1. Dashboard Overview
* **Endpoint**: `GET /admin/dashboard/overview`
* **Query Parameters**: Shared Common Filters (Section 2)
* **Sample Response**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "patients": {
      "value": 156,
      "change_pct": 12.5
    },
    "emergency_cases": {
      "value": 18,
      "change_pct": -5.2
    },
    "bed_occupancy": {
      "value": 45,
      "change_pct": 2.1
    },
    "procedures": {
      "value": 12,
      "change_pct": 20.0
    },
    "revenue": {
      "value": 750000.0,
      "change_pct": 8.4
    },
    "pending_payments": {
      "value": 120000.0,
      "change_pct": -1.5
    },
    "conversion_rate": {
      "value": 65.0,
      "change_pct": 4.2
    }
  }
}
```

---

### 2. Patient Management
* **Endpoint**: `GET /admin/dashboard/patients`
* **Query Parameters**: Shared Common Filters (Section 2) + the following:
  * `page` (Integer, default: `1`): The page number.
  * `page_size` (Integer, default: `10`): The records per page.
  * `search` (String, optional): Filter by patient name, UHID, or phone.
* **Sample Response**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "patients": [
      {
        "patient_id": "c1a2f3a4-b5b6-789a-0123-456789abcdef",
        "uhid": "PAT-2026-0084",
        "name": "Aarav Mehta",
        "age": 29,
        "gender": "MALE",
        "blood_group": "A+",
        "registration_date": "2026-07-22T10:45:00.000Z",
        "status": "Active"
      }
    ],
    "gender_stats": {
      "male": 420,
      "female": 380,
      "other": 5
    },
    "age_stats": {
      "under_18": 120,
      "age_18_35": 310,
      "age_36_50": 205,
      "age_51_65": 115,
      "above_65": 55
    },
    "department_stats": [
      {
        "department_name": "General Medicine",
        "count": 340
      }
    ],
    "total_registered": 805,
    "meta": {
      "total": 805,
      "page": 1,
      "page_size": 10,
      "total_pages": 81
    }
  }
}
```

---

### 3. OPD Management
* **Endpoint**: `GET /admin/dashboard/opd`
* **Query Parameters**: Shared Common Filters (Section 2)
* **Sample Response**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "department_performance": [
      {
        "department_name": "Pediatrics",
        "total_visits": 45,
        "completed": 40,
        "pending": 5,
        "avg_wait_minutes": 15.0
      }
    ],
    "doctor_performance": [
      {
        "doctor_name": "Dr. Priya Sharma",
        "department_name": "Pediatrics",
        "visits_count": 25
      }
    ],
    "visit_trends": [
      {
        "date": "2026-07-23",
        "visits": 45
      }
    ],
    "total_opd_visits": 45
  }
}
```

---

### 4. IPD Management
* **Endpoint**: `GET /admin/dashboard/ipd`
* **Query Parameters**: Shared Common Filters (Section 2)
* **Sample Response**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "ward_overview": [
      {
        "ward_name": "General Ward A",
        "ward_type": "GENERAL",
        "capacity": 20,
        "occupied": 15,
        "available": 5,
        "utilization_pct": 75.0
      }
    ],
    "length_of_stay": [
      {
        "department_name": "General Medicine",
        "avg_days": 4.5
      }
    ],
    "total_admissions": 15,
    "bed_occupancy_pct": 75.0
  }
}
```

---

### 5. Emergency Management
* **Endpoint**: `GET /admin/dashboard/emergency`
* **Query Parameters**: Shared Common Filters (Section 2)
* **Sample Response**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "cases": [
      {
        "case_id": "e210-4f8e-8e9a-0123456789ab",
        "patient_name": "Rohan Gupta",
        "triage_level": "RED",
        "arrival_time": "2026-07-23T11:15:30.000Z",
        "department_name": "Cardiology",
        "status": "ACTIVE"
      }
    ],
    "department_split": [
      {
        "department_name": "Cardiology",
        "count": 1
      }
    ],
    "avg_response_minutes": 8.5,
    "avg_wait_minutes": 12.0,
    "total_critical_cases": 1
  }
}
```

---

### 6. OT Management
* **Endpoint**: `GET /admin/dashboard/ot`
* **Query Parameters**: Shared Common Filters (Section 2)
* **Sample Response**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "today_procedures": [
      {
        "procedure_id": "ba679aec-1dc2-455e-ab58-58147f396d74",
        "patient_name": "Suresh Patel",
        "procedure_name": "Knee Arthroplasty",
        "theater_name": "OT Complex 1",
        "surgeon_name": "Dr. Priya Sharma",
        "scheduled_start": "2026-07-23T09:00:00.000Z",
        "status": "COMPLETED"
      }
    ],
    "completion_report": {
      "scheduled": 1,
      "completed": 1,
      "in_progress": 0,
      "delayed": 0
    },
    "theater_utilization_pct": 82.5,
    "delay_analysis": [
      {
        "name": "Anesthesia Delay",
        "value": 1
      }
    ]
  }
}
```

---

### 7. Laboratory Management
* **Endpoint**: `GET /admin/dashboard/laboratory`
* **Query Parameters**: Shared Common Filters (Section 2)
* **Sample Response**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "tests": [
      {
        "test_id": "ced4ccbc-6b2b-43ef-8a4e-1a857619f156",
        "test_name": "Complete Blood Count (CBC)",
        "patient_name": "Vikram Sen",
        "priority": "ROUTINE",
        "status": "COMPLETED",
        "ordered_at": "2026-07-23T10:10:00.000Z"
      }
    ],
    "sample_status": {
      "PENDING": 2,
      "PROCESSING": 1,
      "COMPLETED": 5
    },
    "machine_status": [
      {
        "machine_name": "Sysmex XN-1000 Hematology",
        "status": "CONNECTED",
        "last_active": "2026-07-23T12:00:00.000Z"
      }
    ],
    "avg_tat_minutes": 145.0
  }
}
```

---

### 8. Radiology Management
* **Endpoint**: `GET /admin/dashboard/radiology`
* **Query Parameters**: Shared Common Filters (Section 2)
* **Sample Response**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "imaging_schedule": [
      {
        "item_id": "532c8c78-74d0-44a3-a73c-dffd155b7659",
        "modality": "MRI",
        "patient_name": "Amit Shah",
        "scheduled_time": "2026-07-23T08:30:00.000Z",
        "status": "SCHEDULED"
      }
    ],
    "reporting_queue": [
      {
        "item_id": "rad-103",
        "modality": "XRAY",
        "patient_name": "Rahul Verma",
        "study_date": "2026-07-23T09:15:00.000Z",
        "status": "COMPLETED",
        "is_critical": true
      }
    ],
    "radiologist_workload": [
      {
        "radiologist_name": "Dr. Sanjeev Kumar",
        "completed_count": 12,
        "pending_count": 3
      }
    ],
    "pacs_health": [
      {
        "service_name": "DICOM Router",
        "status": "Healthy",
        "latency_ms": 15
      }
    ]
  }
}
```

---

### 9. Pharmacy Management
* **Endpoint**: `GET /admin/dashboard/pharmacy`
* **Query Parameters**: Shared Common Filters (Section 2)
* **Sample Response**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "daily_sales": [
      {
        "date": "2026-07-23",
        "revenue": "45000.00",
        "transaction_count": 25
      }
    ],
    "top_selling": [
      {
        "medicine_name": "Paracetamol 650mg",
        "category": "Analgesic",
        "quantity": 120,
        "revenue": "2400.00"
      }
    ],
    "category_sales": [
      {
        "category": "Analgesic",
        "revenue": "12400.00"
      }
    ],
    "stock_movement": [
      {
        "medicine_name": "Paracetamol 650mg",
        "opening_stock": 500,
        "received": 200,
        "dispensed": 120,
        "closing_stock": 580
      }
    ],
    "total_prescriptions_today": 15,
    "dispensing_queue_count": 3
  }
}
```

---

### 10. Inventory Management
* **Endpoint**: `GET /admin/dashboard/inventory`
* **Query Parameters**: None (Returns aggregate state of pharmacy inventory items).
* **Sample Response**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "warehouse_inventory": [
      {
        "item_name": "Atorvastatin 10mg",
        "category": "Cardiovascular",
        "warehouse_stock": 450,
        "unit_price": "12.50",
        "valuation": "5625.00"
      }
    ],
    "category_valuation": [
      {
        "category": "Cardiovascular",
        "total_value": "5625.00"
      }
    ],
    "total_inventory_value": "5625.00"
  }
}
```

---

### 11. Staff Management
* **Endpoint**: `GET /admin/dashboard/staff`
* **Query Parameters**: None (Returns current utilization and staffing numbers).
* **Sample Response**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "utilization": [
      {
        "role_name": "Doctor",
        "total_count": 8,
        "active_count": 6,
        "utilization_pct": 75.0
      }
    ],
    "department_staff": [
      {
        "department_name": "Emergency Medicine",
        "doctors": 3,
        "nurses": 8,
        "other_staff": 2
      }
    ],
    "total_staff": 13,
    "on_duty_count": 10
  }
}
```
