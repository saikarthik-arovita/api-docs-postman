# HMS — KPI Dashboard Screenshot to API Mappings

This document provides a systematic mapping of all KPI card screenshots in `kpis.docx` to the specific API endpoints in the HMS codebase that generate these values.

---

## 1. Global Conventions
All dashboard and analytics endpoints require authentication:
```http
Authorization: Bearer <access_token>
```
Response payloads follow the standard success envelope:
```json
{
  "success": true,
  "data": { ... }
}
```

---

## 2. KPI Mappings by Screenshot

### image1.png — Bed Management Dashboard
* **Screen Context:** Dashboard > Bed Management
* **Metrics Shown:**
  * Total Beds: 1,240 (Total configured beds)
  * Occupied Beds: 892 (Current Utilization, +1.2%)
  * Available Beds: 248 (Ready for admission, -2.1%)
  * Reserved Beds: 64 (Next 24 hours, +0.5%)
  * Under Maintenance: 36 (Temporary blocked beds)
  * Occupancy Rate: 71.9%
* **API Endpoint:** `GET /ipd/beds/availability`
* **Service:** IPD Service
* **Required Permission:** `ipd:view`
* **Response Sample:**
```json
{
  "success": true,
  "data": {
    "total": 1240,
    "occupied": 892,
    "available": 248,
    "reserved": 64,
    "maintenance": 36,
    "occupancy_rate_pct": 71.9
  }
}
```

---

### image2.png — Ward Management Dashboard
* **Screen Context:** Dashboard > Ward Management
* **Metrics Shown:**
  * Total Wards: 24
  * Total Beds: 860
  * Occupied Wards/Beds: 741
  * Available Wards/Beds: 163
  * Occupancy Rate: 73%
  * Critical Cases: 2
  * Discharge Today: 91%
  * Bed Turnover: 74%
* **API Endpoint:** `GET /ipd/dashboard/wards`
* **Service:** IPD Service
* **Required Permission:** `dashboard:view`
* **Response Sample:**
```json
{
  "success": true,
  "data": {
    "total_wards": 24,
    "total_beds": 860,
    "occupied_beds": 741,
    "available_beds": 163,
    "occupancy_rate_pct": 73.0,
    "critical_cases_count": 2,
    "discharge_today_pct": 91.0,
    "bed_turnover_pct": 74.0
  }
}
```

---

### image3.png — Staff Management Dashboard (Roster Overview)
* **Screen Context:** Dashboard > Staff Management
* **Metrics Shown:**
  * Total Staff: 140 (+13 this month)
  * Doctors Available: 45 (+5 this month)
  * Nurses on Duty: 32 (+3 this month)
  * Technicians: 150 (+2 this month)
  * Administrative Staff: 50 (+2 this month)
  * Support Staff: 20 (+1 this month)
* **API Endpoint:** `GET /admin/dashboard/summary`
* **Service:** Admin Service
* **Required Permission:** `dashboard:view`
* **Response Sample:**
```json
{
  "success": true,
  "data": {
    "total_staff": 140,
    "active_staff": 131,
    "staff_metrics": {
      "doctors_available": 45,
      "nurses_on_duty": 32,
      "technicians_on_duty": 150,
      "admin_staff": 50,
      "support_staff": 20
    }
  }
}
```

---

### image4.png — Dashboard Overview (Global Command Center)
* **Screen Context:** Dashboard > Dashboard Overview
* **Metrics Shown:**
  * OPD Today: 248 (Live) [Completed: 192, Waiting: 56]
  * IPD Census: 142 [Available Beds: 38, Occupied Beds: 104]
  * Emergency: 24 (4 Critical) [Waiting Triage: 6, Critical: 4]
  * Revenue: ₹2.8L (Today) [Monthly: ₹7.8L, Outstanding: ₹4.2L]
  * Staff Available: 131 (Today) [Doctors: 42, Nurses: 68, Technicians: 21]
  * Bed Occupancy: 73% (Moderate) [Occupied: 104, Available: 38]
