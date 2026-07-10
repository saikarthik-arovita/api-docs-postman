# HMS — Admin Service API Documentation

This document covers all administrative dashboard, analytics, configuration settings, alerting, staff role/permission tuning, leave workflow, capacity management, TPA insurance, pharmacy inventory, procurement, and compliance audit log endpoints for the HMS **Admin Service**.

---

## 1. Global Conventions

### Base URL
All requests are sent to the Admin Service API Gateway stage:
```
https://<api-id>.execute-api.ap-south-1.amazonaws.com/<stage>
```

### Authorization Header
All endpoints require a valid JWT Access Token passed in the `Authorization` header:
```http
Authorization: Bearer <access_token>
```

### Universal Response Envelope
All API responses follow the standard envelope format:

**Success Response (200 / 201):**
```json
{
  "success": true,
  "data": { ... },
  "message": "Optional descriptive success message"
}
```

**Error Response (4xx / 5xx):**
```json
{
  "success": false,
  "code": 409,
  "message": "Conflict: Leave request overlaps with an existing pending or approved leave"
}
```

---

## 2. Dashboards & Analytics

### 2.0 Command Center Aggregate Dashboard APIs

The following endpoints support consolidated aggregate metric retrieval for the Analytics & Reports Dashboard figma cards:

#### 2.0.1 GET /admin/dashboard/overview
Returns all top KPI cards (Patients, Emergency Cases, Bed Occupancy, Procedures, Revenue, Pending Payments, Conversion Rate) for the current period, along with percentage change compared to the previous period of equal duration.

* **Query Parameters:**
  * `hospitalId` / `hospital_id` (UUID, optional)
  * `branchId` / `branch_id` (UUID, optional)
  * `departmentId` / `department_id` (UUID, optional)
  * `fromDate` / `from_date` (date: `YYYY-MM-DD`, optional)
  * `toDate` / `to_date` (date: `YYYY-MM-DD`, optional)
  * `timeRange` / `time_range` (string, optional: `today`, `this-week`, `this-month`, `last-30-days`, `last-90-days`)
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "patients": { "value": 1248, "change_pct": 12.0 },
    "emergency_cases": { "value": 42, "change_pct": -3.0 },
    "bed_occupancy": { "value": 85.0, "change_pct": 5.0 },
    "procedures": { "value": 18, "change_pct": -12.0 },
    "revenue": { "value": 840000.00, "change_pct": 15.0 },
    "pending_payments": { "value": 120000.00, "change_pct": -5.0 },
    "conversion_rate": { "value": 15.0, "change_pct": 2.0 }
  }
}
```

#### 2.0.2 GET /admin/dashboard/hospital-performance
Returns Operations Overview trends (OPD, IPD, Emergency, OT) and Revenue vs Expense trends.

* **Query Parameters:** Same as overview filters.
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "operations_overview": [
      { "label": "01 Jun 2026", "opd": 150, "ipd": 40, "emergency": 25, "ot": 12 },
      { "label": "15 Jun 2026", "opd": 180, "ipd": 45, "emergency": 30, "ot": 15 }
    ],
    "revenue_overview": [
      { "label": "01 Jun 2026", "revenue": 240000.00, "expenses": 180000.00 },
      { "label": "15 Jun 2026", "revenue": 280000.00, "expenses": 210000.00 }
    ]
  }
}
```

#### 2.0.3 GET /admin/dashboard/resource-utilization
Returns Staff, Bed, Ward, and Ambulance utilization metrics.

* **Query Parameters:** Same as overview filters.
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "staff_utilization": 78.8,
    "bed_utilization": 82.4,
    "ward_utilization": 68.3,
    "ambulance_utilization": 91.7
  }
}
```

#### 2.0.4 GET /admin/dashboard/reports
Returns paginated, searchable, and filtered recent reports.

* **Query Parameters:**
  * `page` (int, default: 1)
  * `page_size` (int, default: 10)
  * `search` (string, optional)
  * `status` (string, optional)
  * `department` (string, optional)
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "rep-001",
        "report_name": "OPD Summary Report",
        "department": "Outpatient",
        "reporting_period": "Jun 1 - Jun 30, 2025",
        "generated_by": "Dr. Priya Mehta",
        "generated_on": "2025-07-01T00:00:00Z",
        "status": "Pending"
      }
    ],
    "meta": {
      "total": 124,
      "page": 1,
      "page_size": 5,
      "total_pages": 25
    }
  }
}
```

#### 2.0.5 GET /admin/dashboard/statistics
Returns Hospital Occupancy split (by Ward Type) and Expense Breakdown splits.

* **Query Parameters:** Same as overview filters.
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "hospital_occupancy": [
      { "name": "ICU", "value": 82.0 },
      { "name": "General Ward", "value": 10.0 }
    ],
    "expense_breakdown": [
      { "name": "Salaries & Wages", "value": 288000.00 },
      { "name": "Maintenance & Ops", "value": 128000.00 }
    ]
  }
}
```

#### 2.0.6 GET /admin/dashboard/department-performance
Returns revenue, expenses, net profit, and profit margin metrics for each active department.

* **Query Parameters:** Same as overview filters.
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "departments": [
      {
        "department_name": "Pharmacy",
        "revenue": 230000.00,
        "expenses": 190000.00,
        "net_profit": 40000.00,
        "margin_pct": 17.4
      }
    ]
  }
}
```

#### 2.0.7 GET /admin/dashboard/report-library
Returns roster of available report templates, supported export formats, and active status.

* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "templates": [
      {
        "report_type": "REVENUE_SUMMARY",
        "name": "Revenue Summary Report",
        "description": "Aggregated daily/monthly billing revenues...",
        "category": "FINANCE",
        "supported_formats": ["PDF", "CSV", "XLSX"],
        "status": "Active"
      }
    ],
    "total": 4
  }
}
```

---

### 2.1 Dashboard Summary (Short View)
Returns a basic count overview of active staff, pending leaves, and bed capacities.

* **Endpoint:** `GET /admin/dashboard/summary`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "total_staff": 85,
    "active_staff": 78,
    "inactive_staff": 7,
    "total_patients": 542,
    "total_departments": 12,
    "total_roles": 35,
    "recent_audit_events": 234
  }
}
```

---

### 2.2 Full Aggregate Dashboard (Extended View)
Aggregates metrics from clinical admissions, workforce staffing, finances, operations, lab TAT, and pharmacy inventories.

* **Endpoint:** `GET /admin/dashboard`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "tenant_id": "a1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d",
    "generated_at": "2026-06-13T11:45:00Z",
    "patient_metrics": {
      "total_patients": 542,
      "new_today": 24,
      "opd_total": 412,
      "ipd_total": 130,
      "active_patients": 142,
      "active_admissions": 8,
      "emergency_admissions": 5,
      "discharge_pending": 3,
      "emergency_today": 12
    },
    "staff_metrics": {
      "total_staff": 85,
      "active_staff": 78,
      "inactive_staff": 7,
      "doctors_available": 18,
      "nurses_on_duty": 32,
      "leave_pending": 4
    },
    "financial_metrics": {
      "today_revenue": 345000.00,
      "pending_dues": 45000.00,
      "total_collected": 300000.00,
      "bills_pending": 15,
      "bills_today": 45,
      "payment_mode_split": [
        { "payment_mode": "UPI", "total_amount": 180000.00, "count": 28 },
        { "payment_mode": "CASH", "total_amount": 70000.00, "count": 12 },
        { "payment_mode": "CARD", "total_amount": 95000.00, "count": 5 }
      ]
    },
    "ops_metrics": {
      "total_beds": 200,
      "occupied_beds": 142,
      "available_beds": 58,
      "bed_occupancy_pct": 71.0,
      "ot_in_progress": 2,
      "ot_scheduled": 5,
      "ot_today": 3,
      "emergency_active": 4,
      "emergency_critical": 1,
      "emergency_today": 12,
      "avg_wait_minutes": 18.5
    },
    "lab_metrics": {
      "pending_tests": 12,
      "in_progress_tests": 8,
      "completed_tests": 94,
      "cancelled_tests": 1,
      "avg_tat_hours": 2.4,
      "critical_results_today": 2
    },
    "pharmacy_metrics": {
      "total_medicines": 1205,
      "active_medicines": 1150,
      "prescriptions_today": 84,
      "dispensed_today": 78,
      "pending_dispense": 6
    }
  }
}
```

---

### 2.3 Analytics: KPI Summary
* **Endpoint:** `GET /admin/analytics/kpi`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "total_patients": 542,
    "revenue_30d": 10540000.00,
    "active_ipd": 130,
    "bed_occupancy_pct": 71.00,
    "emergency_active_today": 4,
    "pending_leaves": 4
  }
}
```

---

### 2.4 Analytics Trends and Metrics
All analytics endpoints support query parameters `granularity` (`daily`, `weekly`, `monthly`) and `periods` (`int` default lookbacks).

#### Patient Inflow Trends
* **Endpoint:** `GET /admin/analytics/patient-inflow?granularity=daily&periods=7`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "metric": "patient_inflow",
    "granularity": "daily",
    "data": [
      { "date": "2026-06-07", "opd": 85, "ipd": 12, "emergency": 4 },
      { "date": "2026-06-08", "opd": 92, "ipd": 15, "emergency": 6 }
    ],
    "total": 214.0
  }
}
```

#### Revenue Trends
* **Endpoint:** `GET /admin/analytics/revenue?granularity=monthly&periods=3`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "metric": "revenue",
    "granularity": "monthly",
    "data": [
      { "period": "2026-04", "revenue": 8900000.00 },
      { "period": "2026-05", "revenue": 10540000.00 }
    ],
    "total": 19440000.00
  }
}
```

