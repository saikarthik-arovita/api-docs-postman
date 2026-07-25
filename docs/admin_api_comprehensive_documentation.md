# HMS Admin Service: Universal API Documentation Reference

This document provides exhaustive, production-grade API specifications for the entire `hms-admin` microservice. It covers every endpoint, HTTP method, query filter parameter, request body, and JSON response payload.

---

## Global Headers & Environment
* **Base URL**: `https://vo585bjv4a.execute-api.ap-south-1.amazonaws.com/dev`
* **Authorization**: `Bearer <Cognito_Access_Token>`
* **Branch Isolation**: Uses active user session or override via `X-Branch-ID: <UUID>` header.

---

## 1. Hospital Profile & Settings APIs

### 1.1 GET Hospital Profile
Retrieve the facility profile configuration details.

* **Method**: `GET`
* **URL**: `/admin/hospital-profile`

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "589b018a-f8ed-4388-b49e-cfe46fb2d3f8",
    "organization_name": "Sai LJB Healthcare",
    "hospital_code": "SAI-LJB",
    "official_email": "admin@sailjbcare.com",
    "primary_phone": "+91 98765 43210",
    "address_line1": "123, Hospital Road, Sector 18",
    "city": "New Delhi",
    "state": "Delhi",
    "country": "India",
    "postal_code": "110001",
    "time_zone": "Asia/Kolkata",
    "currency": "INR (₹)",
    "is_active": true
  }
}
```

---

### 1.2 POST Create Hospital Profile
Initialize hospital profile specifications.

* **Method**: `POST`
* **URL**: `/admin/hospital-profile`

#### Request Body (`HospitalProfileCreateRequest`)
```json
{
  "organization_name": "Sai LJB Healthcare",
  "hospital_code": "SAI-LJB",
  "official_email": "admin@sailjbcare.com",
  "primary_phone": "+91 98765 43210",
  "address_line1": "123, Hospital Road",
  "city": "New Delhi",
  "state": "Delhi",
  "country": "India",
  "postal_code": "110001"
}
```

#### Response (`201 Created`)
*Matches GET structure with a success code of `201`.*

---

### 1.3 GET List Settings
List configuration settings.

* **Method**: `GET`
* **URL**: `/admin/settings`

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": [
    { "id": "s1-aa30b4b7", "category": "GENERAL", "setting_key": "enable_mfa", "setting_value": "true" }
  ]
}
```

---

### 1.4 POST Upsert Setting
Add or update a configuration key-value pair.

* **Method**: `POST`
* **URL**: `/admin/settings`

#### Request Body (`SettingUpsertRequest`)
```json
{
  "category": "GENERAL",
  "setting_key": "enable_mfa",
  "setting_value": "true"
}
```

#### Response (`200 OK`)
*Returns the updated SettingItem object.*

---

## 2. Dashboard & Performance Analytics

### Global Query Filter Parameters (Dashboard & Analytics)
* `hospitalId` (UUID, Optional) — Filter metrics by specific hospital.
* `branchId` (UUID, Optional) — Filter metrics by branch.
* `departmentId` (UUID, Optional) — Filter metrics by clinical department.
* `fromDate` / `toDate` (String: `YYYY-MM-DD`, Optional) — Filter by custom date ranges.
* `timeRange` (String, Optional) — Pre-defined filter ranges (`"today"`, `"this-week"`, etc.).

---

### 2.1 GET Dashboard Summary
Retrieve high-level counts.

* **Method**: `GET`
* **URL**: `/admin/dashboard/summary`
* **Query Parameters**: *Global query filters*

#### Response (`200 OK`)
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

### 2.2 GET Dashboard Overview
Outpatient vs inpatient daily flow counts.

* **Method**: `GET`
* **URL**: `/admin/dashboard/overview`
* **Query Parameters**: *Global query filters*

#### Response (`200 OK`)
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

### 2.3 GET Hospital Performance
Retrieve operational metrics.

* **Method**: `GET`
* **URL**: `/admin/dashboard/hospital-performance`
* **Query Parameters**: *Global query filters*

#### Response (`200 OK`)
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

### 2.4 GET Resource Utilization
Retrieve active usage metrics for Operating Theatres and ICU Beds.

* **Method**: `GET`
* **URL**: `/admin/dashboard/resource-utilization`
* **Query Parameters**: *Global query filters*

#### Response (`200 OK`)
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