* **API Endpoint:** `GET /admin/dashboard`
* **Service:** Admin Service
* **Required Permission:** `dashboard:view`
* **Response Sample:**
```json
{
  "success": true,
  "data": {
    "patient_metrics": {
      "opd_total": 248,
      "opd_completed": 192,
      "opd_waiting": 56,
      "ipd_total": 142,
      "active_admissions": 104,
      "emergency_today": 24,
      "emergency_critical": 4,
      "emergency_waiting_triage": 6
    },
    "financial_metrics": {
      "today_revenue": 280000.00,
      "monthly_revenue": 780000.00,
      "pending_dues": 420000.00
    },
    "staff_metrics": {
      "active_staff": 131,
      "doctors_available": 42,
      "nurses_on_duty": 68,
      "technicians_on_duty": 21
    },
    "ops_metrics": {
      "total_beds": 142,
      "occupied_beds": 104,
      "available_beds": 38,
      "bed_occupancy_pct": 73.0
    }
  }
}
```

---

### image5.png — Patient Overview Analytics
* **Screen Context:** Dashboard > Patient Overview
* **Metrics Shown:**
  * Total Patients: 1,842 (12% vs last week)
  * New Registrations: 364 (20% vs last week)
  * Repeat Patients: 1,478 (60% of total)
  * Active Patients: 986 (Currently on premises)
  * Pending Discharges: 43 (Avg wait: 1.4 hrs)
  * Average Daily Footfall: 1,710 (Last 7 days)
* **API Endpoint:** `GET /admin/dashboard` (specifically checking the `patient_metrics` section)
* **Service:** Admin Service
* **Required Permission:** `dashboard:view`
* **Response Sample:**
```json
{
  "success": true,
  "data": {
    "patient_metrics": {
      "total_patients": 1842,
      "new_today": 364,
      "repeat_patients": 1478,
      "active_patients": 986,
      "discharge_pending": 43,
      "avg_discharge_wait_hours": 1.4,
      "avg_daily_footfall": 1710
    }
  }
}
```

---

### image6.png — OPD Management Dashboard
* **Screen Context:** Dashboard > OPD Management
* **Metrics Shown:**
  * Total OPD Patients: 934 (+6.3% vs yesterday)
  * Waiting Patients: 127 (Queue building)
  * Completed: 689 (73.8% completion)
  * Avg. Waiting Time: 28m (+4 min vs target)
  * Revenue Generated: ₹4.2L (₹3.8L target)
* **API Endpoint:** `GET /opd/dashboard`
* **Service:** OPD Service
* **Required Permission:** `dashboard:view`
* **Response Sample:**
```json
{
  "success": true,
  "data": {
    "total_appointments": 934,
    "scheduled": 127,
    "completed": 689,
    "avg_waiting_time_minutes": 28,
    "revenue_generated": 420000.00
  }
}
```

---

### image7.png — IPD (Inpatients) Management
* **Screen Context:** Dashboard > IPD (Inpatients) Management
* **Metrics Shown:**
  * Admissions Today: 12 (12% vs last week)
  * Transfers: 24 (Cross-ward)
  * Discharges Today: 61 (On schedule)
  * Bed Occupancy Rate: 83.5%
  * Avg Length of Stay: 4.2 days (Target: 3.8 days)
  * Ward Occupancy breakdown (ICU, Emergency, Semi-Private, Private, General)
* **API Endpoint:** `GET /ipd/dashboard` (Admissions Stats) and `GET /ipd/dashboard/wards` (Ward occupancy details)
* **Service:** IPD Service
* **Required Permission:** `dashboard:view`
* **Response Sample (`GET /ipd/dashboard`):**
```json
{
  "success": true,
  "data": {
    "total_admitted_today": 12,
    "transfers_today": 24,
    "pending_discharge_today": 61,
    "bed_occupancy_rate_pct": 83.5,
    "avg_length_of_stay_days": 4.2,
    "wards": [
      { "ward_name": "ICU Ward", "occupied_beds": 8, "total_beds": 10, "occupancy_rate_pct": 80 },
      { "ward_name": "Emergency", "occupied_beds": 5, "total_beds": 10, "occupancy_rate_pct": 50 },
      { "ward_name": "Semi-Private", "occupied_beds": 15, "total_beds": 20, "occupancy_rate_pct": 75 },
      { "ward_name": "Private Rooms", "occupied_beds": 4, "total_beds": 10, "occupancy_rate_pct": 40 },
      { "ward_name": "General", "occupied_beds": 15, "total_beds": 20, "occupancy_rate_pct": 75 }
    ]
  }
}
```

---

