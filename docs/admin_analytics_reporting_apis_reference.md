# Admin Advanced Analytics & Reporting API Documentation

This document provides a comprehensive API reference for the **Advanced Analytics & Reporting** module of the HMS Admin Dashboard. These endpoints power dashboards for patient flow, financial performance, clinical metrics, operational efficiency, and automated PDF/CSV report generation.

---

## 1. Global Authorization & Headers

* **Authorization**: All endpoints require a valid Cognito JWT Access Token passed in the `Authorization` header:
  * `Authorization: Bearer <jwt_access_token>`
* **RBAC Permission ID**: `SYS-001` (System Admin access or Super Admin bypass) is required to view these dashboards.
* **Tenant Scoping**: All queries are tenant-scoped automatically by resolving the caller's active `branch_id` from their session in the DB.

---

## 2. API Endpoints Reference

### 1. Analytics KPI Summary
* **Endpoint**: `GET /admin/analytics/kpi`
* **Query Parameters**: None
* **Description**: Returns top-level overview metric counts for the hospital including active patients, 30-day revenue trends, active admissions, bed occupancy percentage, pending leaves, and emergency stats.
* **Sample Response**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "total_patients": 2060,
    "totalPatients": 2060,
    "revenue_30d": "197528.50",
    "revenue30d": "197528.50",
    "active_ipd": 36,
    "activeIpd": 36,
    "bed_occupancy_pct": "9.8",
    "bedOccupancyPct": "9.8",
    "emergency_active_today": 0,
    "emergencyActiveToday": 0,
    "pending_leaves": 0,
    "pendingLeaves": 0,
    "procedures_performed_today": 0,
    "proceduresPerformedToday": 0,
    "pending_payments_value": "2007690.00",
    "pendingPaymentsValue": "2007690.00",
    "conversion_rate_pct": "0.9",
    "conversionRatePct": "0.9"
  }
}
```

---

### 2. Patient Inflow Trend
* **Endpoint**: `GET /admin/analytics/patient-inflow`
* **Query Parameters**:
  * `granularity` (String, default: `daily`): Interval grouping (`daily`, `weekly`, `monthly`, `yearly`).
  * `periods` (Integer, default: `30`): Number of history intervals to return.
* **Description**: Returns trend analytics of patient registrations broken down by outpatient (OPD) and inpatient (IPD) counts.
* **Sample Response**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "metric": "patient_inflow",
    "granularity": "daily",
    "data": [
      {
        "label": "21 Jul 2026",
        "total": 6,
        "opd": 6,
        "ipd": 0
      },
      {
        "label": "22 Jul 2026",
        "total": 3,
        "opd": 3,
        "ipd": 0
      },
      {
        "label": "23 Jul 2026",
        "total": 3,
        "opd": 3,
        "ipd": 0
      }
    ],
    "total": 21.0
  }
}
```

---

### 3. Financial Performance (Revenue & Bills)
* **Endpoint**: `GET /admin/analytics/revenue`
* **Query Parameters**:
  * `granularity` (String, default: `monthly`): Interval grouping (`daily`, `weekly`, `monthly`, `yearly`).
  * `periods` (Integer, default: `12`): Number of history intervals to return.
* **Description**: Returns financial performance indicators such as total billed amount, total collected amount, outstanding balances, and total bills processed.
* **Sample Response**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "metric": "revenue",
    "granularity": "monthly",
    "data": [
      {
        "label": "Jun 2026",
        "total_billed": "29000.00",
        "total_collected": "1000.00",
        "outstanding": "28000.00",
        "bill_count": 58
      },
      {
        "label": "Jul 2026",
        "total_billed": "2174450.00",
        "total_collected": "197528.50",
        "outstanding": "1979690.00",
        "bill_count": 1957
      }
    ],
    "total": 198528.5
  }
}
```

---

### 4. OPD to IPD Conversion Rate
* **Endpoint**: `GET /admin/analytics/opd-ipd-conversion`
* **Query Parameters**:
  * `granularity` (String, default: `monthly`): Interval grouping (`daily`, `weekly`, `monthly`, `yearly`).
  * `periods` (Integer, default: `6`): Number of history intervals to return.
* **Description**: Tracks the conversion rate of patients visiting Outpatient Departments (OPD) who end up admitted as Inpatients (IPD).
* **Sample Response**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "metric": "opd_ipd_conversion",
    "granularity": "monthly",
    "data": [
      {
        "label": "Jun 2026",
        "opd_visits": 60,
        "converted_to_ipd": 8,
        "conversion_pct": "13.3"
      },
      {
        "label": "Jul 2026",
        "opd_visits": 1848,
        "converted_to_ipd": 16,
        "conversion_pct": "0.9"
      }
    ],
    "total": 1908.0
  }
}
```

---