### 2.5 GET Dashboard Reports list
Retrieve recently generated clinical or inventory reports metadata.

* **Method**: `GET`
* **URL**: `/admin/dashboard/reports`
* **Query Parameters**:
  * `page` (*Optional*, Default: `1`): Current page integer.
  * `page_size` (*Optional*, Default: `10`): Items limit.
  * `search` (*Optional*): Filter by report name.
  * `status` (*Optional*): Filter by status (`SUCCESS`, `PENDING`, `FAILED`).

#### Response (`200 OK`)
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

## 3. Staff Profile & Permissions Management

### 3.1 GET List Staff
Retrieve a list of all active staff members registered at the branch.

* **Method**: `GET`
* **URL**: `/admin/staff`
* **Query Parameters**:
  * `page` (*Optional*, Default: `1`): Current page offset.
  * `page_size` (*Optional*, Default: `10`): Items limit.
  * `role_name` (*Optional*): Filter by specific role (e.g. `'DOCTOR'`, `'NURSE'`).
  * `search` (*Optional*): Search query matching staff name or email.
  * `department_id` (*Optional*): Filter by department UUID.
  * `shift` (*Optional*): Filter by shift type (`'Day'`, `'Night'`).
  * `status` (*Optional*): Filter by status (`'Active'`, `'On Call'`, `'On Leave'`).

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "id": "ffeee23d-9a29-454f-9f46-192f1aaab285",
        "email": "doctor@sailjb.com",
        "full_name": "Dr. Arjun Mehta",
        "role_name": "DOCTOR",
        "department_id": "0fbfdbef-9873-4824-b1fd-6fb827d3ba57",
        "department_name": "Cardiology",
        "is_active": true,
        "created_at": "2026-06-15T12:00:00.000Z",
        "specialization_name": "Interventional Cardiology",
        "shift_label": "Day (08:00 - 16:00)",
        "current_status": "Active"
      }
    ],
    "total": 1
  }
}
```

---

### 3.2 PATCH Update Staff Fee
Set consultation fees for a doctor.

* **Method**: `PATCH`
* **URL**: `/admin/staff/{user_id}/fee`
* **Path Parameters**:
  * `user_id`: UUID of the staff user.

#### Request Body (`UpdateConsultationFeeRequest`)
```json
{
  "consultation_fee": 500.00,
  "follow_up_fee": 300.00,
  "validity_days": 10
}
```

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "message": "Consultation fees updated successfully"
}
```

---

### 3.3 GET Custom User Permissions
List specific permission overrides granted to a user.

* **Method**: `GET`
* **URL**: `/admin/staff/{user_id}/permissions`
* **Path Parameters**:
  * `user_id`: UUID of the staff user.

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "user_id": "ffeee23d-9a29-454f-9f46-192f1aaab285",
    "permissions": ["PATIENT_DELETE", "PROCUREMENT_ORDER_APPROVE"]
  }
}
```

---

### 3.4 POST Assign Custom Permissions
Apply overriding permission privileges to a staff user.

* **Method**: `POST`
* **URL**: `/admin/staff/{user_id}/permissions`
* **Path Parameters**:
  * `user_id`: UUID of the staff user.

#### Request Body (`AssignStaffPermissionsRequest`)
```json
{
  "permissions": ["PATIENT_DELETE", "PROCUREMENT_ORDER_APPROVE"]
}
```

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "message": "Custom user permissions assigned successfully"
}
```

---

## 4. Attendance & Leaves Management

### 4.1 GET List Applied Leaves
Retrieve leave requests applied by the active caller.

