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
                "value": "2003",
                "change_pct": "3414.0"
            },
            "emergency_cases": {
                "value": "6",
                "change_pct": "100.0"
            },
            "bed_occupancy": {
                "value": "9.8",
                "change_pct": "0.0"
            },
            "procedures": {
                "value": "2",
                "change_pct": "-66.7"
            },
            "revenue": {
                "value": "197528.50",
                "change_pct": "19652.9"
            },
            "pending_payments": {
                "value": "1985690.00",
                "change_pct": "8925.9"
            },
            "conversion_rate": {
                "value": "0.9",
                "change_pct": "-94.6"
            }
        },
        "hospital_performance": {
            "operations_overview": [
                {
                    "label": "23 Jun 2026",
                    "opd": 3,
                    "ipd": 0,
                    "emergency": 0,
                    "ot": 0
                },
                {
                    "label": "25 Jun 2026",
                    "opd": 0,
                    "ipd": 11,
                    "emergency": 0,
                    "ot": 9
                },
                {
                    "label": "26 Jun 2026",
                    "opd": 3,
                    "ipd": 0,
                    "emergency": 0,
                    "ot": 0
                },
                {
                    "label": "27 Jun 2026",
                    "opd": 1,
                    "ipd": 0,
                    "emergency": 0,
                    "ot": 0
                },
                {
                    "label": "29 Jun 2026",
                    "opd": 5,
                    "ipd": 0,
                    "emergency": 0,
                    "ot": 0
                },
                {
                    "label": "30 Jun 2026",
                    "opd": 0,
                    "ipd": 0,
                    "emergency": 1,
                    "ot": 0
                },
                {
                    "label": "01 Jul 2026",
                    "opd": 4,
                    "ipd": 0,
                    "emergency": 1,
                    "ot": 0
                },
                {
                    "label": "02 Jul 2026",
                    "opd": 3,
                    "ipd": 0,
                    "emergency": 1,
                    "ot": 0
                },
                {
                    "label": "03 Jul 2026",
                    "opd": 16,
                    "ipd": 15,
                    "emergency": 1,
                    "ot": 14
                },
                {
                    "label": "04 Jul 2026",
                    "opd": 2,
                    "ipd": 4,
                    "emergency": 0,
                    "ot": 5
                },
                {
                    "label": "06 Jul 2026",
                    "opd": 6,
                    "ipd": 0,
                    "emergency": 0,
                    "ot": 0
                },
                {
                    "label": "08 Jul 2026",
                    "opd": 1,
                    "ipd": 1,
                    "emergency": 0,
                    "ot": 0
                },
                {
                    "label": "09 Jul 2026",
                    "opd": 1806,
                    "ipd": 3,
                    "emergency": 0,
                    "ot": 4
                },
                {
                    "label": "10 Jul 2026",
                    "opd": 1,
                    "ipd": 1,
                    "emergency": 0,
                    "ot": 1
                },
                {
                    "label": "13 Jul 2026",
                    "opd": 1,
                    "ipd": 0,
                    "emergency": 0,
                    "ot": 0
                },
                {
                    "label": "17 Jul 2026",
                    "opd": 1,
                    "ipd": 0,
                    "emergency": 0,
                    "ot": 0
                },
                {
                    "label": "18 Jul 2026",
                    "opd": 1,
                    "ipd": 2,
                    "emergency": 0,
                    "ot": 4
                },
                {
                    "label": "19 Jul 2026",
                    "opd": 0,
                    "ipd": 0,
                    "emergency": 1,
                    "ot": 1
                },
                {
                    "label": "20 Jul 2026",
                    "opd": 0,
                    "ipd": 1,
                    "emergency": 1,
                    "ot": 3
                },
                {
                    "label": "21 Jul 2026",
                    "opd": 3,
                    "ipd": 0,
                    "emergency": 0,
                    "ot": 0
                },
                {
                    "label": "22 Jul 2026",
                    "opd": 1,
                    "ipd": 0,
                    "emergency": 0,
                    "ot": 4
                },
                {
                    "label": "23 Jul 2026",
                    "opd": 2,
                    "ipd": 0,
                    "emergency": 0,
                    "ot": 0
                }
            ],
            "revenue_overview": [
                {
                    "label": "23 Jun 2026",
                    "revenue": "0.00",
                    "expenses": "0.0"
                },
                {
                    "label": "26 Jun 2026",
                    "revenue": "0.00",
                    "expenses": "0.0"
                },
                {
                    "label": "27 Jun 2026",
                    "revenue": "0.00",
                    "expenses": "0.0"
                },
                {
                    "label": "29 Jun 2026",
                    "revenue": "0.00",
                    "expenses": "0.0"
                },
                {
                    "label": "01 Jul 2026",
                    "revenue": "0.00",
                    "expenses": "0.0"
                },
                {
                    "label": "02 Jul 2026",
                    "revenue": "11200.00",
                    "expenses": "0.0"
                },
                {
                    "label": "03 Jul 2026",
                    "revenue": "158750.00",
                    "expenses": "0.0"
                },
                {
                    "label": "04 Jul 2026",
                    "revenue": "500.00",
                    "expenses": "0.0"
                },
                {
                    "label": "06 Jul 2026",
                    "revenue": "500.00",
                    "expenses": "0.0"
                },
                {
                    "label": "08 Jul 2026",
                    "revenue": "0.00",
                    "expenses": "0.0"
                },
                {
                    "label": "09 Jul 2026",
                    "revenue": "500.00",
                    "expenses": "0.0"
                },
                {
                    "label": "10 Jul 2026",
                    "revenue": "0.00",
                    "expenses": "0.0"
                },
                {
                    "label": "11 Jul 2026",
                    "revenue": "100.00",
                    "expenses": "0.0"
                },
                {
                    "label": "13 Jul 2026",
                    "revenue": "0.00",
                    "expenses": "0.0"
                },
                {
                    "label": "16 Jul 2026",
                    "revenue": "0.00",
                    "expenses": "0.0"
                },
                {
                    "label": "17 Jul 2026",
                    "revenue": "16056.00",
                    "expenses": "0.0"
                },
                {
                    "label": "18 Jul 2026",
                    "revenue": "0.00",
                    "expenses": "0.0"
                },
                {
                    "label": "20 Jul 2026",
                    "revenue": "5880.00",
                    "expenses": "0.0"
                },
                {
                    "label": "21 Jul 2026",
                    "revenue": "4042.50",
                    "expenses": "0.0"
                },
                {
                    "label": "22 Jul 2026",
                    "revenue": "0.00",
                    "expenses": "0.0"
                },
                {
                    "label": "23 Jul 2026",
                    "revenue": "0.00",
                    "expenses": "0.0"
                }
            ]
        },
        "resource_utilization": {
            "staff_utilization": "100.0",
            "bed_utilization": "9.8",
            "ward_utilization": "9.3",
            "ambulance_utilization": "0.0"
        },
        "recent_reports": {
            "items": [
                {
                    "id": "rep-818",
                    "report_name": "Surgery Audit Log",
                    "department": "Surgery",
                    "reporting_period": "Jun 22 - Jul 22, 2026",
                    "generated_by": "Dr. Aarav Patel",
                    "generated_on": "2026-07-22 15:57:56.305105+00:00",
                    "status": "Ready"
                },
                {
                    "id": "rep-032",
                    "report_name": "Orthopaedics Operational Analysis",
                    "department": "Orthopaedics",
                    "reporting_period": "Jun 22 - Jul 22, 2026",
                    "generated_by": "Dr. Dev Verma",
                    "generated_on": "2026-07-22 10:03:56.305168+00:00",
                    "status": "Ready"
                },
                {
                    "id": "rep-992",
                    "report_name": "Gynaecology Operational Analysis",
                    "department": "Gynaecology",
                    "reporting_period": "Jun 20 - Jul 20, 2026",
                    "generated_by": "Dr. Anil Pillai",
                    "generated_on": "2026-07-20 17:23:56.305187+00:00",
                    "status": "Ready"
                },
                {
                    "id": "rep-156",
                    "report_name": "Paediatrics Operational Analysis",
                    "department": "Paediatrics",
                    "reporting_period": "Jun 19 - Jul 19, 2026",
                    "generated_by": "Meena Krishnan",
                    "generated_on": "2026-07-19 13:19:56.305202+00:00",
                    "status": "Ready"
                },
                {
                    "id": "rep-517",
                    "report_name": "Ophthalmology Summary Report",
                    "department": "Ophthalmology",
                    "reporting_period": "Jun 19 - Jul 19, 2026",
                    "generated_by": "Dr. Vikram Shah",
                    "generated_on": "2026-07-19 04:58:56.305216+00:00",
                    "status": "Ready"
                },
                {
                    "id": "rep-277",
                    "report_name": "Psychiatry Summary Report",
                    "department": "Psychiatry",
                    "reporting_period": "Jun 18 - Jul 18, 2026",
                    "generated_by": "Dr. Swati Mehta",
                    "generated_on": "2026-07-18 04:58:56.305230+00:00",
                    "status": "Ready"
                },
                {
                    "id": "rep-480",
                    "report_name": "Anaesthesiology Operational Analysis",
                    "department": "Anaesthesiology",
                    "reporting_period": "Jun 17 - Jul 17, 2026",
                    "generated_by": "Dr. Kiran Sharma",
                    "generated_on": "2026-07-17 10:15:56.305243+00:00",
                    "status": "Ready"
                },
                {
                    "id": "rep-829",
                    "report_name": "Proctology Summary Report",
                    "department": "Proctology",
                    "reporting_period": "Jun 16 - Jul 16, 2026",
                    "generated_by": "Dr. Anil Pandey",
                    "generated_on": "2026-07-16 04:46:56.305255+00:00",
                    "status": "Ready"
                },
                {
                    "id": "rep-556",
                    "report_name": "ICU / Critical Care Operational Analysis",
                    "department": "ICU / Critical Care",
                    "reporting_period": "Jun 15 - Jul 15, 2026",
                    "generated_by": "Meena Krishnan",
                    "generated_on": "2026-07-15 05:59:56.305268+00:00",
                    "status": "Ready"
                },
                {
                    "id": "rep-287",
                    "report_name": "Operation Theatre Performance Trends",
                    "department": "Operation Theatre",
                    "reporting_period": "Jun 13 - Jul 13, 2026",
                    "generated_by": "Dr. Ananya Reddy",
                    "generated_on": "2026-07-13 10:28:56.305280+00:00",
                    "status": "Ready"
                }
            ],
            "meta": {
                "total": 50,
                "page": 1,
                "page_size": 10,
                "total_pages": 5
            }
        },
        "statistics": {
            "hospital_occupancy": [
                {
                    "name": "ISOLATION",
                    "value": "7.1"
                },
                {
                    "name": "GENERAL",
                    "value": "27.4"
                },
                {
                    "name": "OT",
                    "value": "11.8"
                },
                {
                    "name": "HDU",
                    "value": "0.0"
                },
                {
                    "name": "ICU",
                    "value": "5.0"
                },
                {
                    "name": "EMERGENCY",
                    "value": "3.7"
                }
            ],
            "expense_breakdown": [
                {
                    "name": "Salaries & Wages",
                    "value": "0.0"
                },
                {
                    "name": "Maintenance & Ops",
                    "value": "0.0"
                },
                {
                    "name": "Medicines & Pharmacy",
                    "value": "0.0"
                },
                {
                    "name": "Utilities & Admin",
                    "value": "0.0"
                }
            ]
        },
        "department_performance": {
            "departments": [
                {
                    "department_name": "Pharmacy",
                    "revenue": "0.0",
                    "expenses": "0.0",
                    "net_profit": "0.0",
                    "margin_pct": "0.0"
                },
                {
                    "department_name": "Laboratory",
                    "revenue": "0.0",
                    "expenses": "0.0",
                    "net_profit": "0.0",
                    "margin_pct": "0.0"
                },
                {
                    "department_name": "OPD",
                    "revenue": "112850.0",
                    "expenses": "0.0",
                    "net_profit": "112850.0",
                    "margin_pct": "100.0"
                },
                {
                    "department_name": "Emergency",
                    "revenue": "0.0",
                    "expenses": "0.0",
                    "net_profit": "0.0",
                    "margin_pct": "0.0"
                },
                {
                    "department_name": "ICU",
                    "revenue": "58700.0",
                    "expenses": "0.0",
                    "net_profit": "58700.0",
                    "margin_pct": "100.0"
                },
                {
                    "department_name": "OT / Surgery",
                    "revenue": "0.0",
                    "expenses": "0.0",
                    "net_profit": "0.0",
                    "margin_pct": "0.0"
                }
            ]
        },
        "report_library": {
            "templates": [
                {
                    "report_type": "REVENUE_SUMMARY",
                    "name": "Revenue Summary Report",
                    "description": "Aggregated daily/monthly billing revenues and transaction totals grouped by payment mode.",
                    "category": "FINANCE",
                    "supported_formats": [
                        "PDF",
                        "CSV",
                        "XLSX"
                    ],
                    "status": "Active"
                },
                {
                    "report_type": "INVENTORY_VALUATION",
                    "name": "Inventory Master & Valuation Report",
                    "description": "Stock valuation breakdown, low stock counts, critical alerts, and near-expiry metrics.",
                    "category": "INVENTORY",
                    "supported_formats": [
                        "PDF",
                        "CSV",
                        "XLSX"
                    ],
                    "status": "Active"
                },
                {
                    "report_type": "CLINICAL_CENSUS",
                    "name": "Hospital Clinical Census Report",
                    "description": "Patient inflow counts, bed occupancy distributions, and emergency room statistics.",
                    "category": "CLINICAL",
                    "supported_formats": [
                        "PDF",
                        "CSV",
                        "XLSX"
                    ],
                    "status": "Active"
                },
                {
                    "report_type": "STAFF_COMPLIANCE",
                    "name": "Workforce & Compliance Overview Report",
                    "description": "Active staff roster ratios, leave backlogs, and regulatory document upload status.",
                    "category": "WORKFORCE",
                    "supported_formats": [
                        "PDF",
                        "CSV",
                        "XLSX"
                    ],
                    "status": "Active"
                }
            ],
            "total": 4
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
  * `status` (String, optional): Filter by patient status (`Active` or `Inactive`).
  * `branchId` / `branch_id` (String (UUID), optional): Scope list to a specific facility/branch.
  * `departmentId` / `department_id` (String (UUID), optional): Scope list to patients who visited a specific department.
* **Sample Response**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "patients": [
      {
        "patient_id": "fbbd6afd-cb81-47a2-a782-9d3f42a2dfd9",
        "uhid": "PAT-2026-4324",
        "name": "ghshd sdf",
        "age": 24,
        "gender": "FEMALE",
        "blood_group": "B+",
        "phone": "+91 7906927827",
        "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
        "registration_date": "2026-07-23 16:23:28.438140+05:30",
        "status": "Active",
        "patient_type": "OPD",
        "ward": "UNKNOWN",
        "bed": "UNKNOWN",
        "admitted_on": "UNKNOWN",
        "attending_doctor": "UNKNOWN",
        "diagnosis": "UNKNOWN",
        "last_visit_date": "UNKNOWN"
      },
      {
        "patient_id": "536a6d42-c438-45ca-9ad2-204ea15af642",
        "uhid": "PAT-2026-4323",
        "name": "Test5 sd",
        "age": 22,
        "gender": "MALE",
        "blood_group": "A+",
        "phone": "+91 9410452083",
        "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
        "registration_date": "2026-07-23 16:05:28.466594+05:30",
        "status": "Active",
        "patient_type": "OPD",
        "ward": "UNKNOWN",
        "bed": "UNKNOWN",
        "admitted_on": "UNKNOWN",
        "attending_doctor": "UNKNOWN",
        "diagnosis": "UNKNOWN",
        "last_visit_date": "2026-07-23 00:00:00"
      }
    ],
    "gender_stats": {
      "male": 64,
      "female": 1919,
      "other": 1
    },
    "age_stats": {
      "under_18": 5,
      "age_18_35": 1953,
      "age_36_50": 25,
      "age_51_65": 0,
      "above_65": 0
    },
    "department_stats": [
      {
        "department_name": "Surgery",
        "count": 85
      },
      {
        "department_name": "Orthomology",
        "count": 79
      },
      {
        "department_name": "Priority-Pediatrics",
        "count": 68
      }
    ],
    "total_registered": 1984,
    "critical_patients": 4,
    "opd_patients": 1831,
    "ipd_patients": 24,
    "meta": {
      "total": 1984,
      "page": 1,
      "page_size": 2,
      "total_pages": 992
    }
  }
}
```

---

### 3. OPD Management
* **Endpoint**: `GET /admin/dashboard/opd`
* **Query Parameters**: Shared Common Filters (Section 2) + the following:
  * `department_id` (String (UUID), optional): Scope response to a single department by ID.
  * `department_name` (String, optional): Scope response to a single department by name (case-insensitive lookup).
* **Sample Response**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "department_performance": [
      {
        "department_name": "General Medicine",
        "department_id": "61776859-fb29-4e8e-8e01-7013d804eca8",
        "operating_hours": "08:00 AM - 06:00 PM",
        "total_visits": 47,
        "completed": 45,
        "pending": 2,
        "avg_wait_minutes": 22.0
      }
    ],
    "doctor_performance": [
      {
        "doctor_id": "doc-101",
        "doctor_name": "Dr. Sarah Johnson",
        "department_name": "General Medicine",
        "visits_count": 21,
        "designation": "Head of Department / Senior MD",
        "is_active": true
      }
    ],
    "visit_trends": [
      {
        "date": "2026-07-23",
        "visits": 3
      }
    ],
    "total_opd_visits": 1849
  }
}
```

