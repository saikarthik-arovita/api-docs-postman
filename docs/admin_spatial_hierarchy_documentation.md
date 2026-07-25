# Admin Spatial Hierarchy (Floors, Blocks, Wards, Units, Beds) API Documentation

This document provides exhaustive, production-grade specifications for managing the spatial and operational asset hierarchy (**Floors ➔ Blocks ➔ Wards ➔ Units ➔ Beds**) within the `hms-admin` microservice.

---

## Global Environment & Configuration

* **Base URL**: `https://vo585bjv4a.execute-api.ap-south-1.amazonaws.com/dev`
* **Authentication**: Bearer Token (Cognito / HMS JWT Access Token)
* **Branch Resolution**: Pass header `X-Branch-ID` to explicitly override the target branch. If omitted, the request defaults to the caller's active session branch.

### Global Request Headers

| Header Name | Type | Constraint | Description |
| :--- | :--- | :--- | :--- |
| `Authorization` | String | **Required** | `Bearer <access_token>` |
| `Content-Type` | String | **Required** (for POST/PATCH) | `application/json` |
| `X-Branch-ID` | UUID | *Optional* | Target facility/branch ID override |

---

## Data Type Definitions

### `WardType` (Literal String)
`"GENERAL"`, `"PRIVATE"`, `"ICU"`, `"HDU"`, `"EMERGENCY"`, `"MATERNITY"`, `"PAEDIATRIC"`, `"SURGICAL"`, `"ISOLATION"`, `"OT"`, `"NICU"`, `"PICU"`, `"OBSERVATION"`, `"DAY_CARE"`, `"SEMI_PRIVATE"`, `"OTHER"`

### `BedType` (Literal String)
`"STANDARD"`, `"ICU"`, `"HDU"`, `"ISOLATION"`, `"RECOVERY"`, `"MATERNITY"`, `"EMERGENCY"`, `"NICU"`, `"PICU"`, `"SEMI_PRIVATE"`, `"PRIVATE"`, `"DAY_CARE"`, `"OTHER"`

### `BedStatus` (Literal String)
`"AVAILABLE"`, `"OCCUPIED"`, `"RESERVED"`, `"MAINTENANCE"`

---

## 1. Floors API

### 1.1 GET List Floors
Retrieve a list of all floors registered under the active branch.

* **Method**: `GET`
* **URL**: `/admin/floors`
* **Query Parameters**:
  * `include_inactive` (*Optional*): `true` or `false` (Default: `false`)

#### Response Body (`200 OK`)
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

### 1.2 POST Create Floor
Add a new floor to the facility.

* **Method**: `POST`
* **URL**: `/admin/floors`

#### Request Body
```json
{
  "floor_number": 2,
  "floor_label": "First Floor"
}
```
* **Constraints**:
  * `floor_number`: Integer, must be between `-5` and `120` (Required, unique per branch).
  * `floor_label`: String, between `1` and `50` characters (Required).

#### Response Body (`201 Created`)
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "39ff007e-1daf-4c5f-87a7-be8f712a2e46",
    "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "floor_number": 2,
    "floor_label": "First Floor",
    "is_active": true,
    "created_at": "2026-07-25T11:30:00.000Z"
  }
}
```

---

### 1.3 GET Floor Details
Retrieve details of a single floor.

* **Method**: `GET`
* **URL**: `/admin/floors/{floor_id}`

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "39ff007e-1daf-4c5f-87a7-be8f712a2e46",
    "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "floor_number": 2,
    "floor_label": "First Floor",
    "is_active": true,
    "created_at": "2026-07-25T11:30:00.000Z"
  }
}
```

---

### 1.4 PATCH Update Floor
Update an existing floor's attributes.

* **Method**: `PATCH`
* **URL**: `/admin/floors/{floor_id}`