* **Method**: `GET`
* **URL**: `/admin/staff/me/leaves`

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "id": "l1-ffeee23d-9a29",
      "leave_type": "CASUAL",
      "from_date": "2026-08-01",
      "to_date": "2026-08-05",
      "days": 4,
      "reason": "Family vacation",
      "status": "PENDING"
    }
  ]
}
```

---

### 4.2 POST Apply Leave Request
Submit a new leave application.

* **Method**: `POST`
* **URL**: `/admin/staff/me/leaves/apply`

#### Request Body (`ApplyLeaveRequest`)
```json
{
  "leave_type": "CASUAL",
  "from_date": "2026-08-01",
  "to_date": "2026-08-05",
  "reason": "Family vacation"
}
```

#### Response (`201 Created`)
*Returns the applied LeaveResponse object.*

---

### 4.3 GET List Pending Leaves
List pending leave applications requiring approval.

* **Method**: `GET`
* **URL**: `/admin/staff/leaves/pending`
* **Query Parameters**:
  * `page` (*Optional*, Default: `1`): Current page number.
  * `page_size` (*Optional*, Default: `10`): Items limit.

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "leave_id": "l1-ffeee23d-9a29",
        "user_id": "8c59f032-47ef-4e1b-b46e-1d54238e4a90",
        "full_name": "Nisha Verma",
        "role_name": "NURSE",
        "leave_type": "CASUAL",
        "from_date": "2026-08-01",
        "to_date": "2026-08-05",
        "days": 4,
        "reason": "Family vacation",
        "status": "PENDING"
      }
    ],
    "total": 1
  }
}
```

---

### 4.4 PATCH Approve Leave Request
Authorize a pending leave application.

* **Method**: `PATCH`
* **URL**: `/admin/staff/leaves/{leave_id}/approve`
* **Path Parameters**:
  * `leave_id`: UUID of the leave.

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "message": "Leave application approved successfully",
  "data": {
    "leave_id": "l1-ffeee23d-9a29",
    "status": "APPROVED"
  }
}
```

---

## 5. Floors, Blocks, Wards, Units & Beds Setup

### 5.1 GET List Floors
Retrieve all registered floors.

* **Method**: `GET`
* **URL**: `/admin/floors`
* **Query Parameters**:
  * `include_inactive` (*Optional*, Default: `false`): Include inactive floors.

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "id": "93920321-63e8-4646-ba4f-dc976ec6dfda",
        "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
        "floor_number": 1,
        "floor_label": "Ground Floor",
        "is_active": true,
        "created_at": "2026-07-04T12:41:25.168100+05:30"
      }
    ],
    "total": 1
  }
}
```

---

### 5.2 POST Create Floor
Add a new floor.

* **Method**: `POST`
* **URL**: `/admin/floors`

#### Request Body (`FloorCreateRequest`)
```json
{
  "floor_number": 2,
  "floor_label": "First Floor"
}
```

#### Response (`201 Created`)
*Returns the newly created FloorResponse details.*

---

### 5.3 GET List Blocks
Retrieve all building blocks.

* **Method**: `GET`
* **URL**: `/admin/blocks`
* **Query Parameters**:
  * `floor_id` (*Optional*): Filter by specific floor (UUID).
  * `include_inactive` (*Optional*): Include inactive blocks.

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "id": "8c59f032-47ef-4e1b-b46e-1d54238e4a90",
        "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
        "floor_id": "93920321-63e8-4646-ba4f-dc976ec6dfda",
        "name": "General Block A",
        "code": "BLK-A",
        "description": "Primary clinical inpatient tower block",
        "is_active": true,
        "created_at": "2026-07-04T12:45:00.000Z"
      }
    ],
    "total": 1
  }
}
```

---

### 5.4 GET List Wards
List wards with active statistics.

* **Method**: `GET`
* **URL**: `/admin/wards`
* **Query Parameters**:
  * `include_inactive` (*Optional*): Include inactive records (Default: `false`).
  * `floor` (*Optional*): Filter by floor string label.
  * `ward_type` (*Optional*): Filter by `WardType` value.
  * `work_area_id` (*Optional*): Filter by mapped work area ID (UUID).

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "id": "673f8a09-7d8b-402a-a92c-7b2496a7d5ea",
        "tenant_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
        "name": "Male Medical Ward",
        "ward_type": "GENERAL",
        "work_area_id": "aa30b4b7-0da8-4b9d-8643-4fdffefabe9d",
        "floor": "Ground Floor",
        "block_id": "8c59f032-47ef-4e1b-b46e-1d54238e4a90",
        "capacity": 30,
        "is_active": true,
        "created_at": "2026-07-04T12:50:00.000Z",
        "total_beds": 30,
        "available_beds": 25,
        "occupied_beds": 5
      }
    ],
    "total": 1
  }
}
```

---

### 5.5 POST Create Ward
Create a new ward.

* **Method**: `POST`
* **URL**: `/admin/wards`

#### Request Body (`WardCreateRequest`)
```json
{
  "name": "Male Medical Ward",
  "ward_type": "GENERAL",
  "capacity": 30,
  "floor": "Ground Floor",
  "block_id": "8c59f032-47ef-4e1b-b46e-1d54238e4a90"
}
```

