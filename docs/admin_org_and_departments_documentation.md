# Admin Organization Operations & Department Setup API Documentation

This document provides exhaustive, production-grade specifications for **Hospital/Organization Profile Management** and **Department Setup & Operations** within the `hms-admin` microservice.

---

## Global Environment & Configuration

* **Base URL**: `https://4t1eo222cl.execute-api.ap-south-1.amazonaws.com/dev`
* **Authentication**: Bearer Token (Cognito / HMS JWT Access Token)
* **Branch Resolution**: Pass header `X-Branch-ID` to explicitly override the target branch. If omitted, the request defaults to the caller's active session branch.

---

## 1. Hospital / Organization Profile Operations

The Hospital / Organization Profile APIs allow self-service setup and configuration of identity credentials, contact details, regional settings, and Base64 logo.

### Global Request Headers

| Header Name | Type | Constraint | Description |
| :--- | :--- | :--- | :--- |
| `Authorization` | String | **Required** | `Bearer <access_token>` |
| `Content-Type` | String | **Required** (for POST/PATCH) | `application/json` |
| `X-Branch-ID` | UUID | *Optional* | Target facility/branch ID override |

---

### 1.1 GET Hospital Profile
Retrieve complete identity, contact info, address, regional preferences, and metadata.

* **Method**: `GET`
* **URL**: `/admin/hospital-profile`
* **Aliases**: `/admin/organization/profile`, `/admin/organizations`
* **Request Body**: *None*

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "hp-001",
    "org_id": "589b018a-f8ed-4388-b49e-cfe46fb2d3f8",
    "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "logo_base64": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAA...",
    "organization_name": "Sai LJB Healthcare",
    "hospital_code": "SAI-LJB",
    "hospital_type": "Multispecialty",
    "branch_name": "Main Campus",
    "registration_number": "HLTH-2024-001",
    "established_year": "2010",
    "nabh_nabl_accreditation_number": "NABH-2024-001",
    "gst_number": "07AAAAA0000A1Z5",
    "pan_number": "AAAAA0000A",
    "hospital_website": "https://sailjbcare.com",
    "is_active": true,
    "official_email": "admin@sailjbcare.com",
    "support_email": "support@sailjbcare.com",
    "primary_phone": "+91 98765 43210",
    "emergency_contact": "+91 00000 88888",
    "fax_number": "+91 11 2345 6789",
    "customer_care_number": "1800 123 4567",
    "website": "https://sailjbcare.com",
    "admin_user_email": "admin@sailjbcare.com",
    "address_line1": "123, Hospital Road, Sector 18",
    "address_line2": "Near City Center, Main Market",
    "city": "New Delhi",
    "district": "Central Delhi",
    "state": "Delhi",
    "country": "India",
    "postal_code": "110001",
    "google_maps_link": "https://maps.google.com/?q=28.6139,77.2090",
    "time_zone": "Asia/Kolkata",
    "currency": "INR (₹)",
    "language": "English",
    "fiscal_year": "April - March",
    "date_format": "DD/MM/YYYY",
    "time_format": "24-Hour",
    "week_starts_on": "Monday",
    "number_format": "1,23,456.78",
    "default_country": "India",
    "default_state": "Delhi",
    "system_version": "v2.4.1",
    "license_type": "Enterprise",
    "environment": "Production",
    "created_at": "2026-01-15T00:00:00Z",
    "updated_at": "2026-07-24T15:10:00Z"
  }
}
```

---

### 1.2 POST Create / Initialize Hospital Profile
Create a new hospital profile record.

* **Method**: `POST`
* **URL**: `/admin/hospital-profile`

#### Request Body Field Specifications

| Field Name | Data Type | Constraint | Default / Notes |
| :--- | :--- | :--- | :--- |
| `organization_name` | String | *Optional* | Legal entity name |
| `hospital_code` | String | *Optional* | Short code ID |
| `hospital_type` | String | *Optional* | Multispecialty, Super Specialty |
| `branch_name` | String | *Optional* | Main Campus, North Branch |
| `registration_number` | String | *Optional* | License number |
| `established_year` | String | *Optional* | Foundation year |
| `nabh_nabl_accreditation_number` | String | *Optional* | Accreditation certificate |
| `gst_number` | String | *Optional* | Tax GSTIN |
| `pan_number` | String | *Optional* | PAN tax ID |
| `logo_base64` | String | *Optional* | Encoded logo string |
| `official_email` | String | *Optional* | Primary contact email |
| `support_email` | String | *Optional* | Patient support email |
| `primary_phone` | String | *Optional* | Main office phone |
| `emergency_contact` | String | *Optional* | 24x7 helpline |
| `fax_number` | String | *Optional* | Fax contact |
| `customer_care_number` | String | *Optional* | Toll-free number |
| `website` | String | *Optional* | Hospital website URL |
| `admin_user_email` | String | *Optional* | Primary admin email |
| `address_line1` | String | *Optional* | Street address line 1 |
| `address_line2` | String | *Optional* | Street address line 2 |
| `city` | String | *Optional* | City name |
| `district` | String | *Optional* | District name |
| `state` | String | *Optional* | State |
| `country` | String | *Optional* | Country |
| `postal_code` | String | *Optional* | Postal PIN code |
| `google_maps_link` | String | *Optional* | Google Maps URL |
| `time_zone` | String | *Optional* | Default: `"Asia/Kolkata"` |
| `currency` | String | *Optional* | Default: `"INR (₹)"` |
| `language` | String | *Optional* | Default: `"English"` |
| `fiscal_year` | String | *Optional* | Default: `"April - March"` |
| `date_format` | String | *Optional* | Default: `"DD/MM/YYYY"` |
| `time_format` | String | *Optional* | Default: `"24-Hour"` |
| `week_starts_on` | String | *Optional* | Default: `"Monday"` |
| `number_format` | String | *Optional* | Default: `"1,23,456.78"` |
| `default_country` | String | *Optional* | Default: `"India"` |
| `default_state` | String | *Optional* | Default: `"Delhi"` |

#### Sample Request Payload
```json
{
  "logo_base64": null,
  "organization_name": "Sai LJB Healthcare",
  "hospital_code": "SAI-LJB",
  "hospital_type": "Multispecialty",
  "branch_name": "Main Campus",
  "registration_number": "HLTH-2024-001",
  "established_year": "2010",
  "official_email": "admin@sailjbcare.com",
  "primary_phone": "+91 98765 43210",
  "emergency_contact": "+91 00000 88888",
  "address_line1": "123, Hospital Road, Sector 18",
  "city": "New Delhi",
  "postal_code": "110001",
  "time_zone": "Asia/Kolkata",
  "currency": "INR (₹)"
}
```

#### Sample Response Body (`201 Created`)
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "e8a9103c-7890-4123-ab45-67890abcdef1",
    "organization_name": "Sai LJB Healthcare",
    "hospital_code": "SAI-LJB",
    "hospital_type": "Multispecialty",
    "official_email": "admin@sailjbcare.com",
    "is_active": true,
    "created_at": "2026-07-24T15:20:00Z",
    "updated_at": "2026-07-24T15:20:00Z"
  }
}
```