### image8.png — Emergency (ED) Management Dashboard
* **Screen Context:** Dashboard > Emergency (ED) Management
* **Metrics Shown:**
  * Emergency Department Status: Operating at Level 2 Capacity
  * Active Cases: 24 (Severely critical: 4)
  * Procedures Required: 3
  * Active Emergency Registrations: 118 (+16 vs yesterday)
  * Active Cases: 67
  * Critical Cases: 14
  * Waiting Triage: 22 (Target: <15)
* **API Endpoint:** `GET /opd/emergency`
* **Service:** OPD/Clinical Service
* **Required Permission:** `dashboard:view`
* **Response Sample:**
```json
{
  "success": true,
  "data": {
    "status": "Level 2 Capacity",
    "active_cases": 24,
    "severe_cases": 4,
    "procedures_required": 3,
    "active_registrations": 118,
    "total_active_cases": 67,
    "critical_cases": 14,
    "waiting_triage": 22
  }
}
```

---

### image9.png — Procedures & OT (Operating Theatre) Management
* **Screen Context:** Dashboard > Procedures / OT Management
* **Metrics Shown:**
  * Scheduled Procedures: 32 (For today)
  * Completed: 4 (65.6% completion - Note: mock text percentage mismatch)
  * In Progress: 3 (+2% vs last week)
  * Delayed Procedures: 4 (Review required)
  * OT Utilization: 79.2% (Target: 85%)
* **API Endpoint:** `GET /admin/analytics/ot`
* **Service:** Admin/Clinical Service
* **Required Permission:** `dashboard:view`
* **Response Sample:**
```json
{
  "success": true,
  "data": {
    "scheduled_today": 32,
    "completed_today": 4,
    "in_progress_today": 3,
    "delayed_today": 4,
    "ot_utilization_pct": 79.2
  }
}
```

---

### image10.png — Laboratory Management Dashboard
* **Screen Context:** Dashboard > Laboratory Management
* **Metrics Shown:**
  * Total Tests Today: 428 (+14% vs yesterday)
  * Pending Tests: 64
  * Completed Tests: 332
  * Department Volume: Proctology: 46 (Critical: 5), Gynaecology: 86 (Critical: 2)
  * Avg Turnaround Time (TAT): 38m (8% faster)
  * Machine Sync Status: 98%
  * Critical Results: 8
  * Active Technicians: 14
  * Delayed Reports: 12
* **API Endpoint:** `GET /diagnostic-orders/lab/dashboard`
* **Service:** Diagnostics (Lab) Service
* **Required Permission:** None (Authenticated)
* **Response Sample:**
```json
{
  "success": true,
  "data": {
    "pending_samples": 64,
    "completed": 332,
    "total_results": 428,
    "critical_results": 8,
    "avg_tat_minutes": 38,
    "machine_sync_pct": 98,
    "active_technicians": 14,
    "delayed_reports": 12,
    "departments": [
      { "name": "Proctology", "total_tests": 46, "critical_findings": 5 },
      { "name": "Gynaecology", "total_tests": 86, "critical_findings": 2 }
    ]
  }
}
```

---

### image11.png — Radiology Operations Center
* **Screen Context:** Dashboard > Radiology Operations Center
* **Metrics Shown:**
  * MRI Machine (MR-1): 92% (In Use)
  * CT Scanner (CT-1): 78% (Next Patient)
  * X-Ray Unit (XR-2): 64% (Available)
  * Ultrasound (USG-1): Offline (Since 8:30 AM)
* **API Endpoint:** `GET /admin/analytics/ot` (which fetches diagnostic machinery metrics) or a dedicated machine status query
* **Service:** Diagnostics (Radiology) Service
* **Required Permission:** `dashboard:view`
* **Response Sample:**
```json
{
  "success": true,
  "data": {
    "equipment": [
      { "id": "MR-1", "name": "MRI Machine", "status": "IN_USE", "utilization_pct": 92.0, "maintenance_in_days": 12 },
      { "id": "CT-1", "name": "CT Scanner", "status": "NEXT_PATIENT", "utilization_pct": 78.0, "wait_time_minutes": 12 },
      { "id": "XR-2", "name": "X-Ray Unit", "status": "AVAILABLE", "utilization_pct": 64.0, "queued_patients": 3 },
      { "id": "USG-1", "name": "Ultrasound", "status": "OFFLINE", "utilization_pct": 0, "offline_since": "08:30 AM" }
    ]
  }
}
```

---

