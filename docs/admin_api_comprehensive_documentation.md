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

---

## 11. Service Catalog Management APIs

### 11.1 GET Service Catalogue Summary
Retrieve hospital-wide metrics for the Service Catalog.

* **Method**: `GET`
* **URL**: `/admin/service-catalogue/summary`

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "total_catalogue_items": 248,
    "active_services": 215,
    "speciality_packages": 142,
    "diagnostic_tests": 108,
    "total_monthly_revenue": "125000.00",
    "services_by_category": [
      {
        "category": "PREVENTIVE",
        "percentage": 12.0,
        "count": 30
      }
    ],
    "top_services_by_volume": [
      {
        "service": "Cardiology Consultation",
        "count": 420
      }
    ]
  }
}
```

---

### 11.2 GET List Service Catalogue Records
Retrieve a paginated, filterable list of clinical services.

* **Method**: `GET`
* **URL**: `/admin/service-catalogue/records`
* **Query Parameters**:
  * `category` (*Optional*): Filter by service category (`CLINICAL`, `DIAGNOSTIC`, `SURGICAL`, `PREVENTIVE`, `EMERGENCY`).
  * `department_id` (*Optional*): Filter by department UUID.
  * `status` (*Optional*): Filter by status (`ACTIVE`, `INACTIVE`, `SEASONAL`, `SUSPENDED`).
  * `search` (*Optional*): Search query matching service name or code.
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
        "id": "0b4c69fb-63d4-4f16-aa28-96f841d54485",
        "service_name": "General Consultation",
        "category": "PREVENTIVE",
        "department_id": "46d19155-09a1-462e-b256-ce2a5741b091",
        "department_name": "Surgery",
        "service_code": "SERV-20260727",
        "base_fee": "650.00",
        "doctor_share_pct": "60.00",
        "hospital_share_pct": "40.00",
        "tax_rate_pct": "5.00",
        "status": "Active",
        "is_active": true,
        "availability_status": "ACTIVE",
        "description": "Standard general medical consultation",
        "emergency_surcharge": "0.00",
        "after_hours_surcharge": "0.00",
        "avg_duration_minutes": 15,
        "max_daily_capacity": 40,
        "is_insurance_covered": true,
        "tpa_approval_required": false
      }
    ],
    "meta": {
      "total": 1,
      "page": 1,
      "page_size": 10,
      "total_pages": 1
    }
  }
}
```

---

### 11.3 POST Create Service Catalogue Record
Create a new clinical service catalog entry.

* **Method**: `POST`
* **URL**: `/admin/service-catalogue/records`
* **Request Body**:
```json
{
  "service_name": "QA General Consultation",
  "category": "PREVENTIVE",
  "department_id": "46d19155-09a1-462e-b256-ce2a5741b091",
  "base_fee": 650.00,
  "doctor_share_pct": 60.00,
  "hospital_share_pct": 40.00,
  "tax_rate_pct": 5.00,
  "availability_status": "ACTIVE",
  "description": "Standard QA general medical consultation"
}
```

#### Response (`201 Created`)
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "0b4c69fb-63d4-4f16-aa28-96f841d54485",
    "service_name": "QA General Consultation",
    "service_code": "SERV-20260727",
    "base_fee": "650.00",
    "doctor_share_pct": "60.00",
    "hospital_share_pct": "40.00",
    "tax_rate_pct": "5.00",
    "availability_status": "ACTIVE"
  }
}
```

---

### 11.4 GET Single Service Catalogue Record
Retrieve complete configuration details of a single service by its ID.

* **Method**: `GET`
* **URL**: `/admin/service-catalogue/records/{service_id}`

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "0b4c69fb-63d4-4f16-aa28-96f841d54485",
    "service_name": "QA General Consultation",
    "category": "PREVENTIVE",
    "department_id": "46d19155-09a1-462e-b256-ce2a5741b091",
    "department_name": "Surgery",
    "service_code": "SERV-20260727",
    "base_fee": "650.00",
    "doctor_share_pct": "60.00",
    "hospital_share_pct": "40.00",
    "tax_rate_pct": "5.00",
    "status": "Active",
    "is_active": true,
    "availability_status": "ACTIVE",
    "description": "Standard QA general medical consultation",
    "emergency_surcharge": "0.00",
    "after_hours_surcharge": "0.00",
    "avg_duration_minutes": 15,
    "max_daily_capacity": 40,
    "is_insurance_covered": true,
    "tpa_approval_required": false
  }
}
```

---

### 11.5 PATCH Update Service Catalogue Record
Partially update service details. Modifying the `base_fee` will trigger pricing history log insertion.