#### Doctor Performance Analytics
* **Endpoint:** `GET /admin/analytics/doctor-performance?from_date=2026-06-01&to_date=2026-06-13&limit=5`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "from_date": "2026-06-01",
    "to_date": "2026-06-13",
    "doctors": [
      {
        "doctor_id": "902d2bc4-f5ee-45df-98bd-674cd7bb0eef",
        "doctor_name": "Dr. Priya Sharma",
        "consultations_count": 84,
        "revenue_generated": 42000.00,
        "avg_rating": 4.8
      }
    ]
  }
}
```

#### Other Analytics Routes
* `GET /admin/analytics/opd-ipd-conversion` - Conversion indices.
* `GET /admin/analytics/emergency` - Emergency trends and response delays.
* `GET /admin/analytics/bed-occupancy` - Dynamic bed allocations.
* `GET /admin/analytics/ot` - Operating Theater load profiles.
* `GET /admin/analytics/lab-tat` - Lab turnaround efficiency scores.
* `GET /admin/analytics/billing` - Claims processing performance.

---

## 3. Settings & Alerting Management

### 3.1 Hospital Configuration Settings

#### Bulk Settings Upsert
* **Endpoint:** `POST /admin/settings/{category}`
* **Request Body:**
```json
{
  "settings": {
    "OPD_REGISTRATION_FEE": "200",
    "MAX_DAILY_APPOINTMENTS_PER_DOC": "30"
  }
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "e8f77d33-4dfd-4b8c-8ef8-bde88de6d4f9",
        "tenant_id": "a1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d",
        "category": "billing",
        "key": "OPD_REGISTRATION_FEE",
        "value": "200",
        "is_encrypted": false,
        "created_at": "2026-06-13T11:55:00Z",
        "updated_at": "2026-06-13T11:55:00Z"
      }
    ],
    "total": 2,
    "category": "billing"
  }
}
```

#### Single Setting Upsert (PUT)
* **Endpoint:** `PUT /admin/settings/{category}/{key}`
* **Request Body:**
```json
{
  "value": "250",
  "description": "Base pricing for standard doctor consultation registration",
  "is_encrypted": false
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "e8f77d33-4dfd-4b8c-8ef8-bde88de6d4f9",
    "tenant_id": "a1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d",
    "category": "billing",
    "key": "OPD_REGISTRATION_FEE",
    "value": "250",
    "description": "Base pricing for standard doctor consultation registration",
    "is_encrypted": false,
    "created_at": "2026-06-13T11:55:00Z",
    "updated_at": "2026-06-13T11:56:00Z"
  }
}
```

* `GET /admin/settings?category=billing` - Fetch all configuration keys in category.
* `GET /admin/settings/{category}/{key}` - Retrieve unique config value.
* `DELETE /admin/settings/{category}/{key}` - Reset configuration parameter.

---

### 3.2 Smart Alerts

#### Create Smart Alert
* **Endpoint:** `POST /admin/alerts`
* **Request Body:**
```json
{
  "alert_type": "INVENTORY_LOW",
  "severity": "HIGH",
  "title": "Critical Medicine Shortage",
  "message": "Paracetamol 650mg stock has dropped below minimum threshold (150 units remaining).",
  "source_module": "pharmacy",
  "source_entity_id": "med-uuid-123456",
  "assigned_to": "902d2bc4-f5ee-45df-98bd-674cd7bb0eef"
}
```
* **Success Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "id": "f5f77d33-4dfd-4b8c-8ef8-bde88de6d4f9",
    "tenant_id": "a1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d",
    "alert_type": "INVENTORY_LOW",
    "severity": "HIGH",
    "title": "Critical Medicine Shortage",
    "message": "Paracetamol 650mg stock has dropped below minimum threshold (150 units remaining).",
    "source_module": "pharmacy",
    "source_entity_id": "med-uuid-123456",
    "is_acknowledged": false,
    "is_resolved": false,
    "created_at": "2026-06-13T12:02:00Z",
    "updated_at": "2026-06-13T12:02:00Z"
  }
}
```

#### Acknowledge Alert
* **Endpoint:** `POST /admin/alerts/{alert_id}/acknowledge`
* **Request Body:**
```json
{
  "notes": "Checking with procurement regarding PO."
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "f5f77d33-4dfd-4b8c-8ef8-bde88de6d4f9",
    "is_acknowledged": true,
    "acknowledged_by": "902d2bc4-f5ee-45df-98bd-674cd7bb0eef",
    "acknowledged_at": "2026-06-13T12:05:00Z"
  }
}
```

* `GET /admin/alerts?is_resolved=false` - Retrieve unhandled alerts list.
* `POST /admin/alerts/scan` - Trigger rule-engine check for thresholds.
* `POST /admin/alerts/{alert_id}/resolve` - Resolve active alert.

---

## 4. Staff Management

### 4.1 Fetch Staff Directory (Admin View)
Provides robust sorting, searching, and filter indicators for active directory views.

* **Endpoint:** `GET /admin/staff?role_id=DOC-001&is_active=true&search=Priya&page=1&page_size=20`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "902d2bc4-f5ee-45df-98bd-674cd7bb0eef",
        "full_name": "Dr. Priya Sharma",
        "email": "priya.sharma@arovita.com",
        "phone": "+919123456789",
        "employee_id": "DOC-00456",
        "role_id": "DOC-001",
        "role_name": "DOCTOR",
        "is_active": true,
        "created_at": "2026-06-13T11:15:00Z"
      }
    ],
    "meta": {
      "total": 1,
      "page": 1,
      "page_size": 20,
      "total_pages": 1
    }
  }
}
```

---

### 4.2 Adjust Doctor Consultation Fees
Allows admins to change a doctor's base OPD fees.

* **Endpoint:** `PATCH /admin/staff/{user_id}/fee`
* **Request Body:**
```json
{
  "consultation_fee": 600.00
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "user_id": "902d2bc4-f5ee-45df-98bd-674cd7bb0eef",
    "consultation_fee": 600.00,
    "message": "Consultation fee updated successfully"
  }
}
```

---

### 4.3 Promotion and Role Changes
Reassigns the staff's permissions and ID prefix scopes.

* **Endpoint:** `PATCH /admin/staff/{user_id}/role`
* **Request Body:**
```json
{
  "role_id": "HNR-001"
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "user_id": "902d2bc4-f5ee-45df-98bd-674cd7bb0eef",
    "role_id": "HNR-001",
    "role_name": "HEAD_NURSE",
    "message": "Staff member role updated successfully"
  }
}
```

---

### 4.4 Custom Extra Permissions (Override Mapping)
Allows managers to assign or revoke individual permissions directly on a user without changing their role.

#### Assign Custom Permissions
* **Endpoint:** `POST /admin/staff/{user_id}/permissions`
* **Request Body:**
```json
{
  "permission_ids": ["STF-001", "BED-001"]
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "user_id": "902d2bc4-f5ee-45df-98bd-674cd7bb0eef",
    "branch_id": "a1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d",
    "custom_permissions": [
      { "permission_id": "STF-001", "permission_name": "staff:create", "module": "staff", "action": "create" },
      { "permission_id": "BED-001", "permission_name": "bed:view", "module": "bed", "action": "view" }
    ]
  }
}
```

* `GET /admin/staff/{user_id}/permissions` - Get active custom permissions.
* `DELETE /admin/staff/{user_id}/permissions` - Revoke custom overrides.

---

## 5. Roles & Permissions Management

* `GET /admin/roles?active_only=true` - Retrieve all configuration roles.
* `GET /admin/permissions?module=staff` - Retrieve all permission nodes by module.
* `GET /admin/roles/{role_id}/permissions` - Retrieve all permission nodes mapped to a specific role.

#### Map Permission to Role
* **Endpoint:** `POST /admin/roles/{role_id}/permissions`
* **Request Body:**
```json
{
  "permission_ids": ["STF-002", "BED-001"]
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "role_id": "DOC-001",
    "role_name": "DOCTOR",
    "permissions": [
      { "permission_id": "STF-002", "permission_name": "staff:view", "module": "staff", "action": "view" },
      { "permission_id": "BED-001", "permission_name": "bed:view", "module": "bed", "action": "view" }
    ]
  }
}
```

---

## 6. Departments & Specializations

### Create Department
* **Endpoint:** `POST /admin/departments`
* **Request Body:**
```json
{
  "department_name": "Cardiology"
}
```
* **Success Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "department_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "tenant_id": "a1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d",
    "department_name": "Cardiology",
    "is_active": true,
    "created_at": "2026-06-13T12:15:00Z"
  }
}
```