### 5. Doctor Performance Analytics
* **Endpoint**: `GET /admin/analytics/doctor-performance`
* **Query Parameters**:
  * `from_date` (String, optional, `YYYY-MM-DD`): Start date filter.
  * `to_date` (String, optional, `YYYY-MM-DD`): End date filter.
  * `limit` (Integer, default: `20`): Maximum number of doctors to retrieve.
* **Description**: Lists medical staff activity metrics including OPD consult counts, IPD patient allocations, completed surgery counts, and average consultation durations.
* **Sample Response**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "from_date": "2026-07-01",
    "to_date": "2026-07-23",
    "doctors": [
      {
        "doctor_id": "77fa4c12-2247-4081-8d2c-721d2cb91f5b",
        "doctor_name": "Dr. Meera Das",
        "role_name": "DOCTOR",
        "opd_visits": 79,
        "ipd_patients": 0,
        "surgeries": 0,
        "avg_consult_minutes": "0.1"
      },
      {
        "doctor_id": "52197243-3cb3-40d4-aaeb-15c77f5f4f3c",
        "doctor_name": "Dr. Aarav Shrivastava",
        "role_name": "DOCTOR",
        "opd_visits": 68,
        "ipd_patients": 0,
        "surgeries": 0,
        "avg_consult_minutes": "0.1"
      }
    ]
  }
}
```

---

### 6. Emergency Analytics
* **Endpoint**: `GET /admin/analytics/emergency`
* **Query Parameters**:
  * `granularity` (String, default: `daily`): Interval grouping (`daily`, `weekly`, `monthly`, `yearly`).
  * `periods` (Integer, default: `30`): Number of history intervals to return.
* **Description**: Tracks emergency room inflow volume categorized by critical, urgent, moderate, and mild triage levels, along with IPD admission counts.
* **Sample Response**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "metric": "emergency",
    "granularity": "daily",
    "data": [
      {
        "label": "19 Jul 2026",
        "total": 1,
        "critical": 0,
        "urgent": 0,
        "moderate": 0,
        "mild": 1,
        "admitted": 0
      },
      {
        "label": "20 Jul 2026",
        "total": 1,
        "critical": 1,
        "urgent": 0,
        "moderate": 0,
        "mild": 0,
        "admitted": 0
      }
    ],
    "total": 2.0
  }
}
```

---

### 7. Bed Occupancy Trends
* **Endpoint**: `GET /admin/analytics/bed-occupancy`
* **Query Parameters**:
  * `granularity` (String, default: `daily`): Interval grouping (`daily`, `weekly`, `monthly`, `yearly`).
  * `periods` (Integer, default: `30`): Number of history intervals to return.
* **Description**: Tracks IPD admission counts, patient discharges, and emergency-sourced admissions.
* **Sample Response**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "metric": "bed_occupancy",
    "granularity": "daily",
    "data": [
      {
        "label": "18 Jul 2026",
        "admissions": 2,
        "discharges": 1,
        "emergency_admissions": 0
      },
      {
        "label": "20 Jul 2026",
        "admissions": 1,
        "discharges": 0,
        "emergency_admissions": 0
      }
    ],
    "total": 3.0
  }
}
```

---

### 8. Operation Theatre (OT) Performance
* **Endpoint**: `GET /admin/analytics/ot`
* **Query Parameters**:
  * `granularity` (String, default: `monthly`): Interval grouping (`daily`, `weekly`, `monthly`, `yearly`).
  * `periods` (Integer, default: `6`): Number of history intervals to return.
* **Description**: Returns surgical analytics including total sessions scheduled, completed, cancelled surgeries, and average duration of surgeries.
* **Sample Response**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "metric": "ot_utilization",
    "granularity": "monthly",
    "data": [
      {
        "label": "Jun 2026",
        "total_sessions": 10,
        "completed": 1,
        "cancelled": 0,
        "avg_surgery_minutes": "0.0"
      },
      {
        "label": "Jul 2026",
        "total_sessions": 36,
        "completed": 14,
        "cancelled": 0,
        "avg_surgery_minutes": "0.0"
      }
    ],
    "total": 46.0
  }
}
```

---

### 9. Laboratory Turnaround Time (TAT)
* **Endpoint**: `GET /admin/analytics/lab-tat`
* **Query Parameters**:
  * `granularity` (String, default: `weekly`): Interval grouping (`daily`, `weekly`, `monthly`, `yearly`).
  * `periods` (Integer, default: `12`): Number of history intervals to return.
