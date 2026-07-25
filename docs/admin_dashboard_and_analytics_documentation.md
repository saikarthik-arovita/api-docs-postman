# Admin Dashboard & Analytics API Documentation

This document provides specifications for the **Dashboard Overview, Performance Statistics, and Analytical Metrics** within the `hms-admin` microservice.

---

## Global Environment & Configuration
* **Base URL**: `https://vo585bjv4a.execute-api.ap-south-1.amazonaws.com/dev`
* **Authentication**: Bearer Token (Cognito / HMS JWT Access Token)

---

## Global Query Filter Parameters

The dashboard and analytics endpoints accept the following filter query parameters to segment metrics:

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `hospitalId` | UUID | Filter metrics by specific hospital (Super Admin only). |
| `branchId` | UUID | Filter metrics by specific facility/branch. |
| `departmentId` | UUID | Filter metrics by clinical/support department. |
| `fromDate` | String | Lower bound date limit (ISO Date format: `YYYY-MM-DD`). |
| `toDate` | String | Upper bound date limit (ISO Date format: `YYYY-MM-DD`). |
| `timeRange` | String | Pre-defined date filter range: `"today"`, `"this-week"`, `"this-month"`, `"last-30-days"`, `"last-90-days"`, `"all-time"`. |

---

## 1. Dashboard Aggregate APIs

### 1.1 GET Dashboard Summary
Retrieve high-level staff, patient, and operational totals.

* **Method**: `GET`
* **URL**: `/admin/dashboard/summary`
* **Query Parameters**: *Global query filters*

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "total_staff": 150,
    "active_staff": 145,
    "inactive_staff": 5,
    "total_patients": 10500,
    "total_departments": 26,
    "total_roles": 12,
    "recent_audit_events": 45
  }
}
```

---

### 1.2 GET Dashboard Overview
Retrieve chronological outpatient vs inpatient daily flow counts.

* **Method**: `GET`
* **URL**: `/admin/dashboard/overview`
* **Query Parameters**: *Global query filters*

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "total_patients": 214,
    "growth_rate_pct": 10.0,
    "outpatient_count": 150,
    "inpatient_count": 64
  }
}
```

---

### 1.3 GET Hospital Performance
Retrieve operational metrics such as average length of stay and revenue.

* **Method**: `GET`
* **URL**: `/admin/dashboard/hospital-performance`
* **Query Parameters**: *Global query filters*

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "occupancy_rate_pct": 82.5,
    "average_length_of_stay_days": 4.5,
    "inpatient_admissions_count": 64,
    "outpatient_consultations_count": 150
  }
}
```

---

### 1.4 GET Resource Utilization
Retrieve active usage metrics for crucial assets like Operating Theatres and ICU Beds.

* **Method**: `GET`
* **URL**: `/admin/dashboard/resource-utilization`
* **Query Parameters**: *Global query filters*

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "operating_theatres_utilization_pct": 74.2,
    "icu_beds_utilization_pct": 85.0,
    "ventilators_utilization_pct": 40.0
  }
}
```

---

### 1.5 GET Dashboard Reports list
Retrieve recently generated clinical or inventory reports metadata.

* **Method**: `GET`
* **URL**: `/admin/dashboard/reports`
* **Query Parameters**:
  * `page` (*Optional*, Default: `1`): Current page integer.
  * `page_size` (*Optional*, Default: `10`): Items limit.
  * `search` (*Optional*): Filter by report name.
  * `status` (*Optional*): Filter by status (`SUCCESS`, `PENDING`, `FAILED`).

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "report_id": "r1-93920321-63e8-4646-ba4f-dc976ec6dfda",
        "report_name": "Monthly Revenue Report - June 2026",
        "generated_at": "2026-07-01T10:00:00.000Z",
        "generated_by": "System Administrator",
        "status": "SUCCESS"
      }
    ],
    "total": 45
  }
}
```

---

## 2. Standalone Analytics APIs

### 2.1 GET KPI Summary
Retrieve core Hospital KPIs (ARPOB, bed turnovers).

* **Method**: `GET`
* **URL**: `/admin/analytics/kpi`
* **Query Parameters**: *Global query filters*

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "average_revenue_per_occupied_bed": 15000.00,
    "bed_turnover_rate": 2.4,
    "net_promoter_score": 88.5
  }
}
```

---

### 2.2 GET Patient Inflow Analytics
Timeline data tracking patients arriving at OPD and Emergency.

* **Method**: `GET`
* **URL**: `/admin/analytics/patient-inflow`
* **Query Parameters**: *Global query filters*

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "timeline": [
      { "date": "2026-07-20", "opd": 120, "emergency": 25, "ipd_admission": 15 },
      { "date": "2026-07-21", "opd": 135, "emergency": 30, "ipd_admission": 18 }
    ]
  }
}
```

---

### 2.3 GET Revenue Analytics
Retrieve financial inflow segmented by departments.

* **Method**: `GET`
* **URL**: `/admin/analytics/revenue`
* **Query Parameters**: *Global query filters*

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "total_revenue": 1245000.00,
    "currency": "INR (₹)",
    "breakdown_by_department": [
      { "department_name": "Cardiology", "amount": 450000.00 },
      { "department_name": "Orthopedics", "amount": 350000.00 }
    ]
  }
}
```

---

### 2.4 GET Laboratory TAT Compliance
Retrieve statistics on Laboratory Turnaround Time compliance.

* **Method**: `GET`
* **URL**: `/admin/analytics/lab-tat`
* **Query Parameters**: *Global query filters*

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "total_tests_processed": 1450,
    "tat_compliant_tests": 1390,
    "tat_compliance_rate_pct": 95.8,
    "average_delay_mins": 12.4
  }
}
```