### List Departments (Enriched & Paginated)
* **Endpoint:** `GET /admin/departments`
* **Query Parameters:**
  * `page` (integer, optional, default: 1): Page number.
  * `page_size` (integer, optional, default: 10): Page size.
  * `search` (string, optional): Case-insensitive search on department name.
  * `status` (string, optional): Filter by occupancy status (`Healthy`, `Moderate`, `Critical`).
  * `sort_by` (string, optional, default: `department_name`): Whitelisted sort fields: `department_name`, `total_staff`, `capacity_beds`, `occupied_beds`, `available_beds`, `status`, `updated_at`.
  * `sort_order` (string, optional, default: `asc`): Sorting direction (`asc` or `desc`).
  * `include_inactive` (boolean, optional, default: false): Include inactive departments.
* **Success Response (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "department_id": "47832613-a7dc-4584-bec6-79725bf33a21",
        "department_name": "Emergency",
        "is_active": true,
        "created_at": "2026-06-15T12:48:05.606200+05:30",
        "updated_at": "2026-06-15T12:48:50.764601+05:30",
        "block": "Block A",
        "floor": "Ground Floor",
        "doctor_id": "662dde10-b6e9-4d06-b27e-1972b778f72d",
        "doctor_name": "Dr. Anil Pandey",
        "avatar_initial": "A",
        "total_staff": 3,
        "capacity_beds": 20,
        "occupied_beds": 0,
        "available_beds": 19,
        "status": "Healthy"
      }
    ],
    "pagination": {
      "page": 1,
      "page_size": 10,
      "total_records": 65,
      "total_pages": 7
    }
  }
}
```

* `PATCH /admin/departments/{department_id}` - Toggle status or name.
* `GET /admin/specializations` - List medical specialties.
* `POST /admin/specializations` - Create specialization.

---

## 7. Wards, Units, & Bed Capacity Management

### Create Ward
* **Endpoint:** `POST /admin/wards`
* **Request Body:**
```json
{
  "name": "General Ward A",
  "ward_type": "GENERAL",
  "floor": "3",
  "capacity": 30
}
```
* **Success Response (210 Created):**
```json
{
  "success": true,
  "data": {
    "id": "e8f77d33-4dfd-4b8c-8ef8-bde88de6d4f0",
    "tenant_id": "a1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d",
    "name": "General Ward A",
    "ward_type": "GENERAL",
    "floor": "3",
    "capacity": 30,
    "is_active": true,
    "created_at": "2026-06-13T12:20:00Z"
  }
}
```

---

### Create Bed
* **Endpoint:** `POST /admin/beds`
* **Request Body:**
```json
{
  "ward_id": "e8f77d33-4dfd-4b8c-8ef8-bde88de6d4f0",
  "bed_number": "GW-A01",
  "bed_type": "STANDARD"
}
```
* **Success Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "id": "d1f77d33-4dfd-4b8c-8ef8-bde88de6d4f9",
    "tenant_id": "a1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d",
    "ward_id": "e8f77d33-4dfd-4b8c-8ef8-bde88de6d4f0",
    "bed_number": "GW-A01",
    "bed_type": "STANDARD",
    "status": "AVAILABLE",
    "is_active": true,
    "created_at": "2026-06-13T12:22:00Z"
  }
}
```

#### Update Bed Status
* **Endpoint:** `PATCH /admin/beds/{bed_id}/status`
* **Request Body:**
```json
{
  "status": "MAINTENANCE",
  "reason": "Air conditioner servicing near GW-A01"
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "d1f77d33-4dfd-4b8c-8ef8-bde88de6d4f9",
    "status": "MAINTENANCE",
    "reason": "Air conditioner servicing near GW-A01"
  }
}
```

---

## 8. Leave Request Workflow

### Apply for Leave
* **Endpoint:** `POST /admin/staff/me/leaves/apply`
* **Request Body:**
```json
{
  "leave_type": "CASUAL",
  "from_date": "2026-07-10",
  "to_date": "2026-07-15",
  "reason": "Family gathering"
}
```
* **Success Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "id": "c1f77d33-4dfd-4b8c-8ef8-bde88de6d4f1",
    "user_id": "902d2bc4-f5ee-45df-98bd-674cd7bb0eef",
    "tenant_id": "a1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d",
    "leave_type": "CASUAL",
    "from_date": "2026-07-10",
    "to_date": "2026-07-15",
    "days": 6,
    "status": "PENDING",
    "created_at": "2026-06-13T12:30:00Z"
  }
}
```

---

### Approve Leave (Admin View)
* **Endpoint:** `POST /admin/leaves/{leave_id}/approve`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "c1f77d33-4dfd-4b8c-8ef8-bde88de6d4f1",
    "status": "APPROVED",
    "approved_by": "00000000-0000-0000-0000-000000000001",
    "approved_at": "2026-06-13T12:35:00Z"
  }
}
```

* `GET /admin/staff/me/leaves/balance?year=2026` - Get my leave balances.
* `GET /admin/leaves/pending` - List leaves pending review.