### image12.png — Pharmacy Dashboard
* **Screen Context:** Dashboard > Pharmacy
* **Metrics Shown:**
  * Total Prescriptions Today: 214 (+12%) [OPD: 162, IPD: 52]
  * Prescription Returns: 18 [Return Rate: 8.4%]
  * OPD to IPD Conversion: 18% (+4%)
  * Prescription Lifecycle: Verified: 198, Dispensed: 184, Pending: 25
  * Fast-Moving Medicines: Daflon, Lidocaine, Sitz Bath Kit
  * Operational Efficiency: 4.2 mins (-18s)
* **API Endpoint:** `GET /pharmacy/dashboard/summary`
* **Service:** Pharmacy Service
* **Required Permission:** `dashboard:view`
* **Response Sample:**
```json
{
  "success": true,
  "data": {
    "prescriptions_today": 214,
    "opd_prescriptions": 162,
    "ipd_prescriptions": 52,
    "returns_count": 18,
    "return_rate_pct": 8.4,
    "opd_to_ipd_conversion_pct": 18.0,
    "verified_count": 198,
    "dispensed_today": 184,
    "pending_dispense": 25,
    "avg_dispense_time_minutes": 4.2,
    "fast_moving_items": [
      { "item_name": "Daflon 500mg", "stock_status": "LOW" },
      { "item_name": "Lidocaine Gel", "stock_status": "STABLE" },
      { "item_name": "Sitz Bath Kit", "stock_status": "REORDER" }
    ]
  }
}
```

---

### image13.png — Pharmacy & Medical Inventory Management
* **Screen Context:** Dashboard > Inventory Management
* **Metrics Shown:**
  * Total Inventory Value: ₹24,86,450 (+18.4% vs last month)
  * Total Stock Items: 98,340 (+11,240 added this month)
  * Low Stock Items: 26 (8 Critical items)
  * Near Expiry Items: 142 (Expiring within 30 days)
* **API Endpoint:** `GET /admin/inventory?item_type=medical`
* **Service:** Admin Service
* **Required Permission:** `dashboard:view`
* **Response Sample:**
```json
{
  "success": true,
  "data": {
    "total_value": 2486450.00,
    "total_stock_items": 98340,
    "low_stock_count": 26,
    "critical_stock_count": 8,
    "near_expiry_count": 142
  }
}
```

---

### image14.png — Purchase & Procurement Dashboard
* **Screen Context:** Dashboard > Purchase & Procurement
* **Metrics Shown:**
  * Total Purchase Orders: 1,284 (+12.4% vs last month)
  * Procurement Spend Value: ₹48,36,500 (+8.2% vs last month)
  * Pending Purchase Orders: 87 (14 Overdue)
  * Active Suppliers: 134 (+4.5% vs last month)
* **API Endpoint:** `GET /admin/dashboard` (sub-section `procurement_metrics` / `ops_metrics`)
* **Service:** Admin Service
* **Required Permission:** `dashboard:view`
* **Response Sample:**
```json
{
  "success": true,
  "data": {
    "total_pos": 1284,
    "procurement_spend": 4836500.00,
    "pending_pos": 87,
    "overdue_pos": 14,
    "active_suppliers": 134
  }
}
```

---

### image15.png — Vendor & Supply Management
* **Screen Context:** Dashboard > Vendor Management
* **Metrics Shown:**
  * Total Vendors: 148 (+4%)
  * Active Vendors: 124
  * Pending Approvals: 12
  * Expiring Contracts: 9
  * Delayed Deliveries: 17
  * Avg Vendor Rating: 4.5
* **API Endpoint:** `GET /admin/tpa/providers` (lists vendors, TPAs, and suppliers metrics)
* **Service:** Admin Service
* **Required Permission:** `dashboard:view`
* **Response Sample:**
```json
{
  "success": true,
  "data": {
    "total_vendors": 148,
    "active_vendors": 124,
    "pending_approvals": 12,
    "expiring_contracts": 9,
    "delayed_deliveries": 17,
    "avg_vendor_rating": 4.5
  }
}
```

---

### image16.png — Billing & Payment Collection
* **Screen Context:** Dashboard > Billing & Payment
* **Metrics Shown:**
  * Today's Collection: ₹4,25,000 (+12% vs yesterday)
  * Outstanding Dues: ₹84,500 (-4% vs yesterday)
  * TPA Claims Pending: 18 (₹3.2L in pending claims)
  * Refunds Processed: ₹12,400 (4 refunds today)
