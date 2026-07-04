# Ward and Bed Management API Documentation

This document describes the newly implemented backend API endpoints designed for managing wards and beds from the IPD Dashboard Ward card.

---

### 1. Edit Ward Details
Modifies ward configuration metadata such as name, capacity, and gender restrictions.

* **Endpoint**: `PATCH /ipd/wards/{ward_id}`
* **Headers**:
  * `X-Tenant-Id`: *<tenant_uuid>*
* **URL Path Parameters**:
  * `ward_id` (UUID): ID of the ward to update.
* **Request Body** (JSON - all fields optional, at least one required):
  ```json
  {
    "ward_name": "General Ward A - Updated",
    "capacity": 60,
    "gender_restriction": "MALE"
  }
  ```
* **Success Response (200 OK)**:
  ```json
  {
    "success": true,
    "code": 200,
    "data": {
      "id": "ward-uuid",
      "ward_code": "WRD_001",
      "ward_name": "General Ward A - Updated",
      "ward_type": "GENERAL",
      "gender_restriction": "MALE",
      "capacity": 60,
      "is_active": true,
      "total_beds": 50,
      "available_beds": 5,
      "occupied_beds": 42,
      "maintenance_beds": 3,
      "created_at": "2026-03-12T10:00:00Z"
    }
  }
  ```

---

### 2. Block/Unblock Ward
Suspends or resumes admissions into a ward by toggling its active status.

* **Endpoint**: `POST /ipd/wards/{ward_id}/block`
* **Headers**:
  * `X-Tenant-Id`: *<tenant_uuid>*
* **URL Path Parameters**:
  * `ward_id` (UUID): ID of the ward to block/unblock.
* **Request Body** (JSON - Required):
  ```json
  {
    "is_active": false
  }
  ```
* **Success Response (200 OK)**:
  ```json
  {
    "success": true,
    "code": 200,
    "data": {
      "id": "ward-uuid",
      "ward_code": "WRD_001",
      "ward_name": "General Ward A",
      "ward_type": "GENERAL",
      "gender_restriction": null,
      "capacity": 50,
      "is_active": false,
      "created_at": "2026-03-12T10:00:00Z"
    }
  }
  ```

---

### 3. Mark Ward Beds as Under Maintenance
Bulk transitions all unoccupied beds in a specific ward into the `MAINTENANCE` state. Occupied beds are skipped to avoid disrupting admitted patients.

* **Endpoint**: `POST /ipd/wards/{ward_id}/maintenance`
* **Headers**:
  * `X-Tenant-Id`: *<tenant_uuid>*
* **URL Path Parameters**:
  * `ward_id` (UUID): ID of the target ward.
* **Request Body**: None (Empty)
* **Success Response (200 OK)**:
  ```json
  {
    "success": true,
    "code": 200,
    "data": {
      "message": "Beds in ward ward-uuid marked as maintenance"
    }
  }
  ```

---

### 4. Assign (Onboard) Bed
Registers a new physical bed within a ward.

* **Endpoint**: `POST /ipd/beds`
* **Headers**:
  * `X-Tenant-Id`: *<tenant_uuid>*
* **Request Body** (JSON - Required):
  ```json
  {
    "ward_id": "ward-uuid",
    "bed_number": "B-51",
    "bed_type": "STANDARD"
  }
  ```
* **Success Response (200 OK)**:
  ```json
  {
    "success": true,
    "code": 200,
    "data": {
      "id": "new-bed-uuid",
      "branch_id": "tenant-uuid",
      "ward_id": "ward-uuid",
      "bed_number": "B-51",
      "bed_type": "STANDARD",
      "status": "AVAILABLE",
      "is_active": true,
      "created_at": "2026-07-04T11:32:00Z",
      "updated_at": "2026-07-04T11:32:00Z"
    }
  }
  ```

---

### 5. Transfer Bed
Relocates an existing physical bed configuration to another ward.

* **Endpoint**: `POST /ipd/beds/transfer`
* **Headers**:
  * `X-Tenant-Id`: *<tenant_uuid>*
* **Request Body** (JSON - Required):
  ```json
  {
    "bed_id": "bed-uuid",
    "target_ward_id": "new-ward-uuid"
  }
  ```
* **Success Response (200 OK)**:
  ```json
  {
    "success": true,
    "code": 200,
    "data": {
      "id": "bed-uuid",
      "branch_id": "tenant-uuid",
      "ward_id": "new-ward-uuid",
      "bed_number": "B-51",
      "bed_type": "STANDARD",
      "status": "AVAILABLE",
      "is_active": true,
      "created_at": "2026-07-04T11:32:00Z",
      "updated_at": "2026-07-04T11:34:00Z"
    }
  }
  ```