---

## 9. TPA (Third Party Administrator) & Insurance Claims

### Create TPA Provider
* **Endpoint:** `POST /admin/tpa/providers`
* **Request Body:**
```json
{
  "provider_name": "Star Health Insurance TPA",
  "short_name_code": "STAR-TPA",
  "contact_name": "Mr. Rajesh Kumar",
  "contact_email": "tpa.star@starhealth.in",
  "contact_phone": "+919999888877",
  "status": "ACTIVE"
}
```
* **Success Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "id": "tpa-uuid-123456",
    "provider_name": "Star Health Insurance TPA",
    "short_name_code": "STAR-TPA",
    "status": "ACTIVE",
    "created_at": "2026-06-13T12:40:00Z"
  }
}
```

* `GET /admin/tpa/providers` - Query TPAs.
* `POST /admin/tpa/schemes` - Link dynamic policy coverage parameters.
* `PATCH /admin/tpa/claims/{claim_id}` - Update authorization parameters.

---

## 10. Inventory & Pharmacy Stock Management

### Create Medical Inventory Stock
* **Endpoint:** `POST /admin/inventory/medical`
* **Request Body:**
```json
{
  "item_code": "MED-PAR-650",
  "item_name": "Paracetamol 650mg Tablets",
  "category": "TABLETS",
  "batch_number": "BATCH-2026-A",
  "expiry_date": "2028-05-31",
  "opening_balance": 5000,
  "reorder_level": 500,
  "mrp": 2.50
}
```
* **Success Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "id": "inv-uuid-123456",
    "item_code": "MED-PAR-650",
    "item_name": "Paracetamol 650mg Tablets",
    "status_level": "GOOD",
    "created_at": "2026-06-13T12:45:00Z"
  }
}
```

* `GET /admin/inventory?item_type=medical` - Query inventory levels.
* `POST /admin/inventory/transfers` - Move stocks between departments (e.g. Pharmacy to Emergency Ward).

---

## 11. Procurement Workflows

### Create Purchase Order (PO)
* **Endpoint:** `POST /admin/procurement/orders`
* **Request Body:**
```json
{
  "vendor_id": "vendor-uuid-123456",
  "expected_delivery_date": "2026-06-25",
  "items": [
    { "item_code": "MED-PAR-650", "quantity": 10000, "unit_price": 2.10 }
  ]
}
```
* **Success Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "po_id": "po-uuid-123456",
    "po_number": "PO-2026-0089",
    "vendor_id": "vendor-uuid-123456",
    "status": "DRAFT",
    "total_amount": 21000.00,
    "created_at": "2026-06-13T12:50:00Z"
  }
}
```

* `POST /admin/procurement/grn` - Receive items against a PO (Goods Receipt Note).
* `POST /admin/procurement/invoices/{invoice_id}/reconcile` - Reconcile vendor billing values.

---

## 12. Security Compliance & Audit Trail Logs

### Search Security Audit Logs
* **Endpoint:** `GET /admin/compliance/audit?action=staff.role.update&page=1&page_size=50`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "log-uuid-987654",
        "user_id": "00000000-0000-0000-0000-000000000001",
        "user_name": "Super Admin",
        "user_email": "admin@arovita.com",
        "action": "staff.role.update",
        "entity": "staff_profile",
        "entity_id": "902d2bc4-f5ee-45df-98bd-674cd7bb0eef",
        "role_id": "SYS-001",
        "status": "success",
        "ip_address": "103.111.42.2",
        "metadata": { "old_role": "DOCTOR", "new_role": "HEAD_NURSE" },
        "created_at": "2026-06-13T12:00:00Z"
      }
    ],
    "meta": {
      "total": 1,
      "page": 1,
      "page_size": 50,
      "total_pages": 1
    }
  }
}
```

* `GET /admin/compliance/suspicious-activity` - Query anomalies (e.g. multi-IP accesses).
* `GET /admin/compliance/security-score` - Security parameter status grades.

---

## 13. Wards & Beds Management

All endpoints require `Authorization: Bearer <access_token>`.

> **Enums**
> - `ward_type`: `GENERAL` | `PRIVATE` | `ICU` | `HDU` | `EMERGENCY` | `MATERNITY` | `PAEDIATRIC` | `SURGICAL`
> - `bed_type`: `STANDARD` | `ICU` | `HDU` | `ISOLATION` | `RECOVERY` | `MATERNITY`
> - `bed_status`: `AVAILABLE` | `OCCUPIED` | `RESERVED` | `MAINTENANCE`

---

### 13.1 Blocks

Physical building blocks (e.g. "Block A", "Block B", "OPD Block").

#### Create Block
* **Endpoint:** `POST /admin/blocks`
* **Request Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `floor_id` | UUID | ✅ | Parent Floor UUID |
| `name` | string | ✅ | Block name (2–100 chars) |
| `code` | string | ✅ | Unique block code per facility (1–50 chars) |
| `description` | string | ➖ | Optional description (max 500 chars) |

```json
{
  "floor_id": "22222222-2222-2222-2222-222222222222",
  "name": "Block B",
  "code": "BLK-B",
  "description": "Critical care building block"
}
```
* **Success Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "id": "11111111-1111-1111-1111-111111111111",
    "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "floor_id": "22222222-2222-2222-2222-222222222222",
    "name": "Block B",
    "code": "BLK-B",
    "description": "Critical care building block",
    "is_active": true,
    "created_at": "2026-07-10T12:00:00Z"
  }
}
```

#### List Blocks
* **Endpoint:** `GET /admin/blocks`
* **Query Parameters:**
  * `floor_id` *(optional)* — Filter blocks belonging to a specific floor.
  * `include_inactive` *(optional)* — `true` | `false` (default: `false`)
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "11111111-1111-1111-1111-111111111111",
        "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
        "floor_id": "22222222-2222-2222-2222-222222222222",
        "name": "Block B",
        "code": "BLK-B",
        "description": "Critical care building block",
        "is_active": true,
        "created_at": "2026-07-10T12:00:00Z"
      }
    ],
    "total": 1
  }
}
```

