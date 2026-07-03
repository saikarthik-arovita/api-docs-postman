# Bed Quick Actions API Reference

This document defines the request and response structures for the six backend APIs mapped to the actions available on the **Bed Information** modal UI.

---

## 1. Assign Beds

Assigns a patient to a bed. In the HMS, this is done when admitting a new inpatient or updating an emergency patient's bed assignment.

### Option A: Regular IPD Admission
* **Endpoint**: `POST /ipd/admissions`
* **Headers**:
  - `Content-Type: application/json`
  - `Authorization: Bearer <token>`

#### Request Body
```json
{
  "patient_id": "f5b8d234-c712-4eb3-81ef-8b26e55fc244",
  "ward_id": "4100a692-6605-452d-baca-314892ba002d",
  "bed_id": "3aa3856b-c80c-4ba5-9d5a-bfbbd4961a1f",
  "attending_doctor_id": "a90df234-601e-4cb8-9bde-452f38abf001",
  "admission_reason": "Severe pneumonia treatment",
  "admission_type": "ROUTINE",
  "expected_discharge": "2026-07-10",
  "notes": "Patient requires oxygen support"
}
```

#### Response (201 Created)
```json
{
  "success": true,
  "data": {
    "id": "b453258f-23a7-4047-b316-ab5a655fd898",
    "admission_number": "IPD-2026-00042",
    "patient_id": "f5b8d234-c712-4eb3-81ef-8b26e55fc244",
    "patient_name": "John Doe",
    "patient_uhid": "UHID123456",
    "ward_id": "4100a692-6605-452d-baca-314892ba002d",
    "ward_name": "General Ward",
    "bed_id": "3aa3856b-c80c-4ba5-9d5a-bfbbd4961a1f",
    "bed_number": "B-GEN-09",
    "attending_doctor_id": "a90df234-601e-4cb8-9bde-452f38abf001",
    "status": "ADMITTED",
    "ipd_status": "ADMITTED",
    "admitted_at": "2026-07-03T17:03:00Z"
  }
}
```

### Option B: Emergency Visit Bed Assignment
* **Endpoint**: `PATCH /ipd/emergency/{visit_id}/bed`
* **Headers**:
  - `Content-Type: application/json`
  - `Authorization: Bearer <token>`

#### Request Body
```json
{
  "bed_id": "3aa3856b-c80c-4ba5-9d5a-bfbbd4961a1f"
}
```

#### Response (200 OK)
```json
{
  "success": true,
  "message": "Emergency bed assignment updated",
  "data": {
    "visit_id": "e20fa712-192a-43b8-8902-7c4278df2582",
    "bed_id": "3aa3856b-c80c-4ba5-9d5a-bfbbd4961a1f"
  }
}
```

---

## 2. Release Bed

Removes any active admission occupying the bed, freeing it up and updating its status to `AVAILABLE`.

* **Endpoint**: `POST /ipd/beds/{bed_id}/release`
* **Headers**:
  - `Authorization: Bearer <token>`

#### Request Body
*None (Empty)*

#### Response (200 OK)
```json
{
  "success": true,
  "data": {
    "bed_id": "3aa3856b-c80c-4ba5-9d5a-bfbbd4961a1f",
    "status": "AVAILABLE"
  }
}
```

---

## 3. Admission History

Retrieves the list of all historical and active patient admissions/occupations for a given bed.

* **Endpoint**: `GET /ipd/admissions?bed_id={bed_id}`
* **Headers**:
  - `Authorization: Bearer <token>`

#### Query Parameters
| Parameter | Type | Required | Description |
| :--- | :--- | :---: | :--- |
| `bed_id` | UUID String | **Yes** | The UUID of the bed to query |
| `status` | String | No | Filter by active/inactive state (`ADMITTED`, `DISCHARGED`) |
| `page` | Integer | No | Page index (defaults to `1`) |
| `page_size` | Integer | No | Items per page (defaults to `20`) |