#### Request Body
```json
{
  "floor_label": "First Floor (Modified)",
  "is_active": true
}
```
* **Constraints**:
  * `floor_number`: Integer, between `-5` and `120` (Optional).
  * `floor_label`: String, between `1` and `50` characters (Optional).
  * `is_active`: Boolean (Optional).

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "39ff007e-1daf-4c5f-87a7-be8f712a2e46",
    "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "floor_number": 2,
    "floor_label": "First Floor (Modified)",
    "is_active": true,
    "created_at": "2026-07-25T11:30:00.000Z"
  }
}
```

---

## 2. Blocks API

### 2.1 GET List Blocks
Retrieve all building blocks under the branch.

* **Method**: `GET`
* **URL**: `/admin/blocks`
* **Query Parameters**:
  * `floor_id` (*Optional*): Filter blocks by parent floor ID (UUID).
  * `include_inactive` (*Optional*): Include blocks where `is_active` is `false` (Default: `false`).

#### Response Body (`200 OK`)
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

### 2.2 POST Create Block
Create a new building block.

* **Method**: `POST`
* **URL**: `/admin/blocks`

#### Request Body
```json
{
  "floor_id": "93920321-63e8-4646-ba4f-dc976ec6dfda",
  "name": "General Block A",
  "code": "BLK-A",
  "description": "Primary clinical inpatient tower block"
}
```
* **Constraints**:
  * `floor_id`: UUID of parent floor (Required).
  * `name`: String, length `2` to `100` characters (Required).
  * `code`: String, unique identifier for the block, length `1` to `50` characters (Required).
  * `description`: String, length up to `500` characters (Optional).

#### Response Body (`201 Created`)
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "8c59f032-47ef-4e1b-b46e-1d54238e4a90",
    "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "floor_id": "93920321-63e8-4646-ba4f-dc976ec6dfda",
    "name": "General Block A",
    "code": "BLK-A",
    "description": "Primary clinical inpatient tower block",
    "is_active": true,
    "created_at": "2026-07-25T11:32:00.000Z"
  }
}
```

---

### 2.3 GET Block Details
Retrieve a single block's profile.

* **Method**: `GET`
* **URL**: `/admin/blocks/{block_id}`

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "8c59f032-47ef-4e1b-b46e-1d54238e4a90",
    "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "floor_id": "93920321-63e8-4646-ba4f-dc976ec6dfda",
    "name": "General Block A",
    "code": "BLK-A",
    "description": "Primary clinical inpatient tower block",
    "is_active": true,
    "created_at": "2026-07-25T11:32:00.000Z"
  }
}
```

---

### 2.4 PATCH Update Block
Modify a block's metadata.

* **Method**: `PATCH`
* **URL**: `/admin/blocks/{block_id}`

#### Request Body
```json
{
  "name": "General Block A (Renamed)",
  "description": "Updated tower block details"
}
```
* **Constraints**:
  * `floor_id`: UUID (Optional).
  * `name`: String, length `2` to `100` characters (Optional).
  * `code`: String, length `1` to `50` characters (Optional).
  * `description`: String, length up to `500` characters (Optional).
  * `is_active`: Boolean (Optional).

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "8c59f032-47ef-4e1b-b46e-1d54238e4a90",
    "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "floor_id": "93920321-63e8-4646-ba4f-dc976ec6dfda",
    "name": "General Block A (Renamed)",
    "code": "BLK-A",
    "description": "Updated tower block details",
    "is_active": true,
    "created_at": "2026-07-25T11:32:00.000Z"
  }
}
```

---

## 3. Wards & Units API

### 3.1 GET List Wards
Retrieve a list of wards.

* **Method**: `GET`
* **URL**: `/admin/wards`
* **Query Parameters**:
  * `include_inactive` (*Optional*): Include inactive records (Default: `false`).
  * `floor` (*Optional*): Filter by floor string identifier.
  * `ward_type` (*Optional*): Filter by `WardType` value.
  * `work_area_id` (*Optional*): Filter by mapped work area ID (UUID).

#### Response Body (`200 OK`)
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
        "work_area_code": "WA-MED-M",
        "work_area_name": "Male Medical Area",
        "ward_area": "Inpatient Wing 1",
        "ward_area_id": "bb30b4b7-0da8-4b9d-8643-4fdffefabe9d",
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

### 3.2 POST Create Ward
Create a new ward.