#### Get Block by ID
* **Endpoint:** `GET /admin/blocks/{block_id}`
* **Success Response (200 OK):** Same as individual item in List Blocks response.

#### Update Block
* **Endpoint:** `PATCH /admin/blocks/{block_id}`
* **Request Body** *(all fields optional)*:

| Field | Type | Description |
|---|---|---|
| `floor_id` | UUID | Re-associate block to a different Floor |
| `name` | string | Updated block name |
| `code` | string | Updated block code |
| `description` | string | Updated description |
| `is_active` | boolean | `true` to reactivate, `false` to deactivate |

```json
{
  "name": "Block B - ICU Wing",
  "floor_id": "22222222-2222-2222-2222-222222222222"
}
```
* **Success Response (200 OK):** Updated `BlockResponse` object.

---

### 13.2 Floors

Floors allocated inside the hospital facility.

#### Create Floor
* **Endpoint:** `POST /admin/floors`
* **Request Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `floor_number` | integer | ✅ | Numeric floor number (e.g. `0` for Ground, `1`, `2`, `-1` for Basement) |
| `floor_label` | string | ✅ | Display label (e.g. `G`, `1`, `B1`, `3rd Floor`) |

```json
{
  "floor_number": 3,
  "floor_label": "3"
}
```
* **Success Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "id": "22222222-2222-2222-2222-222222222222",
    "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "floor_number": 3,
    "floor_label": "3",
    "is_active": true,
    "created_at": "2026-07-10T12:05:00Z"
  }
}
```

#### List Floors
* **Endpoint:** `GET /admin/floors`
* **Query Parameters:**
  * `include_inactive` *(optional)* — `true` | `false` (default: `false`)
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "22222222-2222-2222-2222-222222222222",
        "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
        "floor_number": 3,
        "floor_label": "3",
        "is_active": true,
        "created_at": "2026-07-10T12:05:00Z"
      }
    ],
    "total": 1
  }
}
```

#### Get Floor by ID
* **Endpoint:** `GET /admin/floors/{floor_id}`
* **Success Response (200 OK):** Same as individual item in List Floors response.

#### Update Floor
* **Endpoint:** `PATCH /admin/floors/{floor_id}`
* **Request Body** *(all fields optional)*:

| Field | Type | Description |
|---|---|---|
| `floor_number` | integer | Updated floor number |
| `floor_label` | string | Updated floor label |
| `is_active` | boolean | `true` to reactivate, `false` to deactivate |

```json
{
  "floor_label": "Third Floor (ICU)"
}
```
* **Success Response (200 OK):** Updated `FloorResponse` object.

---

### 13.3 Wards

#### List Wards
* **Endpoint:** `GET /admin/wards`
* **Query Parameters:**
  * `ward_type` *(optional)* — Filter by type (e.g. `ICU`)
  * `is_active` *(optional)* — `true` | `false`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "4c5d12d2-79dd-4650-867e-079ac3b85a5a",
        "tenant_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
        "name": "General Ward A",
        "ward_type": "GENERAL",
        "floor": "1",
        "capacity": 30,
        "is_active": true,
        "created_at": "2026-01-10T09:00:00Z",
        "total_beds": 30,
        "available_beds": 18,
        "occupied_beds": 12
      }
    ],
    "total": 1
  }
}
```

---

#### Create Ward
* **Endpoint:** `POST /admin/wards`
* **Request Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string | ✅ | Ward name (2–100 chars) |
| `ward_type` | enum | ✅ | `GENERAL` / `ICU` / `SURGICAL` / etc. |
| `capacity` | integer | ✅ | Total capacity (1–500) |
| `floor` | string | ➖ | Legacy floor label (e.g. `G`, `1`) |
| `block_id` | UUID | ➖ | Physical Block UUID link |
| `work_area_id` | UUID | ➖ | Links ward to a departmental work area |

```json
{
  "name": "ICU Block B",
  "ward_type": "ICU",
  "capacity": 12,
  "block_id": "11111111-1111-1111-1111-111111111111",
  "work_area_id": "a1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d"
}
```
* **Success Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "id": "4c5d12d2-79dd-4650-867e-079ac3b85a5a",
    "tenant_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "name": "ICU Block B",
    "ward_type": "ICU",
    "floor": "3",
    "block_id": "11111111-1111-1111-1111-111111111111",
    "capacity": 12,
    "is_active": true,
    "created_at": "2026-07-01T09:00:00Z",
    "total_beds": 0,
    "available_beds": 0,
    "occupied_beds": 0
  }
}
```

---

#### Get Ward by ID
* **Endpoint:** `GET /admin/wards/{ward_id}`
* **Success Response (200 OK):** Same as individual item in List Wards response.
* **Error (404):** `{ "success": false, "message": "Ward not found." }`

---

#### Update Ward
* **Endpoint:** `PATCH /admin/wards/{ward_id}`
* **Request Body** *(all fields optional)*:

| Field | Type | Description |
|---|---|---|
| `name` | string | New ward name |
| `ward_type` | enum | Updated type |
| `capacity` | integer | Updated total capacity (1–500) |
| `floor` | string | Updated legacy floor label |
| `block_id` | UUID | Updated physical Block UUID |
| `is_active` | boolean | `true` to reactivate, `false` to deactivate |

```json
{
  "capacity": 15,
  "block_id": "11111111-1111-1111-1111-111111111111",
  "is_active": true
}
```
* **Success Response (200 OK):** Updated `WardResponse` object.

---