#### Response (`201 Created`)
*Returns the newly created WardResponse details.*

---

### 5.6 GET Beds Summary
Retrieve aggregated bed counts and statuses.

* **Method**: `GET`
* **URL**: `/admin/beds/summary`

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "total_beds": 100,
    "available": 60,
    "occupied": 30,
    "reserved": 5,
    "maintenance": 5,
    "by_ward": [
      {
        "ward_id": "673f8a09-7d8b-402a-a92c-7b2496a7d5ea",
        "ward_name": "Male Medical Ward",
        "ward_type": "GENERAL",
        "floor": "Ground Floor",
        "capacity": 30,
        "total_beds": 30,
        "available": 25,
        "occupied": 5,
        "reserved": 0,
        "maintenance": 0
      }
    ]
  }
}
```

---

### 5.7 GET List Beds
Retrieve a paginated, filterable list of all beds.

* **Method**: `GET`
* **URL**: `/admin/beds`
* **Query Parameters**:
  * `page` (*Optional*, Default: `1`): Current page.
  * `page_size` (*Optional*, Default: `20`): Page limit.
  * `ward_id` (*Optional*): Filter by parent ward ID.
  * `status` (*Optional*): Filter by `BedStatus` value.

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "id": "e3cd1405-c1d4-482a-a82f-b4de673ba51f",
        "tenant_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
        "ward_id": "673f8a09-7d8b-402a-a92c-7b2496a7d5ea",
        "ward_name": "Male Medical Ward",
        "floor": "Ground Floor",
        "bed_number": "B-101",
        "bed_type": "STANDARD",
        "status": "OCCUPIED",
        "is_active": true,
        "created_at": "2026-07-04T13:00:00.000Z"
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

### 5.8 POST Create Bed
Add a new physical bed.

* **Method**: `POST`
* **URL**: `/admin/beds`

#### Request Body (`BedCreateRequest`)
```json
{
  "ward_id": "673f8a09-7d8b-402a-a92c-7b2496a7d5ea",
  "bed_number": "B-102",
  "bed_type": "STANDARD"
}
```

#### Response (`201 Created`)
*Returns the newly created BedResponse details.*

---

### 5.9 PATCH Update Bed Status
Update operational status of a bed.

* **Method**: `PATCH`
* **URL**: `/admin/beds/{bed_id}/status`
* **Path Parameters**:
  * `bed_id`: UUID of the bed.

#### Request Body (`BedStatusUpdateRequest`)
```json
{
  "status": "MAINTENANCE",
  "reason": "Routine cleaning and minor repairs"
}
```

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "message": "Bed status updated successfully"
}
```

---

## 6. Setup & Master Data (Departments & Specializations)

### 6.1 GET List Departments
Lists departments with staff counts, location, and operational status.