* **Method**: `POST`
* **URL**: `/admin/wards`

#### Request Body
```json
{
  "name": "Male Medical Ward",
  "ward_type": "GENERAL",
  "capacity": 30,
  "work_area_id": "aa30b4b7-0da8-4b9d-8643-4fdffefabe9d",
  "floor": "Ground Floor",
  "block_id": "8c59f032-47ef-4e1b-b46e-1d54238e4a90"
}
```
* **Constraints**:
  * `name`: String, length `2` to `100` characters (Required).
  * `ward_type`: `WardType` value (Required).
  * `capacity`: Integer, between `1` and `500` (Required).
  * `work_area_id`: UUID (Optional).
  * `floor`: String, length up to `10` characters (Optional).
  * `block_id`: UUID (Optional).

#### Response Body (`201 Created`)
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "673f8a09-7d8b-402a-a92c-7b2496a7d5ea",
    "tenant_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "name": "Male Medical Ward",
    "ward_type": "GENERAL",
    "work_area_id": "aa30b4b7-0da8-4b9d-8643-4fdffefabe9d",
    "work_area_code": "WA-MED-M",
    "work_area_name": "Male Medical Area",
    "ward_area": "Inpatient Wing 1",
    "ward_area_id": "bb30b4b7-0da8-4b9d-8643-4fdffefabe9d",
    "floor": "Ground Floor",
    "block_id": "8c59f032-47ef-4e1b-b46e-1d54238e4a90",
    "capacity": 30,
    "is_active": true,
    "created_at": "2026-07-25T11:35:00.000Z",
    "total_beds": 0,
    "available_beds": 0,
    "occupied_beds": 0
  }
}
```

---

### 3.3 GET Ward Details
Retrieve details of a single ward.

* **Method**: `GET`
* **URL**: `/admin/wards/{ward_id}`

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "673f8a09-7d8b-402a-a92c-7b2496a7d5ea",
    "tenant_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "name": "Male Medical Ward",
    "ward_type": "GENERAL",
    "work_area_id": "aa30b4b7-0da8-4b9d-8643-4fdffefabe9d",
    "work_area_code": "WA-MED-M",
    "work_area_name": "Male Medical Area",
    "floor": "Ground Floor",
    "block_id": "8c59f032-47ef-4e1b-b46e-1d54238e4a90",
    "capacity": 30,
    "is_active": true,
    "created_at": "2026-07-25T11:35:00.000Z",
    "total_beds": 30,
    "available_beds": 25,
    "occupied_beds": 5
  }
}
```

---

### 3.4 PATCH Update Ward
Modify ward operational capacity or location details.

* **Method**: `PATCH`
* **URL**: `/admin/wards/{ward_id}`

#### Request Body
```json
{
  "name": "Male Medical Ward (Renamed)",
  "capacity": 35
}
```
* **Constraints**:
  * `name`: String, length `2` to `100` characters (Optional).
  * `ward_type`: `WardType` value (Optional).
  * `work_area_id`: UUID (Optional).
  * `floor`: String, length up to `10` characters (Optional).
  * `block_id`: UUID (Optional).
  * `capacity`: Integer, between `1` and `500` (Optional).
  * `is_active`: Boolean (Optional).

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "673f8a09-7d8b-402a-a92c-7b2496a7d5ea",
    "tenant_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "name": "Male Medical Ward (Renamed)",
    "ward_type": "GENERAL",
    "work_area_id": "aa30b4b7-0da8-4b9d-8643-4fdffefabe9d",
    "floor": "Ground Floor",
    "block_id": "8c59f032-47ef-4e1b-b46e-1d54238e4a90",
    "capacity": 35,
    "is_active": true,
    "created_at": "2026-07-25T11:35:00.000Z"
  }
}
```

---

### 3.5 GET List Units Mapped to Ward
Retrieve a list of ward sub-units/beds groups.

* **Method**: `GET`
* **URL**: `/admin/wards/{ward_id}/units`

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "id": "a901f41b-4fef-48a1-b84e-8664fd37a912",
      "ward_id": "673f8a09-7d8b-402a-a92c-7b2496a7d5ea",
      "ward_name": "Male Medical Ward",
      "name": "Bay A",
      "code": "BAY-A",
      "description": "Standard medical inpatient bay",
      "capacity": 6,
      "is_active": true,
      "created_at": "2026-07-04T12:55:00.000Z"
    }
  ]
}
```