#### List Ward Units (Sub-sections)
* **Endpoint:** `GET /admin/wards/{ward_id}/units`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": "7a3f1b9c-2d4e-5f6a-7b8c-9d0e1f2a3b4c",
      "ward_id": "4c5d12d2-79dd-4650-867e-079ac3b85a5a",
      "ward_name": "ICU Block B",
      "name": "Bay 1",
      "code": "ICU-B1",
      "capacity": 4,
      "is_active": true
    }
  ]
}
```

---

#### Create Ward Unit
* **Endpoint:** `POST /admin/wards/{ward_id}/units`
* **Request Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string | ✅ | Unit/bay name (max 120 chars) |
| `code` | string | ➖ | Short code e.g. `ICU-B1` (max 50 chars) |
| `description` | string | ➖ | Notes (max 500 chars) |
| `capacity` | integer | ➖ | Bed capacity for this unit (default `0`) |

```json
{
  "name": "Bay 1",
  "code": "ICU-B1",
  "description": "Critical monitoring bay",
  "capacity": 4
}
```
* **Success Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "id": "7a3f1b9c-2d4e-5f6a-7b8c-9d0e1f2a3b4c",
    "ward_id": "4c5d12d2-79dd-4650-867e-079ac3b85a5a",
    "ward_name": "ICU Block B",
    "name": "Bay 1",
    "code": "ICU-B1",
    "description": "Critical monitoring bay",
    "capacity": 4,
    "is_active": true,
    "created_at": "2026-07-01T09:05:00Z"
  }
}
```

---

### 13.2 Beds

#### List Beds
* **Endpoint:** `GET /admin/beds`
* **Query Parameters:**
  * `ward_id` *(optional)* — Filter by ward UUID
  * `status` *(optional)* — `AVAILABLE` | `OCCUPIED` | `RESERVED` | `MAINTENANCE`
  * `bed_type` *(optional)* — Filter by bed type
  * `page` *(optional, default `1`)*
  * `page_size` *(optional, default `20`)*
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "84820843-9876-4321-9abc-1234567890ab",
        "tenant_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
        "ward_id": "4c5d12d2-79dd-4650-867e-079ac3b85a5a",
        "ward_name": "ICU Block B",
        "floor": "3",
        "bed_number": "ICU-B-001",
        "bed_type": "ICU",
        "status": "AVAILABLE",
        "is_active": true,
        "created_at": "2026-01-15T09:00:00Z"
      }
    ],
    "meta": {
      "total": 1,
      "page": 1,
      "page_size": 20,
      "total_pages": 1
    }
  }
}
```

---

#### Bed Availability Summary
* **Endpoint:** `GET /admin/beds/summary`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "total_beds": 120,
    "available": 45,
    "occupied": 62,
    "reserved": 8,
    "maintenance": 5,
    "by_ward": [
      {
        "ward_id": "4c5d12d2-79dd-4650-867e-079ac3b85a5a",
        "ward_name": "ICU Block B",
        "ward_type": "ICU",
        "floor": "3",
        "capacity": 12,
        "total_beds": 12,
        "available": 4,
        "occupied": 7,
        "reserved": 1,
        "maintenance": 0
      }
    ]
  }
}
```

---

#### Get Bed by ID
* **Endpoint:** `GET /admin/beds/{bed_id}`
* **Success Response (200 OK):** Single `BedResponse` object (same as items array above).
* **Error (404):** `{ "success": false, "message": "Bed not found." }`

---

#### Create Bed
* **Endpoint:** `POST /admin/beds`
* **Request Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `ward_id` | UUID | ✅ | Ward the bed belongs to |
| `bed_number` | string | ✅ | Unique bed identifier e.g. `ICU-B-001` (max 20 chars) |
| `bed_type` | enum | ➖ | Default `STANDARD` |

```json
{
  "ward_id": "4c5d12d2-79dd-4650-867e-079ac3b85a5a",
  "bed_number": "ICU-B-002",
  "bed_type": "ICU"
}
```
* **Success Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "id": "a1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d",
    "tenant_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "ward_id": "4c5d12d2-79dd-4650-867e-079ac3b85a5a",
    "ward_name": "ICU Block B",
    "floor": "3",
    "bed_number": "ICU-B-002",
    "bed_type": "ICU",
    "status": "AVAILABLE",
    "is_active": true,
    "created_at": "2026-07-01T09:10:00Z"
  }
}
```
* **Error (409):** `{ "success": false, "message": "Bed number already exists in this ward." }`

---

#### Update Bed
* **Endpoint:** `PATCH /admin/beds/{bed_id}`
* **Request Body** *(all fields optional)*:

| Field | Type | Description |
|---|---|---|
| `bed_number` | string | New bed number |
| `bed_type` | enum | Updated type |
| `is_active` | boolean | Deactivate / reactivate the bed |

```json
{
  "bed_number": "ICU-B-002A",
  "is_active": true
}
```
* **Success Response (200 OK):** Updated `BedResponse` object.

---

#### Update Bed Status
* **Endpoint:** `PATCH /admin/beds/{bed_id}/status`
* **Request Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `status` | enum | ✅ | `AVAILABLE` / `OCCUPIED` / `RESERVED` / `MAINTENANCE` |
| `reason` | string | ➖ | Reason for status change (max 500 chars) |

```json
{
  "status": "MAINTENANCE",
  "reason": "Deep cleaning scheduled after discharge."
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "84820843-9876-4321-9abc-1234567890ab",
    "bed_number": "ICU-B-001",
    "status": "MAINTENANCE",
    "is_active": true
  }
}
```

---

## 14. Compliance & Roles Configuration Module

All endpoints require `Authorization: Bearer <access_token>`.

### 14.1 Compliance Dashboard

#### System Audit Logs Overview
* **Endpoint:** `GET /admin/compliance/audit-overview`
* **Query Parameters:**
  * `page` (optional, default: `1`)
  * `page_size` (optional, default: `10`)
  * `severity` (optional) - Filter by `CRITICAL`, `WARNING`, `INFO`
  * `user` (optional) - Substring search for user full name/email
  * `module` (optional) - Substring search for action name
  * `fromDate` (optional) - ISO datetime string
  * `toDate` (optional) - ISO datetime string
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "metrics": {
      "totalEvents": 450,
      "criticalEvents": 12,
      "failedLogins": 85,
      "warningsCount": 30
    },
    "logs": [
      {
        "id": "log-uuid-123456",
        "userName": "Alice Smith",
        "userEmail": "alice@hms.com",
        "action": "user.login",
        "module": "auth",
        "status": "success",
        "severity": "INFO",
        "timestamp": "2026-07-08T10:15:00Z",
        "ipAddress": "192.168.1.1",
        "metadata": {}
      }
    ],
    "pagination": {
      "total": 450,
      "page": 1,
      "pageSize": 10,
      "totalPages": 45
    }
  }
}
```