* **API Endpoint:** `GET /admin/dashboard` (specifically checking `financial_metrics`)
* **Service:** Admin Service
* **Required Permission:** `dashboard:view`
* **Response Sample:**
```json
{
  "success": true,
  "data": {
    "financial_metrics": {
      "today_revenue": 425000.00,
      "pending_dues": 84500.00,
      "pending_tpa_claims": 18,
      "pending_tpa_claims_value": 320000.00,
      "refunds_today_amount": 12400.00,
      "refunds_today_count": 4
    }
  }
}
```

---

### image17.png — Revenue Monitoring Dashboard
* **Screen Context:** Dashboard > Revenue Monitoring
* **Metrics Shown:**
  * Today's Revenue: ₹4,25,000 (+12% vs yesterday)
  * Monthly Revenue: ₹84,500 [Target: 75,000, 82% - Mock numbers inconsistent]
  * Highest Revenue Dept.: Radiology (Total dept. revenue: ₹4,25,000)
  * Top Earning Service: MRI Scan (Total service revenue: ₹1,43,600)
* **API Endpoint:** `GET /admin/analytics/revenue?granularity=monthly`
* **Service:** Admin Service
* **Required Permission:** `dashboard:view`
* **Response Sample:**
```json
{
  "success": true,
  "data": {
    "today_revenue": 425000.00,
    "monthly_revenue": 84500.00,
    "target_revenue": 75000.00,
    "highest_revenue_department": "Radiology",
    "highest_department_revenue": 425000.00,
    "top_earning_service": "MRI Scan",
    "top_service_revenue": 143600.00
  }
}
```

---

### image18.png — Staff Management (Roster Overview)
* **Screen Context:** Dashboard > Staff Management
* **Metrics Shown:** Same as image3.png (Staff Management statistics).
* **API Endpoint:** `GET /admin/dashboard/summary`
* **Service:** Admin Service
* **Required Permission:** `dashboard:view`
* **Response Sample:** Identical to image3.png.

---

### image19.png — Hospital Administration
* **Screen Context:** Dashboard > Hospital Administration
* **Metrics Shown:**
  * Summary: Floors: 4, Wards: 12, Rooms: 28, Beds: 785, Occupied: 744, Available: 213 (Inconsistent layout statistics)
  * Floor-wise Bed Free/Busy Count
* **API Endpoint:** `GET /ipd/beds/availability`
* **Service:** IPD Service
* **Required Permission:** `ipd:view`
* **Response Sample:**
```json
{
  "success": true,
  "data": {
    "total_floors": 4,
    "total_wards": 12,
    "total_rooms": 28,
    "total_beds": 785,
    "occupied": 744,
    "available": 213,
    "floors": [
      { "floor": "Ground Floor", "available_beds": 4, "occupied_beds": 2 },
      { "floor": "First Floor", "available_beds": 2, "occupied_beds": 3 },
      { "floor": "Second Floor", "available_beds": 3, "occupied_beds": 1 },
      { "floor": "Third Floor", "available_beds": 3, "occupied_beds": 3 }
    ]
  }
}
```

---

### image20.png — Ground Floor Administration Details
* **Screen Context:** Dashboard > Hospital Administration > Ground Floor Detail
* **Metrics Shown:**
  * Reception: Available
  * Emergency Room (ER): Occupied (8/10 beds, 80%)
  * Radiology: Available (2/4 beds, 50%)
  * Triage: Occupied (5/6 beds, 83%)
  * Lab: Available
  * Pharmacy: Available
  * Summary: 4 Available, 2 Occupied, 0 Maintenance [Total Patients: 15]
* **API Endpoint:** `GET /ipd/dashboard/tracking`
* **Service:** IPD Service
* **Required Permission:** `dashboard:view`
* **Response Sample:**
```json
{
  "success": true,
  "data": {
    "ground_floor_metrics": {
      "reception": "AVAILABLE",
      "er": { "status": "OCCUPIED", "occupied_beds": 8, "total_beds": 10, "occupancy_rate_pct": 80 },
      "radiology": { "status": "AVAILABLE", "occupied_beds": 2, "total_beds": 4, "occupancy_rate_pct": 50 },
      "triage": { "status": "OCCUPIED", "occupied_beds": 5, "total_beds": 6, "occupancy_rate_pct": 83 },
      "lab": "AVAILABLE",
      "pharmacy": "AVAILABLE",
      "available_units": 4,
      "occupied_units": 2,
      "maintenance_units": 0,
      "total_patients": 15
    }
  }
}
```