* **Method**: `PATCH`
* **URL**: `/admin/service-catalogue/records/{service_id}`
* **Request Body**:
```json
{
  "base_fee": 700.00,
  "change_reason": "Adjust fee for annual pricing review"
}
```

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "0b4c69fb-63d4-4f16-aa28-96f841d54485",
    "service_name": "QA General Consultation",
    "base_fee": "700.00",
    "updated_at": "2026-07-27T11:21:36Z"
  }
}
```

---

### 11.6 POST Configure Service Pricing Tiers
Configure ward-specific or TPA-specific pricing overrides for a service.

* **Method**: `POST`
* **URL**: `/admin/service-catalogue/records/{service_id}/pricing-tiers`
* **Request Body**:
```json
{
  "tiers": [
    {
      "room_category": "ICU",
      "custom_fee": 1200.00,
      "tpa_id": null,
      "is_active": true
    }
  ]
}
```

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "success": true,
    "message": "Pricing tiers applied successfully"
  }
}
```

---

### 11.7 GET Service Audit Trail
Fetch audit history tracking modifications to this service catalog record.

* **Method**: `GET`
* **URL**: `/admin/service-catalogue/records/{service_id}/audit-trail`

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "id": "af9bed60-a8ba-4ed9-9787-c6bf2116262a",
      "service_id": "0b4c69fb-63d4-4f16-aa28-96f841d54485",
      "action_type": "UPDATED",
      "old_data": {
        "base_fee": "650.00"
      },
      "new_data": {
        "base_fee": "700.00"
      },
      "performed_by": "902d2bc4-f5ee-45df-98bd-674cd7bb0eef",
      "timestamp": "2026-07-27T11:21:36.727Z"
    }
  ]
}
```

---

### 11.8 GET Service Pricing History
Retrieve historical changes to this service's base fee tariff.

* **Method**: `GET`
* **URL**: `/admin/service-catalogue/records/{service_id}/pricing-history`

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "id": "20f39a9e-82ac-4e2c-b6fc-dabd261d0b75",
      "service_id": "0b4c69fb-63d4-4f16-aa28-96f841d54485",
      "old_base_fee": "650.00",
      "new_base_fee": "700.00",
      "change_reason": "Adjust fee for annual pricing review",
      "changed_by": "902d2bc4-f5ee-45df-98bd-674cd7bb0eef",
      "changed_at": "2026-07-27T11:21:36.727Z"
    }
  ]
}
```

---

## 12. Scheduling & Appointments APIs

### 12.1 GET Scheduling Roster
Retrieve the doctors' roster grid matrix representing shifts, clinical schedules, leaves, and OPD clinic room locations for a given date range.

* **Method**: `GET`
* **URL**: `/admin/scheduling/roster`
* **Query Parameters**:
  * `start_date` (optional, default: Monday of current week YYYY-MM-DD)
  * `end_date` (optional, default: Sunday of current week YYYY-MM-DD)
  * `department_id` (optional, filter by department UUID)
  * `search` (optional, search by clinician name or employee code)

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "doctor_id": "0d9fcf29-112a-43d2-a5cb-830f941596a7",
      "doctor_name": "Dr. Kiran Sharma",
      "specialization": "Surgery Specialist",
      "department": "Surgery / Operation Theatre",
      "avatar_url": null,
      "shifts": [
        {
          "date": "2026-08-04",
          "shift_label": "Leave",
          "start_time": null,
          "end_time": null,
          "room_info": null,
          "status": "Annual Doctor Leave"
        },
        {
          "date": "2026-08-05",
          "shift_label": "Full Day",
          "start_time": "09:00:00",
          "end_time": "18:00:00",
          "room_info": "Room 304, OPD Clinic",
          "status": "Confirmed Shift"
        }
      ]
    }
  ]
}
```

---

### 12.2 POST Create Availability Override
Configure a doctor's schedule override (such as booking leaves, blocks, or adding extra shifts) for a specific date.

* **Method**: `POST`
* **URL**: `/admin/scheduling/availability-overrides`

#### Request Body (`AvailabilityOverrideCreateRequest`)
```json
{
  "doctor_id": "0d9fcf29-112a-43d2-a5cb-830f941596a7",
  "override_date": "2026-08-04",
  "override_type": "BLOCK",
  "reason": "Annual Doctor Leave"
}
```

#### Response (`201 Created`)
```json
{
  "success": true,
  "code": 201,
  "message": "Created successfully",
  "data": {
    "id": "7512a1af-cec8-4e7f-ae77-bca0e712a01a",
    "doctor_id": "0d9fcf29-112a-43d2-a5cb-830f941596a7",
    "override_date": "2026-08-04",
    "override_type": "BLOCK",
    "start_time": null,
    "end_time": null,
    "status": null,
    "reason": "Annual Doctor Leave"
  }
}
```

---

### 12.3 GET Doctor Free Slots
Calculate and retrieve all open/available slot intervals (e.g. 15-minute slot times) for a doctor on a specific date, factoring in templates, overrides, and existing appointments.

* **Method**: `GET`
* **URL**: `/admin/scheduling/doctors/{doctor_id}/free-slots`
* **Query Parameters**:
  * `date` (required, target date YYYY-MM-DD)

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": [
    "09:00",
    "09:15",
    "09:30",
    "09:45"
  ]
}
```