* **Method**: `GET`
* **URL**: `/admin/departments`
* **Query Parameters**:
  * `search` (*Optional*): Search term matching department name.
  * `status` (*Optional*): Filter by status (`Active`, `Under Maintenance`, `Inactive`).
  * `page` (*Optional*, Default: `1`): Page number.
  * `page_size` (*Optional*, Default: `10`): Items limit.
  * `staff_search` (*Optional*): Search query to filter staff members within departments by name/email/role.
  * `staff_shift` (*Optional*): Filter nested staff by shift type (`'Day'`, `'Night'`).
  * `staff_status` (*Optional*): Filter nested staff by status (`'Active'`, `'On Call'`, `'On Leave'`).

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "department_id": "0fbfdbef-9873-4824-b1fd-6fb827d3ba57",
        "department_name": "Cardiology",
        "code": "CARD-001",
        "department_type": "Clinical",
        "head_of_department": {
          "doctor_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
          "name": "Dr. Arjun Nair",
          "title": "Chief of Cardiology"
        },
        "building_block": "A Block",
        "floor": "2nd Floor",
        "wing": "East Wing",
        "rooms": "Rooms 201-210",
        "staff_breakdown": {
          "doctors_count": 12,
          "nurses_count": 18,
          "support_staff_count": 6,
          "rooms_count": 10
        },
        "operating_hours": "09:00 AM - 06:00 PM",
        "status": "Active",
        "is_active": true,
        "created_at": "2026-01-15T00:00:00Z",
        "staffs": [
          {
            "name": "Dr. Sarah Jenkins",
            "email": "sarah.jenkins@arovita.com",
            "role": "HEAD_NURSE",
            "specialization": "General Cardiology",
            "shift": "Day (08:00 - 16:00)",
            "status": "Active"
          }
        ]
      }
    ],
    "pagination": {
      "page": 1,
      "page_size": 10,
      "total_records": 1,
      "total_pages": 1
    }
  }
}
```

---

### 6.2 POST Create Department
Create a new department.

* **Method**: `POST`
* **URL**: `/admin/departments`

#### Request Body (`DepartmentCreateRequest`)
```json
{
  "department_name": "Emergency & Critical Care",
  "code": "EMER-001",
  "department_type": "Clinical"
}
```

#### Response (`201 Created`)
*Returns created department fields.*

---

## 7. Inventory & Procurement Management

### 7.1 GET List Medical Inventory
Retrieve list of medical items in stock.

* **Method**: `GET`
* **URL**: `/admin/inventory/medical`
* **Query Parameters**:
  * `page` (*Optional*, Default: `1`): Page number.
  * `page_size` (*Optional*, Default: `20`): Page size limit.
  * `search` (*Optional*): Search name or code.
  * `low_stock` (*Optional*): `true` to filter items below safety stock.

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "id": "aa30b4b7-0da8-4b9d-8643-4fdffefabe9a",
        "name": "Paracetamol 500mg",
        "sku_code": "MED-PAR-500",
        "batch_count": 3,
        "total_quantity": 450,
        "safety_stock": 100,
        "unit": "Tablet",
        "is_active": true
      }
    ],
    "total": 120
  }
}
```

---

### 7.2 POST Add Medical Inventory
Add a new item to the inventory.

* **Method**: `POST`
* **URL**: `/admin/inventory/medical`

#### Request Body (`AddMedicalInventoryRequest`)
```json
{
  "name": "Amoxicillin Capsule 250mg",
  "sku_code": "MED-AMX-250",
  "safety_stock": 150,
  "unit": "Capsule",
  "description": "Broad-spectrum antibiotic"
}
```

#### Response (`201 Created`)
*Returns the created item fields.*

---

### 7.3 GET List Stock Transfers
Retrieve inter-departmental transfers.

* **Method**: `GET`
* **URL**: `/admin/inventory/transfers`
* **Query Parameters**:
  * `status` (*Optional*): Filter by status (`PENDING`, `APPROVED`, `REJECTED`).

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "id": "t1-aa30b4b7",
        "source_department_name": "Main Store",
        "destination_department_name": "Emergency Ward",
        "item_name": "Paracetamol 500mg",
        "quantity": 100,
        "status": "PENDING"
      }
    ],
    "total": 1
  }
}
```

---

### 7.4 POST Approve Stock Transfer
Commit transfer adjustments to database inventory balances.

* **Method**: `POST`
* **URL**: `/admin/inventory/transfers/{transfer_id}/approve`
* **Path Parameters**:
  * `transfer_id`: UUID of transfer.

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "message": "Stock transfer approved successfully"
}
```

---

## 8. TPA & Insurance Claims Management

### 8.1 GET List TPA Providers
List active TPAs.

* **Method**: `GET`
* **URL**: `/admin/tpa/providers`

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": [
    { "id": "tpa-1", "name": "Star Health Insurance", "is_active": true }
  ]
}
```

---

### 8.2 POST Create TPA Provider
Add a new TPA provider.

* **Method**: `POST`
* **URL**: `/admin/tpa/providers`

#### Request Body (`TPAProviderCreateRequest`)
```json
{
  "name": "Star Health Insurance",
  "contact_person": "Mr. Amit Shah",
  "email": "claims@starhealth.com",
  "phone": "+91 22 2345 6789"
}
```

#### Response (`201 Created`)
*Returns created provider details.*

---

## 9. Audit, Alerts & Compliance

### 9.1 GET List System Alerts
List system alerts.

* **Method**: `GET`
* **URL**: `/admin/alerts`
* **Query Parameters**:
  * `is_resolved` (*Optional*): Filter by resolved status (`true` / `false`).

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "id": "alert-1",
        "alert_type": "STOCK_LOW",
        "severity": "HIGH",
        "title": "Low Stock: Paracetamol 500mg",
        "message": "Current inventory count is below safety stock limit.",
        "is_acknowledged": false,
        "is_resolved": false
      }
    ],
    "total": 1
  }
}
```