---

### image21.png — Analytics & Operational Reports
* **Screen Context:** Dashboard > Analytics & Reports
* **Metrics Shown:**
  * Total Patients: 1,248 (+8% vs last week)
  * Emergency Cases: 42 (+12% vs last week)
  * Bed Occupancy: 85% (+2% vs last week)
  * Procedures Performed: 18 (-1% vs last week)
  * Total Revenue: ₹8.4L (+10% vs last week)
  * Pending Payments: ₹1.2L (-5% vs last week)
  * Conversion Rate: 18% (+1% vs last week)
* **API Endpoint:** `GET /admin/analytics/kpi`
* **Service:** Admin Service
* **Required Permission:** `dashboard:view`
* **Response Sample:**
```json
{
  "success": true,
  "data": {
    "total_patients": 1248,
    "emergency_active_today": 42,
    "bed_occupancy_pct": 85.0,
    "procedures_performed_today": 18,
    "revenue_30d": 840000.00,
    "pending_payments_value": 120000.00,
    "conversion_rate_pct": 18.0
  }
}
```

---

### image22.png — Admin Directory & Hospital Overview
* **Screen Context:** Dashboard > Administration > Overview
* **Metrics Shown:**
  * Total Departments: 18 (+2 this quarter)
  * Total Specialties: 42 (+5 this month)
  * Active Hospital Services: 134 (No change)
  * Scheduled Today: 287 (94% coverage)
  * Staff on Leave Today: 23 (+4 vs yesterday)
  * Reports Generated: 76 (+12% vs last week)
* **API Endpoint:** `GET /admin/dashboard/summary`
* **Service:** Admin Service
* **Required Permission:** `dashboard:view`
* **Response Sample:**
```json
{
  "success": true,
  "data": {
    "total_departments": 18,
    "total_specialties": 42,
    "total_services": 134,
    "scheduled_roster_count": 287,
    "roster_coverage_pct": 94.0,
    "staff_on_leave_count": 23,
    "reports_generated_count": 76
  }
}
```

---

### image23.png — Medical Specialty Management
* **Screen Context:** Dashboard > Administration > Specialty Management
* **Metrics Shown:**
  * Total Specialties: 42
  * Active Specialties: 38
  * Total Consultants: 186
  * Avg. Patients / Specialty: 24.3
* **API Endpoint:** `GET /admin/specializations`
* **Service:** Admin Service
* **Required Permission:** None (Authenticated)
* **Response Sample:**
```json
{
  "success": true,
  "data": {
    "total_specialties": 42,
    "active_specialties": 38,
    "total_consultants": 186,
    "avg_patients_per_specialty": 24.3
  }
}
```

---

### image24.png — Service Master Management
* **Screen Context:** Dashboard > Administration > Service Management
* **Metrics Shown:**
  * Total Services: 134
  * Active Services: 118
  * Revenue This Month: ₹8.4L (+12% vs last month)
  * Avg. Service Price: ₹1,240
* **API Endpoint:** `GET /admin/analytics/revenue` (Specifically fetching hospital billing configuration service summaries)
* **Service:** Admin Service
* **Required Permission:** `dashboard:view`
* **Response Sample:**
```json
{
  "success": true,
  "data": {
    "total_services": 134,
    "active_services": 118,
    "services_revenue_monthly": 840000.00,
    "avg_service_price": 1240.00
  }
}
```

---

### image25.png — Roster Scheduling & Shifts
* **Screen Context:** Dashboard > Administration > Duty Roster
* **Metrics Shown:**
  * Morning Shift: 94 [08:00 - 14:00]
  * Evening Shift: 78 [14:00 - 22:00]
  * Night Shift: 52 [22:00 - 08:00]
  * On-call Staff: 18
  * Open Shifts: 7
  * Coverage %: 94%
* **API Endpoint:** `GET /admin/staff` (specifically roster groupings)
* **Service:** Admin Service
* **Required Permission:** `dashboard:view`
* **Response Sample:**
```json
{
  "success": true,
  "data": {
    "shifts": {
      "morning_shift_count": 94,
      "evening_shift_count": 78,
      "night_shift_count": 52,
      "on_call_count": 18,
      "open_shifts_count": 7,
      "coverage_pct": 94.0
    }
  }
}
```