---

### 12.4 POST Book Appointment
Book a consultation session slot for a patient with a designated doctor.

* **Method**: `POST`
* **URL**: `/admin/scheduling/appointments`

#### Request Body (`AppointmentCreateRequest`)
```json
{
  "patient_id": "93643ddd-0d0d-491d-83d6-37f3bcde518d",
  "doctor_id": "0d9fcf29-112a-43d2-a5cb-830f941596a7",
  "appointment_time": "2026-08-05T09:00:00+05:30",
  "appointment_type": "NEW",
  "department_id": "222505d4-8680-40ba-9be4-bc9b50cca031",
  "is_emergency": false,
  "priority": 2,
  "notes": "Routine physical checkup"
}
```

#### Response (`201 Created`)
```json
{
  "success": true,
  "code": 201,
  "message": "Created successfully",
  "data": {
    "id": "586f1611-641a-4f97-bde3-e2a95d5c7b0a",
    "patient_id": "93643ddd-0d0d-491d-83d6-37f3bcde518d",
    "patient_name": "E2E Simulation Patient",
    "doctor_id": "0d9fcf29-112a-43d2-a5cb-830f941596a7",
    "doctor_name": "Dr. Kiran Sharma",
    "appointment_time": "2026-08-05 09:00:00+05:30",
    "token_number": 3,
    "status": "SCHEDULED",
    "appointment_type": "NEW",
    "is_emergency": false,
    "priority": 2,
    "notes": "Routine physical checkup",
    "created_at": "2026-07-27 11:47:40.677200+05:30"
  }
}
```

---

### 12.5 GET List Appointments
Retrieve a list of booked patient appointments with filters.

* **Method**: `GET`
* **URL**: `/admin/scheduling/appointments`
* **Query Parameters**:
  * `doctor_id` (optional, filter by doctor UUID)
  * `date` (optional, filter by target date YYYY-MM-DD)
  * `status` (optional, filter by status e.g. SCHEDULED, COMPLETED, CANCELLED)
  * `department_id` (optional, filter by department UUID)
  * `appointment_type` or `type` (optional, filter by e.g. NEW, FOLLOW_UP)
  * `start_date` (optional, filter by appointment date range start YYYY-MM-DD)
  * `end_date` (optional, filter by appointment date range end YYYY-MM-DD)

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "id": "586f1611-641a-4f97-bde3-e2a95d5c7b0a",
      "patient_id": "93643ddd-0d0d-491d-83d6-37f3bcde518d",
      "patient_name": "E2E Simulation Patient",
      "patient_phone": "+919987721170",
      "patient_email": "patient@mail.com",
      "doctor_id": "0d9fcf29-112a-43d2-a5cb-830f941596a7",
      "doctor_name": "Dr. Kiran Sharma",
      "department_id": "222505d4-8680-40ba-9be4-bc9b50cca031",
      "department_name": "Surgery / Operation Theatre",
      "appointment_time": "2026-08-05 09:00:00+05:30",
      "token_number": 3,
      "status": "SCHEDULED",
      "appointment_type": "NEW",
      "is_emergency": false,
      "priority": 2,
      "notes": "Routine physical checkup",
      "created_at": "2026-07-27 11:47:40.677200+05:30"
    }
  ]
}
```

---

### 12.6 PATCH Update Appointment Status
Reschedule, cancel, check-in, or complete an appointment booking.

* **Method**: `PATCH`
* **URL**: `/admin/scheduling/appointments/{appointment_id}`

#### Request Body
```json
{
  "status": "CANCELLED",
  "notes": "Patient rescheduled to next week"
}
```

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "586f1611-641a-4f97-bde3-e2a95d5c7b0a",
    "patient_id": "93643ddd-0d0d-491d-83d6-37f3bcde518d",
    "patient_name": "E2E Simulation Patient",
    "doctor_id": "0d9fcf29-112a-43d2-a5cb-830f941596a7",
    "doctor_name": "Dr. Kiran Sharma",
    "appointment_time": "2026-08-05 09:00:00+05:30",
    "token_number": 3,
    "status": "CANCELLED",
    "appointment_type": "NEW",
    "is_emergency": false,
    "priority": 2,
    "notes": "Patient rescheduled to next week",
    "created_at": "2026-07-27 11:47:40.677200+05:30"
  }
}
```

---

## 13. OT (Operation Theatre) Scheduling APIs