---

### 3.1. Override Department Hours
* **Endpoint**: `PUT /admin/dashboard/opd/department`
* **Request Headers**:
  * `Authorization: Bearer <jwt_access_token>`
  * `Content-Type: application/json`
* **Request Body**:
```json
{
  "department_id": "61776859-fb29-4e8e-8e01-7013d804eca8",
  "operating_hours": "08:00 AM - 06:00 PM"
}
```
* **Sample Response**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "message": "Department operating hours updated successfully."
  }
}
```

---

### 4. IPD Management
* **Endpoint**: `GET /admin/dashboard/ipd`
* **Query Parameters**: Shared Common Filters (Section 2) + the following:
  * `ward_type` (String, optional): Filter wards by type (e.g., `ICU`, `GENERAL`, `HDU`, `EMERGENCY`).
  * `load_status` (String, optional): Filter wards by current patient occupancy status (e.g., `High Load`, `Normal`, `Delayed`).
* **Sample Response**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "ward_overview": [
      {
        "ward_id": "995a1615-cb03-4605-8ec8-bd60bfd48e66",
        "ward_name": "General Ward - A",
        "ward_type": "GENERAL",
        "capacity": 80,
        "occupied": 66,
        "available": 14,
        "utilization_pct": 82.5,
        "floor": "2nd Floor",
        "building": "Block A",
        "head_nurse": "Dr. Sarah Mitchell",
        "contact": "+1 (555) 234-5678",
        "load_status": "High Load"
      }
    ],
    "length_of_stay": [
      {
        "department_name": "General Medicine",
        "avg_days": 17.2
      }
    ],
    "total_admissions": 27,
    "bed_occupancy_pct": 9.8
  }
}
```