---

### 1.3 PATCH Update Hospital Profile (Self-Service)
Modify any subset of hospital configuration parameters. Automatically updates `platform.organizations` if `organization_name` is changed.

* **Method**: `PATCH`
* **URL**: `/admin/hospital-profile`

#### Sample Request Body
```json
{
  "organization_name": "Sai LJB Healthcare & Research Center",
  "official_email": "contact@sailjbcare.com",
  "primary_phone": "+91 98765 00000",
  "address_line1": "456, Health Expressway, Tech Zone",
  "postal_code": "110002"
}
```

#### Sample Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "hp-001",
    "organization_name": "Sai LJB Healthcare & Research Center",
    "official_email": "contact@sailjbcare.com",
    "primary_phone": "+91 98765 00000",
    "address_line1": "456, Health Expressway, Tech Zone",
    "postal_code": "110002",
    "updated_at": "2026-07-24T15:21:00Z"
  }
}
```

---

## 2. Department Setup & Operations

The Department Setup APIs manage clinical and diagnostic departments, staff assignments, location mapping, operating hours, and operational status (e.g. `Under Maintenance`).

---

### 2.1 GET Department Summary Cards (KPIs)
Retrieve high-level operational statistics displayed at the top of the Department Setup UI.

* **Method**: `GET`
* **URL**: `/admin/departments/summary`
* **Request Body**: *None*

#### Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "total_departments": 18,
    "active_departments": 14,
    "department_heads_assigned": 16,
    "under_maintenance": 2
  }
}
```

---

### 2.2 GET List Departments
Lists departments with staff counts (doctors, nurses, support), location, operating hours, and operational status.

* **Method**: `GET`
* **URL**: `/admin/departments`
* **Query Parameters**:
  * `search` (*Optional*): Search term (matches department name or code)
  * `status` (*Optional*): Filter by status (`Active`, `Under Maintenance`, `Inactive`)
  * `page` (*Optional*): Page number (Default: `1`)
  * `page_size` (*Optional*): Page size (Default: `10`)

#### Sample Request
```http
GET /admin/departments?status=Active&page=1&page_size=10 HTTP/1.1
Authorization: Bearer YOUR_ACCESS_TOKEN
```