### 13.1 GET List OT Sessions
Query Operation Theatre scheduled surgery slots, filters, room bookings, and statuses.

* **Method**: `GET`
* **URL**: `/admin/ot-scheduling/sessions`
* **Query Parameters**:
  * `date` (optional, filter by surgery date YYYY-MM-DD)
  * `status` (optional, filter by session status: SCHEDULED, PRE_OP, IN_PROGRESS, COMPLETED, CANCELLED)
  * `ot_room_id` (optional, filter by Operation Theatre ID)

#### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "id": "40b17c91-cf62-4214-a957-ea1bcbb2586a",
      "ot_number": "OT-2026-0001",
      "ot_room_id": "OT-2 (Cardiac)",
      "scheduled_start": "2026-08-05T07:30:00+05:30",
      "scheduled_end": "2026-08-05T11:00:00+05:30",
      "patient_id": "93643ddd-0d0d-491d-83d6-37f3bcde518d",
      "patient_name": "E2E Simulation Patient",
      "status": "SCHEDULED",
      "team": [
        {
          "user_id": "0d9fcf29-112a-43d2-a5cb-830f941596a7",
          "name": "Dr. Kiran Sharma",
          "role": "Lead Surgeon",
          "status": "Available",
          "conflict_message": null
        }
      ],
      "checklist": {
        "consent_form_signed": false,
        "blood_reserve_complete": false,
        "pre_op_diagnostics": false,
        "anaesthesia_clearance": false,
        "scrub_nurse_equipment": false,
        "pac_clearance": "PENDING"
      },
      "cost_breakdown": {
        "ot_room_charges": 50000.0,
        "surgeon_fee": 120000.0,
        "anaesthesia_fee": 45000.0,
        "consumables_devices": 85000.0,
        "basic_sterile_bundles": 0.0,
        "subtotal": 300000.0
      },
      "approval_compliance": {
        "insurance_pre_authorization": "PENDING",
        "clinical_quality_approval": "PENDING",
        "basic_sterile_bundle": "PENDING",
        "pac_clearance_approval": "PENDING"
      },
      "warnings": [],
      "emergency_override": false,
      "override_reason": null,
      "created_at": "2026-07-27T11:51:00.000000+05:30"
    }
  ]
}
```

---

### 13.2 POST Create OT Session
Schedule a new surgical operation session slot, allocating resources and team members.

* **Method**: `POST`
* **URL**: `/admin/ot-scheduling/sessions`

#### Request Body (`OTSessionCreateRequest`)
```json
{
  "service_order_id": "8bb38f12-70ff-4c22-b5e1-da8bcfca29a0",
  "admission_id": null,
  "patient_id": "93643ddd-0d0d-491d-83d6-37f3bcde518d",
  "surgeon_id": "0d9fcf29-112a-43d2-a5cb-830f941596a7",
  "anaesthetist_id": "ffeee23d-9a29-454f-9f46-192f1aaab285",
  "scrub_nurse_id": "e48fda57-af9a-4bb3-a80c-10395f0b5a8b",
  "ot_room_id": "OT-2 (Cardiac)",
  "scheduled_start": "2026-08-05T07:30:00+05:30",
  "scheduled_end": "2026-08-05T11:00:00+05:30",
  "pre_op_checklist": {
    "consent_form_signed": false,
    "blood_reserve_complete": false
  }
}
```

#### Response (`201 Created`)
*Matches GET detailed response.*

---

### 13.3 GET OT Session Detail
Retrieve clinical team parameters, checklists, and warning indicators for a single surgery.

* **Method**: `GET`
* **URL**: `/admin/ot-scheduling/sessions/{session_id}`

#### Response (`200 OK`)
*Matches GET detailed session structure with status code `200`.*

---

### 13.4 PATCH Update Pre-Op Checklist
Sign off or toggle surgical safety checklists (consents, clearances, reserves).

* **Method**: `PATCH`
* **URL**: `/admin/ot-scheduling/sessions/{session_id}/checklist`

#### Request Body
```json
{
  "consent_form_signed": true,
  "blood_reserve_complete": true,
  "pre_op_diagnostics": true,
  "anaesthesia_clearance": true,
  "scrub_nurse_equipment": true,
  "pac_clearance": "APPROVED"
}
```

#### Response (`200 OK`)
*Returns updated detailed session.*

---

### 13.5 POST Emergency Override
Bypass surgical team warnings or conflicts during emergency situations.

* **Method**: `POST`
* **URL**: `/admin/ot-scheduling/sessions/{session_id}/override`

#### Request Body
```json
{
  "override_reason": "Acute cardiac failure, immediate intervention required.",
  "override_doctor_id": "0d9fcf29-112a-43d2-a5cb-830f941596a7"
}
```

#### Response (`200 OK`)
*Returns updated detailed session with `emergency_override` flagged as `true`.*