---

### 4.1. Override Ward Details
* **Endpoint**: `PUT /admin/dashboard/ipd/ward`
* **Request Headers**:
  * `Authorization: Bearer <jwt_access_token>`
  * `Content-Type: application/json`
* **Request Body**:
```json
{
  "ward_id": "995a1615-cb03-4605-8ec8-bd60bfd48e66",
  "ward_type": "GENERAL",
  "floor": "2nd Floor",
  "building": "Block A",
  "head_nurse": "Dr. Sarah Mitchell",
  "contact": "+1 (555) 234-5678",
  "load_status": "High Load"
}
```
* **Sample Response**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "message": "Ward details updated successfully."
  }
}
```

---

### 5. Emergency Management
* **Endpoint**: `GET /admin/dashboard/emergency`
* **Query Parameters**: Shared Common Filters (Section 2) + the following:
  * `status` (String, optional): Filter cases by status (e.g., `Awaiting OT`, `In Treatment`, `ICU Transfer`, `Triage Done`).
  * `triage_level` (String, optional): Filter cases by triage priority (e.g., `RED`, `YELLOW`, `GREEN`, `ORANGE`).
  * `search` (String, optional): Case-insensitive search on patient name, phone, UHID, or MRN.
* **Sample Response**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "cases": [
      {
        "case_id": "58169a2e-d2bc-4d0d-8381-57559be735b8",
        "patient_name": "Rajesh Sharma",
        "triage_level": "RED",
        "arrival_time": "2026-07-23 10:34:00",
        "department_name": "Emergency",
        "status": "Awaiting OT",
        "uhid": "UHID-9821",
        "age": 42,
        "gender": "MALE",
        "blood_group": "B+",
        "phone": "+91 98765 43210",
        "arrival_mode": "Ambulance",
        "assigned_doctor": "Dr. Priya Mehta",
        "bed_number": "E-12",
        "chief_complaint": "Chest pain, shortness of breath",
        "vitals": "BP: 130/85 | HR: 112 | SpO2: 94%"
      }
    ],
    "department_split": [
      {
        "department_name": "Emergency",
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

### 5.1. Override Emergency Case Details
* **Endpoint**: `PUT /admin/dashboard/emergency/case`
* **Request Headers**:
  * `Authorization: Bearer <jwt_access_token>`
  * `Content-Type: application/json`
* **Request Body**:
```json
{
  "case_id": "58169a2e-d2bc-4d0d-8381-57559be735b8",
  "triage_level": "RED",
  "status": "Awaiting OT",
  "arrival_mode": "Ambulance",
  "assigned_doctor_id": "97d2a10d-1b8a-40b4-a37a-5daf4e35f630",
  "assigned_bed_id": "3bd15010-f3ec-4343-a241-96afb54ec6db",
  "chief_complaint": "Chest pain, shortness of breath",
  "vitals": "BP: 130/85 | HR: 112 | SpO2: 94%"
}
```
* **Sample Response**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "message": "Emergency case details updated successfully."
  }
}
```