---

### 3.6 GET List All Units
Retrieve paginated unit allocations across all wards.

* **Method**: `GET`
* **URL**: `/admin/units`
* **Query Parameters**:
  * `ward_id` (*Optional*): Filter units by parent ward ID (UUID).
  * `is_active` (*Optional*): Filter by status (`true` / `false`).
  * `page` (*Optional*): Zero-based page offset (Default: `0`).
  * `size` (*Optional*): Page capacity, max `100` (Default: `10`).

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "content": [
      {
        "id": "a901f41b-4fef-48a1-b84e-8664fd37a912",
        "ward_id": "673f8a09-7d8b-402a-a92c-7b2496a7d5ea",
        "ward_name": "Male Medical Ward",
        "name": "Bay A",
        "code": "BAY-A",
        "capacity": 6,
        "is_active": true
      }
    ],
    "total_elements": 1,
    "page": 0,
    "size": 10
  }
}
```

---

### 3.7 POST Create Unit
Allocate a new unit segment inside a ward.

* **Method**: `POST`
* **URL**: `/admin/units`

#### Request Body
```json
{
  "ward_id": "673f8a09-7d8b-402a-a92c-7b2496a7d5ea",
  "name": "Bay A",
  "code": "BAY-A",
  "description": "Standard medical inpatient bay",
  "capacity": 6
}
```
* **Constraints**:
  * `ward_id`: UUID of parent ward (Required).
  * `name`: String, length `1` to `120` characters (Required).
  * `code`: String, length up to `50` characters (Optional).
  * `description`: String, length up to `500` characters (Optional).
  * `capacity`: Integer, between `0` and `10000` (Default: `0`).

#### Response Body (`201 Created`)
```json
{
  "success": true,
  "code": 201,
  "message": "Ward unit created successfully",
  "data": {
    "id": "a901f41b-4fef-48a1-b84e-8664fd37a912",
    "ward_id": "673f8a09-7d8b-402a-a92c-7b2496a7d5ea",
    "ward_name": "Male Medical Ward",
    "name": "Bay A",
    "code": "BAY-A",
    "description": "Standard medical inpatient bay",
    "capacity": 6,
    "is_active": true,
    "created_at": "2026-07-25T11:40:00.000Z"
  }
}
```

---

## 4. Beds API

### 4.1 GET Beds Summary
Retrieve aggregated bed occupancy, availability statistics, and ward metrics.

* **Method**: `GET`
* **URL**: `/admin/beds/summary`

#### Response Body (`200 OK`)
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
        "maintenance": 0,
        "work_area_id": "aa30b4b7-0da8-4b9d-8643-4fdffefabe9d",
        "work_area_code": "WA-MED-M",
        "work_area_name": "Male Medical Area",
        "ward_area": "Inpatient Wing 1"
      }
    ]
  }
}
```

---

### 4.2 GET List Beds
Retrieve a paginated, filterable list of all beds.

* **Method**: `GET`
* **URL**: `/admin/beds`
* **Query Parameters**:
  * `page` (*Optional*): Page offset integer (Default: `1`).
  * `page_size` (*Optional*): Items per page, max `100` (Default: `20`).
  * `ward_id` (*Optional*): Filter by parent ward ID (UUID).
  * `status` (*Optional*): Filter by `BedStatus` value.
  * `bed_status` (*Optional*): Filter by `BedStatus` value (alias for `status`).
  * `bed_type` (*Optional*): Filter by `BedType` value.
  * `ward_type` (*Optional*): Filter by `WardType` value.
  * `search` (*Optional*): String search matched against bed number or ward name.
  * `floor` (*Optional*): Filter by floor string identifier.
  * `include_inactive` (*Optional*): Include inactive beds (Default: `false`).

