# Admin Staff, Leaves & Audit Logs API Documentation

This document provides specifications for the **Staff Management, Security Roles & Permissions, Staff Leaves Approvals, and Compliance Audit Trail APIs** within the `hms-admin` microservice.

---

## Global Environment & Configuration
* **Base URL**: `https://vo585bjv4a.execute-api.ap-south-1.amazonaws.com/dev`
* **Authentication**: Bearer Token (Cognito / HMS JWT Access Token)

---

## 1. Staff & Permissions Management

### 1.1 GET List Staff
Retrieve a list of all active staff members registered at the branch.

* **Method**: `GET`
* **URL**: `/admin/staff`
* **Query Parameters**:
  * `page` (*Optional*, Default: `1`): Current page offset.
  * `page_size` (*Optional*, Default: `10`): Items limit.
  * `role_name` (*Optional*): Filter by specific role (e.g. `'DOCTOR'`, `'NURSE'`, `'PHARMACIST'`).
  * `search` (*Optional*): Search query matching staff name or email.

#### Response Body (`200 OK`)
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
        "created_at": "2026-06-15T12:00:00.000Z"
      }
    ],
    "total": 1
  }
}
```

---

### 1.2 POST Assign Staff Permissions
Assign custom security permission overrides to a specific staff user.

* **Method**: `POST`
* **URL**: `/admin/staff/{user_id}/permissions`
* **Path Parameters**:
  * `user_id`: UUID of the target staff user.

#### Request Body
```json
{
  "permissions": ["PATIENT_DELETE", "PROCUREMENT_ORDER_APPROVE"]
}
```

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "message": "Custom user permissions assigned successfully"
}
```

---

## 2. Leaves Approval Workflows

### 2.1 GET List Pending Leaves
Retrieve a list of all pending staff leave requests requiring approval.

* **Method**: `GET`
* **URL**: `/admin/staff/leaves/pending`
* **Query Parameters**:
  * `page` (*Optional*, Default: `1`): Page offset.
  * `page_size` (*Optional*, Default: `10`): Page size limit.

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "leave_id": "l1-ffeee23d-9a29-454f-9f46-192f1aaab285",
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

### 2.2 PATCH Approve Leave Request
Approve a pending leave application.

* **Method**: `PATCH`
* **URL**: `/admin/staff/leaves/{leave_id}/approve`
* **Path Parameters**:
  * `leave_id`: UUID of the leave application.

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "message": "Leave application approved successfully",
  "data": {
    "leave_id": "l1-ffeee23d-9a29-454f-9f46-192f1aaab285",
    "status": "APPROVED"
  }
}
```

---

## 3. Compliance & Auditing

### 3.1 GET List Audit Logs
Retrieve the branch administrative audit logs list.

* **Method**: `GET`
* **URL**: `/admin/audit-logs`
* **Query Parameters**:
  * `page` (*Optional*, Default: `1`): Page number.
  * `page_size` (*Optional*, Default: `20`): Page size limit.
  * `user_id` (*Optional*): Filter logs by operator user ID (UUID).
  * `action` (*Optional*): Filter by specific event type (e.g. `staff.create`, `bed.status_update`).

#### Response Body (`200 OK`)
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