---

### 9.2 POST Resolve Alert
Mark an active system alert resolved.

* **Method**: `POST`
* **URL**: `/admin/alerts/{alert_id}/resolve`
* **Path Parameters**:
  * `alert_id`: UUID of alert.

#### Request Body (`AlertResolveRequest`)
```json
{
  "resolution_notes": "Stock replenished via purchase order PO-2026-0001."
}
```

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "message": "Alert marked resolved successfully"
}
```

---

### 9.3 GET List System Audit Logs
Retrieve the branch administrative audit logs list.

* **Method**: `GET`
* **URL**: `/admin/audit-logs`
* **Query Parameters**:
  * `page` (*Optional*, Default: `1`): Page number.
  * `page_size` (*Optional*, Default: `20`): Page size limit.
  * `user_id` (*Optional*): Filter logs by operator user ID (UUID).
  * `action` (*Optional*): Filter by specific event type.

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "id": "a901f41b-4fef-48a1-b84e-8664fd37a912",
        "user_id": "ffeee23d-9a29-454f-9f46-192f1aaab285",
        "action": "bed.status_update",
        "entity": "bed",
        "entity_id": "f89b018a-f8ed-4388-b49e-cfe46fb2d3f9",
        "status": "success",
        "ip_address": "192.168.1.15",
        "metadata": {
          "bed_number": "B-102-Mod",
          "status": "MAINTENANCE"
        },
        "created_at": "2026-07-25T11:42:00.000Z"
      }
    ],
    "meta": {
      "total": 1450,
      "page": 1,
      "page_size": 20,
      "total_pages": 73
    }
  }
}
```

---

## 10. Specialty Services (Pharmacy, Radiology, Laboratory)

These endpoints manage diagnostic schedules, laboratory tracking, and pharmacy purchase workflows.

### 10.1 GET Pharmacy Overview Summary
Retrieve aggregated inventory counts, low stock indicators, and pending purchase orders.

* **Method**: `GET`
* **URL**: `/admin/pharmacy-overview/summary`
* **Query Parameters**: None

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "total_items": 214,
    "pending_orders": 18,
    "critical_stock_pct": 18.0,
    "avg_lead_time_days": 4.2
  }
}
```

---

### 10.2 GET List Purchase Orders
Retrieve a paginated list of purchase orders.

* **Method**: `GET`
* **URL**: `/admin/pharmacy-overview/purchase-orders`
* **Query Parameters**:
  * `supplier` (*Optional*): Filter by vendor name.
  * `approval_status` (*Optional*): `Approved`, `Pending`, `Draft`, `Rejected`.
  * `delivery_status` (*Optional*): `Pending`, `Shipped`, `Delivered`.
  * `search` (*Optional*): Match supplier or items.
  * `page` (*Optional*, Default: `1`): Page number.
  * `page_size` (*Optional*, Default: `10`): Items limit.

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "po_id": "PO-2026-0001",
        "supplier": "MedLife Pharmacies",
        "order_date": "2026-07-24",
        "items_count": 8,
        "amount": 14250.00,
        "approval_status": "Approved",
        "delivery_status": "Delivered"
      }
    ],
    "meta": {
      "total": 128,
      "page": 1,
      "page_size": 10,
      "total_pages": 13
    }
  }
}
```

---

### 10.3 POST Create Purchase Order
Create a new purchase order draft or request.

* **Method**: `POST`
* **URL**: `/admin/pharmacy-overview/purchase-orders`
* **Request Body**:
```json
{
  "supplier": "MedLife Pharmacies",
  "items": [
    {
      "item_id": "MED-402",
      "quantity": 1000,
      "unit_price": 2.50
    }
  ]
}
```

#### Response (`201 Created`)
```json
{
  "success": true,
  "code": 201,
  "data": {
    "po_id": "PO-2026-0182",
    "supplier": "MedLife Pharmacies",
    "approval_status": "Pending",
    "delivery_status": "Pending"
  }
}
```

---

### 10.4 PATCH Update Purchase Order
Modify approval status or delivery progress of a purchase order.

* **Method**: `PATCH`
* **URL**: `/admin/pharmacy-overview/purchase-orders/{po_id}`
* **Path Parameters**:
  * `po_id`: String identifier of the purchase order.