#### Get Security Settings
* **Endpoint:** `GET /admin/compliance/security-settings`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "twoFactorAuthentication": true,
    "autoLogoutDuration": "30 Minutes",
    "emergencyAlerts": ["sms", "email"]
  }
}
```

#### Update Security Settings
* **Endpoint:** `PATCH /admin/compliance/security-settings`
* **Request Body:**
```json
{
  "twoFactorAuthentication": true,
  "autoLogoutDuration": "45 Minutes",
  "emergencyAlerts": ["sms", "in-app", "email"]
}
```
* **Success Response (200 OK):** Updated settings model.

#### Security Analytics & Login Trends
* **Endpoint:** `GET /admin/compliance/security-analytics`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "securityEvents": {
      "labels": ["Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"],
      "successfulLogins": [120, 150, 110, 130, 140, 90, 80],
      "failedLogins": [5, 2, 8, 1, 3, 12, 0],
      "criticalEvents": [0, 1, 0, 0, 2, 0, 1]
    },
    "activitySummary": {
      "userActivities": 5400,
      "configurationChanges": 34,
      "resourceUpdates": 128,
      "financialChanges": 92
    }
  }
}
```

#### Role Access matrix & Privileged Users
* **Endpoint:** `GET /admin/compliance/access-overview`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "accessMatrix": [
      {
        "module": "Compliance & Security",
        "admin": "Active",
        "doctor": "None",
        "nurse": "None",
        "receptionist": "None",
        "billing": "None"
      }
    ],
    "privilegedUsers": [
      {
        "name": "Super Admin",
        "role": "Super Admin",
        "accessLevel": "ADMIN",
        "lastLogin": "2026-07-08T12:00:00Z",
        "status": "Active"
      }
    ],
    "elevatedAccess": [
      {
        "id": "event-uuid",
        "description": "Permissions assigned to Doctor role",
        "timestamp": "2026-07-08T09:30:00Z",
        "severity": "CRITICAL"
      }
    ]
  }
}
```

#### Export Compliance Audit Report
* **Endpoint:** `GET /admin/compliance/report`
* **Query Parameters:**
  * `format` (optional, default: `PDF`, options: `PDF`, `EXCEL`, `CSV`)
* **Success Response (200 OK):** Returns raw binary attachment with content types `application/pdf`, `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`, or `text/csv`.

---

### 14.2 Roles and Permissions Configurator

#### GET Role Details
* **Endpoint:** `GET /admin/roles/{roleId}`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "roleId": "MED-001",
    "roleName": "Lead Physician",
    "hierarchy": 10,
    "departmentScope": "CLINICAL",
    "isSuperAdmin": false
  }
}
```

#### PATCH Modify Role Configuration
* **Endpoint:** `PATCH /admin/roles/{roleId}`
* **Request Body:**
```json
{
  "roleName": "Lead Clinical Physician",
  "hierarchy": 12,
  "departmentScope": "CLINICAL"
}
```
* **Success Response (200 OK):** Updated role details.

#### GET Pending Override Requests
* **Endpoint:** `GET /admin/roles/pending-requests`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": "request-uuid-abc",
      "tenantId": "branch-uuid",
      "requestType": "ROLE_CHANGE",
      "requesterName": "Admin user",
      "targetUserName": "Dr. Smith",
      "roleId": "MED-001",
      "roleName": "Lead Physician",
      "reason": "Promotion",
      "status": "PENDING",
      "createdAt": "2026-07-08T08:00:00Z"
    }
  ]
}
```

#### POST Approve Override Request
* **Endpoint:** `POST /admin/roles/pending-requests/{requestId}/approve`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Role request approved and applied successfully."
}
```

#### POST Reject Override Request
* **Endpoint:** `POST /admin/roles/pending-requests/{requestId}/reject`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Role request rejected successfully."
}
```

#### GET Role Granular Permissions
* **Endpoint:** `GET /admin/roles/{roleId}/permissions`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "roleId": "MED-001",
    "roleName": "Lead Physician",
    "categories": [
      {
        "categoryName": "Clinical & Patient Care",
        "permissions": [
          {
            "id": "CMP-001",
            "code": "compliance:view",
            "name": "View Compliance Logs",
            "isActive": true,
            "isDraftActive": true
          }
        ]
      }
    ],
    "hasDraftChanges": false
  }
}
```

#### PATCH Modify Permissions (Save to Draft)
* **Endpoint:** `PATCH /admin/roles/{roleId}/permissions`
* **Request Body:**
```json
{
  "permissionIds": ["CMP-001", "CMP-002", "ROL-001"]
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Permissions updated in draft state."
}
```

#### POST Publish Role Draft Permissions
* **Endpoint:** `POST /admin/roles/{roleId}/publish`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "roleId": "MED-001",
    "roleName": "Lead Physician",
    "publishedCount": 3
  }
}
```

#### POST Discard Role Draft Permissions
* **Endpoint:** `POST /admin/roles/{roleId}/discard`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Draft permission modifications discarded."
}
```
```