#### Sample Response Body (`200 OK`)
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
        "created_at": "2026-01-15T00:00:00Z"
      },
      {
        "department_id": "1b9c2703-ed5a-4041-a0ca-b4a73b64bb71",
        "department_name": "Pulmonology",
        "code": "PULM-007",
        "department_type": "Clinical",
        "head_of_department": {
          "doctor_id": "8a7c2601-ab5d-4032-9bca-c4a63b74bb80",
          "name": "Dr. Anand Iyer",
          "title": "Respiratory Medicine Chief"
        },
        "building_block": "A Block",
        "floor": "2nd Floor",
        "wing": "North Wing",
        "rooms": "Rooms 221-225",
        "staff_breakdown": {
          "doctors_count": 6,
          "nurses_count": 10,
          "support_staff_count": 4,
          "rooms_count": 5
        },
        "operating_hours": "09:00 AM - 06:00 PM",
        "status": "Under Maintenance",
        "is_active": true,
        "created_at": "2026-01-15T00:00:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "page_size": 10,
      "total_records": 18,
      "total_pages": 2
    }
  }
}
```

---

### 2.3 POST Create Department
Add a new department to the facility.

* **Method**: `POST`
* **URL**: `/admin/departments`

#### Request Body Field Specifications

| Field Name | Data Type | Constraint | Description |
| :--- | :--- | :--- | :--- |
| `department_name` | String | **Required** (2-120 chars) | Full department title |
| `code` | String | *Optional* | Short code (e.g. `CARD-001`) |
| `department_type` | String | *Optional* | `Clinical`, `Diagnostic`, `Emergency` |
| `building_block` | String | *Optional* | Building / Block name (e.g. `A Block`) |
| `floor` | String | *Optional* | Floor label (e.g. `2nd Floor`) |
| `wing` | String | *Optional* | Wing label (e.g. `East Wing`) |
| `rooms` | String | *Optional* | Room numbers range (e.g. `Rooms 201-210`) |
| `operating_hours` | String | *Optional* | Working hours (e.g. `09:00 AM - 06:00 PM`) |
| `head_of_department_id` | UUID | *Optional* | User ID of department head doctor |
| `head_title` | String | *Optional* | Title (e.g. `Chief of Cardiology`) |
| `status` | String | *Optional* | `Active`, `Under Maintenance`, `Inactive` |

#### Sample Request Body
```json
{
  "department_name": "Emergency Medicine",
  "code": "EM-003",
  "department_type": "Emergency",
  "building_block": "Emergency Wing",
  "floor": "Ground Floor",
  "wing": "Gate 1",
  "rooms": "Rooms E1-E12",
  "head_of_department_id": "7ca9103c-7890-4123-ab45-67890abcdef9",
  "head_title": "ER HOD & Trauma Lead",
  "operating_hours": "24 Hours",
  "status": "Active"
}
```

#### Sample Response Body (`201 Created`)
```json
{
  "success": true,
  "code": 201,
  "data": {
    "department_id": "f5c9103c-7890-4123-ab45-67890abcdef3",
    "department_name": "Emergency Medicine",
    "code": "EM-003",
    "status": "Active",
    "is_active": true,
    "created_at": "2026-07-24T15:22:00Z"
  }
}
```

---

### 2.4 GET Single Department Details
Retrieve detailed profile of a specific department.

* **Method**: `GET`
* **URL**: `/admin/departments/{department_id}`

#### Sample Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "department_id": "0fbfdbef-9873-4824-b1fd-6fb827d3ba57",
    "department_name": "Cardiology",
    "code": "CARD-001",
    "department_type": "Clinical",
    "head_of_department": {
      "doctor_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
      "name": "Dr. Arjun Nair",
      "title": "Chief of Cardiology"
    },
    "location": {
      "block": "A Block",
      "floor": "2nd Floor",
      "wing": "East Wing",
      "rooms": "Rooms 201-210"
    },
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
    "updated_at": "2026-07-24T15:10:00Z"
  }
}
```

---

### 2.5 PATCH Modify Department / Set Under Maintenance
Update department configuration, change operating hours, reassign head of department, or toggle status to **Under Maintenance**.

* **Method**: `PATCH`
* **URL**: `/admin/departments/{department_id}`

#### Example A: Put Department Under Maintenance
```json
{
  "status": "Under Maintenance"
}
```

#### Example B: Restore Department to Active and Update Hours
```json
{
  "status": "Active",
  "operating_hours": "08:00 AM - 08:00 PM"
}
```

#### Sample Response Body (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "department_id": "1b9c2703-ed5a-4041-a0ca-b4a73b64bb71",
    "department_name": "Pulmonology",
    "status": "Under Maintenance",
    "is_active": true,
    "updated_at": "2026-07-24T15:23:00Z"
  }
}
```

---

## 3. Summary Matrix of Endpoints

| Operations | Method | URL Path | Description |
| :--- | :--- | :--- | :--- |
| **Get Profile** | `GET` | `/admin/hospital-profile` | Fetch hospital profile & identity |
| **Init Profile** | `POST` | `/admin/hospital-profile` | Initialize hospital profile record |
| **Update Profile** | `PATCH` | `/admin/hospital-profile` | Self-service update hospital profile |
| **Dept KPIs** | `GET` | `/admin/departments/summary` | Top KPI statistics cards |
| **List Depts** | `GET` | `/admin/departments` | List departments with location & staff |
| **Create Dept** | `POST` | `/admin/departments` | Add new clinical/diagnostic department |
| **Get Dept Details**| `GET` | `/admin/departments/{id}` | Single department detail lookup |
| **Update / Maintenance** | `PATCH` | `/admin/departments/{id}` | Modify department or set to Under Maintenance |