---

### 6. OT Management (Procedures / OT)
* **Endpoint**: `GET /admin/dashboard/ot`
* **Query Parameters**: Shared Common Filters (Section 2) + the following:
  * `status` (String, optional): Filter procedures by status (e.g., `Delayed`, `In Progress`, `Scheduled`, `Completed`).
  * `surgeon_id` (String (UUID), optional): Filter procedures by lead surgeon.
  * `ot_number` (String, optional): Filter procedures by specific OT room name/number (e.g. `OT-1`).
  * `search` (String, optional): Search by patient name, surgeon name, or procedure name.
* **Sample Response**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "today_procedures": [
      {
        "procedure_id": "ba679aec-1dc2-455e-ab58-58147f396d74",
        "patient_name": "Ramesh D.",
        "procedure_name": "Coronary Bypass",
        "theater_name": "OT-1",
        "surgeon_name": "Dr. Kapoor",
        "surgeon_id": "97d2a10d-1b8a-40b4-a37a-5daf4e35f630",
        "scheduled_start": "2026-07-23 08:00:00",
        "status": "Delayed",
        "tracker_stages": [
          {
            "stage_name": "Theatre Preparation",
            "status": "COMPLETED",
            "time": "07:30 AM",
            "notes": "Theatre prepared, sterilised & surgical tray configured. Checked by Lead Nurse"
          },
          {
            "stage_name": "Anesthesia Induction",
            "status": "COMPLETED",
            "time": "07:50 AM",
            "notes": "Patient administered general anaesthesia. Vital stable and continuously monitored."
          },
          {
            "stage_name": "Surgery In Progress",
            "status": "IN_PROGRESS",
            "time": "08:15 AM",
            "notes": "Coronary artery bypass graft surgery underway. Estimated duration: 3.5 hours."
          },
          {
            "stage_name": "Post-Op Recovery (PACU)",
            "status": "PENDING",
            "time": "Est. 11:50 AM",
            "notes": "Patient will be shifted to recovery wing for intensive post-op monitoring."
          }
        ]
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

### 6.1. Override OT Session Details
* **Endpoint**: `PUT /admin/dashboard/ot/session`
* **Request Headers**:
  * `Authorization: Bearer <jwt_access_token>`
  * `Content-Type: application/json`
* **Request Body**:
```json
{
  "procedure_id": "ba679aec-1dc2-455e-ab58-58147f396d74",
  "status": "In Progress",
  "procedure_name": "Coronary Bypass Extra",
  "theater_name": "OT-Room-3",
  "tracker_stages": [
    {
      "stage_name": "Theatre Preparation",
      "status": "COMPLETED",
      "time": "08:00 AM",
      "notes": "Completed and sterile"
    }
  ]
}
```
* **Sample Response**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "message": "OT session details updated successfully."
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