* **Request Body**:
```json
{
  "approval_status": "Approved",
  "delivery_status": "Shipped"
}
```

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "po_id": "PO-2026-0001",
    "approval_status": "Approved",
    "delivery_status": "Shipped"
  }
}
```

---

### 10.5 GET Radiology Overview Summary
Retrieve diagnostic equipment utilization metrics.

* **Method**: `GET`
* **URL**: `/admin/radiology-overview/summary`

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "mri_utilization": 92.0,
    "ct_utilization": 78.0,
    "xray_utilization": 64.0,
    "ultrasound_utilization": 0.0
  }
}
```

---

### 10.6 GET List Radiology Schedule
Retrieve active scan board appointments list.

* **Method**: `GET`
* **URL**: `/admin/radiology-overview/schedule`
* **Query Parameters**:
  * `modality` (*Optional*): `MRI`, `CT`, `X-RAY`, `ULTRASOUND`.
  * `status` (*Optional*): `Scheduled`, `In Progress`, `Completed`, `Cancelled`.
  * `search` (*Optional*): Search patient name.
  * `page` (*Optional*, Default: `1`): Page offset.
  * `page_size` (*Optional*, Default: `10`): Items limit.

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "appointment_id": "IMG-2026-789",
        "patient_name": "Rohan Gupta",
        "scan_type": "MRI Brain Contrast",
        "modality": "MRI",
        "time_slot": "10:30 AM - 11:15 AM",
        "status": "In Progress"
      }
    ],
    "meta": {
      "total": 48,
      "page": 1,
      "page_size": 10,
      "total_pages": 5
    }
  }
}
```

---

### 10.7 POST Create Radiology Schedule
Book/schedule a scan for a patient.

* **Method**: `POST`
* **URL**: `/admin/radiology-overview/schedule`
* **Request Body**:
```json
{
  "patient_name": "Rohan Gupta",
  "modality": "MRI",
  "scheduled_start": "2026-07-25T10:30:00Z",
  "notes": "Patient has claustrophobia, monitor closely."
}
```

#### Response (`201 Created`)
```json
{
  "success": true,
  "code": 201,
  "data": {
    "appointment_id": "IMG-2026-892",
    "patient_name": "Rohan Gupta",
    "modality": "MRI",
    "status": "Scheduled"
  }
}
```

---

### 10.8 GET Laboratory Overview Summary
Retrieve counts of tests and alerts in lab queues.

* **Method**: `GET`
* **URL**: `/admin/laboratory-overview/summary`

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "total_tests": 428,
    "completed_tests": 332,
    "pending_tests": 46,
    "critical_alerts": 64,
    "avg_turnaround_time_mins": 86
  }
}
```

---

### 10.9 GET List Laboratory Tests
Retrieve paginated laboratory sample scanning logs.

* **Method**: `GET`
* **URL**: `/admin/laboratory-overview/tests`
* **Query Parameters**:
  * `department` (*Optional*): Filter by lab department (e.g. `'Hematology'`).
  * `status` (*Optional*): Filter by status (`Pending`, `Processing`, `Completed`).
  * `search` (*Optional*): Search patient or test.
  * `page` (*Optional*, Default: `1`): Page number.
  * `page_size` (*Optional*, Default: `10`): Items limit.

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "test_id": "LAB-2026-4402",
        "patient_name": "Rohan Gupta",
        "test_type": "Complete Blood Count (CBC)",
        "urgency": "Routine",
        "status": "Processing",
        "received_at": "2026-07-25T11:00:00Z"
      }
    ],
    "meta": {
      "total": 46,
      "page": 1,
      "page_size": 10,
      "total_pages": 5
    }
  }
}
```

---

### 10.10 POST Create Laboratory Test
Register a new lab sample test order.

* **Method**: `POST`
* **URL**: `/admin/laboratory-overview/tests`
* **Request Body**:
```json
{
  "patient_name": "Rohan Gupta",
  "test_type": "Complete Blood Count (CBC)",
  "sample_type": "Whole Blood (EDTA)",
  "urgency": "Routine",
  "notes": "Prioritize reporting."
}
```

#### Response (`201 Created`)
```json
{
  "success": true,
  "code": 201,
  "data": {
    "test_id": "LAB-2026-4589",
    "patient_name": "Rohan Gupta",
    "test_type": "Complete Blood Count (CBC)",
    "status": "Pending"
  }
}
```