#### Response (200 OK)
```json
{
  "success": true,
  "data": {
    "admissions": [
      {
        "id": "b453258f-23a7-4047-b316-ab5a655fd898",
        "admission_number": "IPD-2026-00042",
        "patient_id": "f5b8d234-c712-4eb3-81ef-8b26e55fc244",
        "patient_name": "John Doe",
        "patient_uhid": "UHID123456",
        "ward_id": "4100a692-6605-452d-baca-314892ba002d",
        "ward_name": "General Ward",
        "bed_id": "3aa3856b-c80c-4ba5-9d5a-bfbbd4961a1f",
        "bed_number": "B-GEN-09",
        "attending_doctor_id": "a90df234-601e-4cb8-9bde-452f38abf001",
        "status": "ADMITTED",
        "ipd_status": "ADMITTED",
        "admitted_at": "2026-07-03T17:03:00Z"
      }
    ],
    "total": 1,
    "page": 1,
    "page_size": 20,
    "total_pages": 1
  }
}
```

---

## 4. Edit Bed Assignment

Modifies metadata and details of an active bed assignment/admission (such as attending doctor, notes, expected discharge date, etc.).

* **Endpoint**: `PATCH /ipd/admissions/{id}`
* **Headers**:
  - `Content-Type: application/json`
  - `Authorization: Bearer <token>`

#### Request Body
*(All fields are optional)*
```json
{
  "attending_doctor_id": "a90df234-601e-4cb8-9bde-452f38abf001",
  "expected_discharge": "2026-07-15",
  "notes": "Patient shows signs of recovery",
  "admission_reason": "Observation post-surgery"
}
```

#### Response (200 OK)
```json
{
  "success": true,
  "data": {
    "id": "b453258f-23a7-4047-b316-ab5a655fd898",
    "admission_number": "IPD-2026-00042",
    "patient_id": "f5b8d234-c712-4eb3-81ef-8b26e55fc244",
    "ward_id": "4100a692-6605-452d-baca-314892ba002d",
    "bed_id": "3aa3856b-c80c-4ba5-9d5a-bfbbd4961a1f",
    "attending_doctor_id": "a90df234-601e-4cb8-9bde-452f38abf001",
    "admission_type": "ROUTINE",
    "admission_reason": "Observation post-surgery",
    "expected_discharge": "2026-07-15",
    "notes": "Patient shows signs of recovery",
    "status": "ADMITTED",
    "ipd_status": "ADMITTED"
  }
}
```

---

## 5. Transfer Beds

Transfers an admitted patient from their current bed to another bed, automatically updating the status of both beds and creating a transfer history log.

* **Endpoint**: `POST /ipd/admissions/{id}/transfer`
* **Headers**:
  - `Content-Type: application/json`
  - `Authorization: Bearer <token>`

#### Request Body
```json
{
  "to_ward_id": "5200a782-1105-452d-baba-214892bb1102",
  "to_bed_id": "4f6dce70-b45c-4509-bab0-1cecc3f119f0",
  "transfer_reason": "Condition deteriorated, requires ICU monitoring",
  "transfer_type": "INTERNAL",
  "responsible_doctor_id": "a90df234-601e-4cb8-9bde-452f38abf001",
  "notes": "Oxygen level dropped to 88%"
}
```

#### Response (200 OK)
```json
{
  "success": true,
  "data": {
    "id": "645a8ddd-b0c9-4904-b0e9-ee1062406165",
    "admission_id": "b453258f-23a7-4047-b316-ab5a655fd898",
    "from_ward_id": "4100a692-6605-452d-baca-314892ba002d",
    "from_bed_id": "3aa3856b-c80c-4ba5-9d5a-bfbbd4961a1f",
    "to_ward_id": "5200a782-1105-452d-baba-214892bb1102",
    "to_bed_id": "4f6dce70-b45c-4509-bab0-1cecc3f119f0",
    "transfer_reason": "Condition deteriorated, requires ICU monitoring",
    "transfer_type": "INTERNAL",
    "transferred_by": "fcf18321-5f85-4a9c-817f-a5392e612a34",
    "transferred_at": "2026-07-03T17:05:00Z"
  }
}
```

---

## 6. Mark Maintenance

Updates a bed status directly to `MAINTENANCE` (or returns it to availability).

* **Endpoint**: `PATCH /ipd/beds/{bed_id}/status` (or `/admin/beds/{bed_id}/status` for Admin portal)
* **Headers**:
  - `Content-Type: application/json`
  - `Authorization: Bearer <token>`

#### Request Body
```json
{
  "status": "MAINTENANCE",
  "reason": "Broken lock mechanism on side railing"
}
```

#### Response (200 OK)
```json
{
  "success": true,
  "data": {
    "bed_id": "3aa3856b-c80c-4ba5-9d5a-bfbbd4961a1f",
    "status": "MAINTENANCE"
  }
}
```