* **Description**: Tracks lab test statistics including total tests completed, critical result flags, and average turnaround time (TAT) in hours.
* **Sample Response**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "metric": "lab_tat",
    "granularity": "weekly",
    "data": [
      {
        "label": "2026-26",
        "completed_tests": 3,
        "critical_count": 0,
        "avg_tat_hours": "5.58"
      },
      {
        "label": "2026-28",
        "completed_tests": 8,
        "critical_count": 0,
        "avg_tat_hours": "94.14"
      }
    ],
    "total": 51.0
  }
}
```

---

### 10. Billing Mode Trends
* **Endpoint**: `GET /admin/analytics/billing`
* **Query Parameters**:
  * `granularity` (String, default: `monthly`): Interval grouping (`daily`, `weekly`, `monthly`, `yearly`).
  * `periods` (Integer, default: `12`): Number of history intervals to return.
* **Description**: Returns payment mode statistics detailing the number of transactions and total amount collected via Cash, Card, or UPI.
* **Sample Response**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "metric": "billing",
    "granularity": "monthly",
    "data": [
      {
        "label": "Jul 2026",
        "payment_mode": "CASH",
        "transactions": 11,
        "amount": "149050.00"
      },
      {
        "label": "Jul 2026",
        "payment_mode": "CARD",
        "transactions": 5,
        "amount": "22450.00"
      },
      {
        "label": "Jul 2026",
        "payment_mode": "UPI",
        "transactions": 5,
        "amount": "1040.00"
      }
    ],
    "total": 172540.0
  }
}
```

---

## 3. Operational Reports Reference

### 1. List Report Templates
* **Endpoint**: `GET /admin/reports`
* **Query Parameters**: None
* **Description**: Lists all available report templates registered within the reporting library category catalog.
* **Sample Response**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "report_type": "REVENUE_SUMMARY",
        "name": "Revenue Summary Report",
        "description": "Aggregated daily/monthly billing revenues and transaction totals grouped by payment mode.",
        "category": "FINANCE"
      },
      {
        "report_type": "INVENTORY_VALUATION",
        "name": "Inventory Master & Valuation Report",
        "description": "Stock valuation breakdown, low stock counts, critical alerts, and near-expiry metrics.",
        "category": "INVENTORY"
      },
      {
        "report_type": "CLINICAL_CENSUS",
        "name": "Hospital Clinical Census Report",
        "description": "Patient inflow counts, bed occupancy distributions, and emergency room statistics.",
        "category": "CLINICAL"
      },
      {
        "report_type": "STAFF_COMPLIANCE",
        "name": "Workforce & Compliance Overview Report",
        "description": "Active staff roster ratios, leave backlogs, and regulatory document upload status.",
        "category": "WORKFORCE"
      }
    ],
    "total": 4
  }
}
```

---

### 2. List Recent Generated Reports
* **Endpoint**: `GET /admin/reports/recent`
* **Query Parameters**:
  * `page` (Integer, default: `1`): Pagination page number.
  * `page_size` (Integer, default: `10`): Number of reports to load.
* **Description**: Returns a paginated list of historically generated reports, who generated them, generated dates, and their statuses.
* **Sample Response**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "id": "rep-001",
        "report_name": "OPD Summary Report",
        "department": "Outpatient",
        "reporting_period": "Jun 1 - Jun 30, 2025",
        "generated_by": "Dr. Priya Mehta",
        "generated_on": "2025-07-01 00:00:00+00:00",
        "status": "Pending"
      },
      {
        "id": "rep-002",
        "report_name": "IPD Admissions Report",
        "department": "Inpatient",
        "reporting_period": "Jun 1 - Jun 30, 2025",
        "generated_by": "Dr. Priya Mehta",
        "generated_on": "2025-07-01 00:00:00+00:00",
        "status": "Processing"
      },
      {
        "id": "rep-003",
        "report_name": "Emergency Case Summary",
        "department": "Emergency",
        "reporting_period": "Jun 1 - Jun 30, 2025",
        "generated_by": "Dr. Priya Mehta",
        "generated_on": "2025-07-01 00:00:00+00:00",
        "status": "Ready"
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

### 3. Generate Specific Report
* **Endpoint**: `GET /admin/reports/{report_type}`
* **Path Parameters**:
  * `report_type` (String, required): The template code key (e.g. `REVENUE_SUMMARY`, `INVENTORY_VALUATION`, `CLINICAL_CENSUS`, `STAFF_COMPLIANCE`).
* **Description**: Triggers a compilation request for a specific report template type, returning the generated document layout data structure.
* **Sample Response**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "report_type": "REVENUE_SUMMARY",
    "generated_at": "2026-07-23 10:29:53.403755+00:00",
    "summary": {
      "revenue_30d": 197528.5,
      "total_transactions": 21,
      "report_title": "Executive Revenue Summary"
    },
    "data": [
      {
        "label": "Jul 2026",
        "payment_mode": "CASH",
        "transactions": 11,
        "amount": 149050.0
      },
      {
        "label": "Jul 2026",
        "payment_mode": "CARD",
        "transactions": 5,
        "amount": 22450.0
      },
      {
        "label": "Jul 2026",
        "payment_mode": "UPI",
        "transactions": 5,
        "amount": 1040.0
      }
    ]
  }
}
```