#### Response Body (`200 OK`)
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
        "created_at": "2026-07-04T13:00:00.000Z",
        "current_patient_id": "ffeee23d-9a29-454f-9f46-192f1aaab285",
        "current_patient_name": "John Doe",
        "current_patient_uhid": "UHID-1002345",
        "current_admission_id": "d7815663-9c5a-41f1-8d65-60cf9ae0d50d"
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

### 4.3 POST Create Bed
Add a new physical bed.

* **Method**: `POST`
* **URL**: `/admin/beds`

#### Request Body
```json
{
  "ward_id": "673f8a09-7d8b-402a-a92c-7b2496a7d5ea",
  "bed_number": "B-102",
  "bed_type": "STANDARD"
}
```
* **Constraints**:
  * `ward_id`: UUID of parent ward (Required).
  * `bed_number`: String identifier, length `1` to `20` characters (Required, unique per ward).
  * `bed_type`: `BedType` value (Default: `"STANDARD"`).

#### Response Body (`201 Created`)
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "f89b018a-f8ed-4388-b49e-cfe46fb2d3f9",
    "tenant_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "ward_id": "673f8a09-7d8b-402a-a92c-7b2496a7d5ea",
    "ward_name": "Male Medical Ward",
    "floor": "Ground Floor",
    "bed_number": "B-102",
    "bed_type": "STANDARD",
    "status": "AVAILABLE",
    "is_active": true,
    "created_at": "2026-07-25T11:42:00.000Z"
  }
}
```

---

### 4.4 GET Bed Details
Retrieve details of a single bed.

* **Method**: `GET`
* **URL**: `/admin/beds/{bed_id}`

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "f89b018a-f8ed-4388-b49e-cfe46fb2d3f9",
    "tenant_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "ward_id": "673f8a09-7d8b-402a-a92c-7b2496a7d5ea",
    "ward_name": "Male Medical Ward",
    "floor": "Ground Floor",
    "bed_number": "B-102",
    "bed_type": "STANDARD",
    "status": "AVAILABLE",
    "is_active": true,
    "created_at": "2026-07-25T11:42:00.000Z"
  }
}
```

---

### 4.5 PATCH Update Bed
Update bed configurations like number, classification, or active state.

* **Method**: `PATCH`
* **URL**: `/admin/beds/{bed_id}`

#### Request Body
```json
{
  "bed_number": "B-102-Mod",
  "bed_type": "ICU"
}
```
* **Constraints**:
  * `bed_number`: String, length `1` to `20` characters (Optional).
  * `bed_type`: `BedType` value (Optional).
  * `is_active`: Boolean (Optional).

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "f89b018a-f8ed-4388-b49e-cfe46fb2d3f9",
    "tenant_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "ward_id": "673f8a09-7d8b-402a-a92c-7b2496a7d5ea",
    "ward_name": "Male Medical Ward",
    "floor": "Ground Floor",
    "bed_number": "B-102-Mod",
    "bed_type": "ICU",
    "status": "AVAILABLE",
    "is_active": true,
    "created_at": "2026-07-25T11:42:00.000Z"
  }
}
```

---

### 4.6 PATCH Update Bed Status
Update operational/clinical status of a bed.

* **Method**: `PATCH`
* **URL**: `/admin/beds/{bed_id}/status`

#### Request Body
```json
{
  "status": "MAINTENANCE",
  "reason": "Routine cleaning and minor repairs"
}
```
* **Constraints**:
  * `status`: `BedStatus` value (Required).
  * `reason`: String, length up to `500` characters (Optional).

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "f89b018a-f8ed-4388-b49e-cfe46fb2d3f9",
    "tenant_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "ward_id": "673f8a09-7d8b-402a-a92c-7b2496a7d5ea",
    "ward_name": "Male Medical Ward",
    "floor": "Ground Floor",
    "bed_number": "B-102-Mod",
    "bed_type": "ICU",
    "status": "MAINTENANCE",
    "is_active": true,
    "created_at": "2026-07-25T11:42:00.000Z"
  }
}
```
