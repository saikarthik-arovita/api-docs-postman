# IPD Dashboard API Reference

This document defines the request and response contracts for the IPD Dashboard APIs represented in the **Ward & Bed grid dashboard** (Panel 1) of the UI.

---

## 1. Get Admission Dashboard Stats

Retrieves top-level operational statistics for the IPD ward occupancy, admissions, and discharges.

* **Endpoint**: `GET /ipd/dashboard`
* **Method**: `GET`
* **Headers**:
  - `Authorization: Bearer <token>`
* **Query Parameters**:
  - `target_date` (Optional, format: `YYYY-MM-DD`): Filters stats for a specific date (defaults to current date).

#### Response (200 OK)
```json
{
  "success": true,
  "data": {
    "tenant_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "date": "2026-07-03",
    "pending_requests": 5,
    "ready_for_admission": 3,
    "admissions_today": 8,
    "occupied_beds": 24,
    "available_beds": 12,
    "maintenance_beds": 2,
    "total_beds": 38,
    "occupancy_pct": 63.16,
    "elective_count": 5,
    "emergency_count": 3,
    "transfer_count": 1,
    "discharged_today": 4,
    "dama_today": 1,
    "deceased_today": 0
  }
}
```

---

## 2. Get Ward Occupancy Breakdown

Retrieves the listing and detail status metrics for each individual ward (such as capacity, occupied beds, and active state).

* **Endpoint**: `GET /ipd/dashboard/wards`
* **Method**: `GET`
* **Headers**:
  - `Authorization: Bearer <token>`

#### Response (200 OK)
```json
{
  "success": true,
  "data": {
    "wards": [
      {
        "id": "4100a692-6605-452d-baca-314892ba002d",
        "ward_code": "GW-01",
        "ward_name": "General Ward A",
        "ward_type": "GENERAL",
        "capacity": 15,
        "total_beds": 15,
        "occupied_beds": 10,
        "available_beds": 4,
        "maintenance_beds": 1,
        "is_active": true
      },
      {
        "id": "5200a782-1105-452d-baba-214892bb1102",
        "ward_code": "ICU-01",
        "ward_name": "Intensive Care Unit",
        "ward_type": "ICU",
        "capacity": 8,
        "total_beds": 8,
        "occupied_beds": 6,
        "available_beds": 2,
        "maintenance_beds": 0,
        "is_active": true
      }
    ],
    "total_beds": 23,
    "total_occupied": 16,
    "total_available": 6,
    "total_maintenance": 1,
    "overall_occupancy_pct": 69.57
  }
}
```

---

## 3. Get Patient Location Tracking

Retrieves a paginated list of all active patients currently admitted in the IPD, along with their assigned ward and bed details.

* **Endpoint**: `GET /ipd/dashboard/tracking`
* **Method**: `GET`
* **Headers**:
  - `Authorization: Bearer <token>`
* **Query Parameters**:
  - `ward_type` (Optional): Filter patients by ward category (`GENERAL`, `ICU`, `OBSERVATION`, `DAY_CARE`, etc.)
  - `page` (Optional): Page index (defaults to `1`)
  - `page_size` (Optional): Items per page (defaults to `50`)

#### Response (200 OK)
```json
{
  "success": true,
  "data": {
    "tenant_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "total_patients": 16,
    "ward_patients": 10,
    "icu_patients": 6,
    "ot_patients": 0,
    "observation_patients": 0,
    "day_care_patients": 0,
    "patients": [
      {
        "admission_id": "b453258f-23a7-4047-b316-ab5a655fd898",
        "admission_number": "IPD-2026-00042",
        "patient_id": "f5b8d234-c712-4eb3-81ef-8b26e55fc244",
        "patient_name": "Rajesh Kumar",
        "patient_uhid": "UHID123456",
        "ward_id": "4100a692-6605-452d-baca-314892ba002d",
        "ward_name": "General Ward A",
        "ward_type": "GENERAL",
        "bed_id": "3aa3856b-c80c-4ba5-9d5a-bfbbd4961a1f",
        "bed_number": "ER-02",
        "attending_doctor_name": "Dr. Supreeth",
        "ipd_status": "UNDER_TREATMENT",
        "admitted_at": "2026-07-03T10:15:30Z"
      }
    ],
    "page": 1,
    "page_size": 50,
    "total_pages": 1
  }
}
```
