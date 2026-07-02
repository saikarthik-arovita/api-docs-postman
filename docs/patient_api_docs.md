# HMS — Patient Service API Documentation

This document covers all patient registration, demographic updates, medical histories, vitals recording, insurance policy linkage, attendant configurations, and diagnostic/prescription order audits for the HMS **Patient Service**.

---

## 1. Global Conventions

### Base URL
All requests are sent to the Patient Service API Gateway stage:
```
https://<api-id>.execute-api.ap-south-1.amazonaws.com/<stage>
```

### Authorization Header
All endpoints require a valid JWT Access Token passed in the `Authorization` header:
```http
Authorization: Bearer <access_token>
```

### Universal Response Envelope
All API responses follow the standard envelope format:

**Success Response (200 OK / 201 Created):**
```json
{
  "success": true,
  "data": { ... },
  "message": "Optional descriptive success message"
}
```

**Error Response (4xx / 5xx):**
```json
{
  "success": false,
  "code": 404,
  "message": "Patient not found"
}
```

---

## 2. Patient Registration & Search

### 2.1 Register Patient
Used to register a new patient in the hospital.

* **Endpoint:** `POST /patients/register`
* **Required Permission:** `patients:create`
* **Request Body:**
  | Field | Type | Required? | Description |
  | :--- | :--- | :--- | :--- |
  | `full_name` | String | **Mandatory** | Min length: 2, Max length: 100 characters. |
  | `phone` | String | **Mandatory** | Min length: 10, Max length: 50 characters. Strips non-digits. |
  | `age` | Integer | **Mandatory** | Must be between `0` and `150`. |
  | `dob` | Date | Optional | Format: `YYYY-MM-DD`. |
  | `gender` | String | Optional | Must be: `MALE`, `FEMALE`, `OTHER`, or `UNKNOWN`. |
  | `blood_group` | String | Optional | Must be: `A+`, `A-`, `B+`, `B-`, `AB+`, `AB-`, `O+`, `O-`, or `UNKNOWN`. |
  | `address` | Object | Optional | Nested Address object (line1, city, state, pincode). |
  | `abha_number` | String | Optional | ABHA ID number. Max length: 100 characters. |
  | `abha_address` | String | Optional | PHR Address. Max length: 255 characters. |

* **Example Request:**
```json
{
  "full_name": "Jane Doe",
  "phone": "+919999999999",
  "age": 29,
  "gender": "FEMALE",
  "blood_group": "O+",
  "address": {
    "line1": "Flat 302, Sector 12",
    "city": "Bengaluru",
    "state": "Karnataka",
    "pincode": "560001"
  }
}
```
* **Example Response (201 Created):**
```json
{
  "success": true,
  "message": "Patient registered successfully",
  "data": {
    "id": "e302008e-5b12-421c-a111-9a99fcd23b89",
    "uhid": "PAT-2026-0010",
    "patient_number": "PAT-2026-0010",
    "full_name": "Jane Doe",
    "phone": "+919999999999",
    "age": 29,
    "gender": "FEMALE",
    "blood_group": "O+",
    "address": {
      "line1": "Flat 302, Sector 12",
      "city": "Bengaluru",
      "state": "Karnataka",
      "pincode": "560001"
    },
    "abha_number": null,
    "abha_address": null,
    "is_temporary": false,
    "is_active": true,
    "created_at": "2026-06-23T12:00:00Z",
    "updated_at": "2026-06-23T12:00:00Z"
  }
}
```

---

### 2.2 Search Patients
Search the patient registry using keywords.

* **Endpoint:** `GET /patients/search`
* **Required Permission:** `patients:search`
* **Query Parameters:**
  | Parameter | Type | Required? | Description |
  | :--- | :--- | :--- | :--- |
  | `q` | String | **Mandatory** | Search query string (names, phone numbers, UHID, or MRN) |
  | `page` | Integer | Optional | Defaults to `1` |
  | `page_size` | Integer | Optional | Defaults to `20`, Max `100` |

* **Example Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "e302008e-5b12-421c-a111-9a99fcd23b89",
        "uhid": "PAT-2026-0010",
        "full_name": "Jane Doe",
        "phone": "+919999999999",
        "gender": "FEMALE",
        "is_active": true
      }
    ],
    "total": 1,
    "page": 1,
    "page_size": 20
  }
}
```

---

### 2.3 List Patients
Retrieves a paginated list of patients filtered by demographic or visit attributes.

* **Endpoint:** `GET /patients`
* **Required Permission:** `patients:view`
* **Query Parameters:**
  | Parameter | Type | Required? | Description |
  | :--- | :--- | :--- | :--- |
  | `q` | String | Optional | Keyword search. |
  | `status` | String | Optional | Filter by patient/visit lifecycle status. |
  | `department_id` | UUID | Optional | Filter by department context. |
  | `doctor_id` | UUID | Optional | Filter by attending doctor. |
  | `converted_only` | Boolean | Optional | Filter only converted patients. |
  | `page` | Integer | Optional | Defaults to `1`. |
  | `page_size` | Integer | Optional | Defaults to `20`. |

* **Example Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "e302008e-5b12-421c-a111-9a99fcd23b89",
        "uhid": "PAT-2026-0010",
        "full_name": "Jane Doe",
        "phone": "+919999999999",
        "gender": "FEMALE",
        "is_active": true
      }
    ],
    "total": 1,
    "page": 1,
    "page_size": 20
  }
}
```

---

### 2.4 Get / Update Patient Details

#### Get Patient Details
* **Endpoint:** `GET /patients/{patient_id}`
* **Required Permission:** `patients:view`

* **Example Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "e302008e-5b12-421c-a111-9a99fcd23b89",
    "uhid": "PAT-2026-0010",
    "patient_number": "PATNO-2026-0010",
    "full_name": "Jane Doe",
    "phone": "+919999999999",
    "age": 29,
    "gender": "FEMALE",
    "blood_group": "O+",
    "address": {
      "line1": "Flat 302, Sector 12",
      "city": "Bengaluru",
      "state": "Karnataka",
      "pincode": "560001"
    },
    "is_active": true,
    "created_at": "2026-06-23T12:00:00Z"
  }
}
```

#### Update Patient Details
Allows updating one or more fields.

* **Endpoint:** `PATCH /patients/{patient_id}`
* **Required Permission:** `patients:edit`
* **Request Body:**
```json
{
  "phone": "+918888888888",
  "address": {
    "line1": "Flat 505, Sector 15",
    "city": "Bengaluru",
    "state": "Karnataka",
    "pincode": "560015"
  }
}
```
* **Example Response (200 OK):**
```json
{
  "success": true,
  "message": "Patient updated successfully",
  "data": {
    "id": "e302008e-5b12-421c-a111-9a99fcd23b89",
    "uhid": "PAT-2026-0010",
    "phone": "+918888888888",
    "address": {
      "line1": "Flat 505, Sector 15",
      "city": "Bengaluru",
      "state": "Karnataka",
      "pincode": "560015"
    }
  }
}
```

---

### 2.5 Register Temporary Emergency Patient
Used when a patient arrives in an emergency state and full demographic details are not yet available. Creates a placeholder profile with a `TEMP-EMG-YYYYMMDD-XXX` UHID that can later be promoted to a full registration.

* **Endpoint:** `POST /patients/register-temp`
* **Required Permission:** `patients:create`
* **Request Body:** All fields are optional.
  | Field | Type | Required? | Description |
  | :--- | :--- | :--- | :--- |
  | `full_name` | String | Optional | Defaults to `"Unknown Patient"`. Max 100 chars. |
  | `phone` | String | Optional | If provided, must contain at least 10 digits. |
  | `age` | Integer | Optional | Must be between `0` and `150`. |
  | `gender` | String | Optional | `MALE`, `FEMALE`, `OTHER`, or `UNKNOWN`. Defaults to `UNKNOWN`. |

* **Example Request (minimal — unknown patient):**
```json
{}
```
* **Example Request (partial details known):**
```json
{
  "full_name": "Unknown Male Patient",
  "age": 35,
  "gender": "MALE"
}
```
* **Example Response (201 Created):**
```json
{
  "success": true,
  "message": "Temporary patient registered successfully",
  "data": {
    "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "uhid": "TEMP-EMG-20260625-001",
    "patient_number": "TEMP-EMG-20260625-001",
    "full_name": "Unknown Male Patient",
    "phone": null,
    "age": 35,
    "gender": "MALE",
    "is_temporary": true,
    "is_active": true,
    "created_at": "2026-06-25T07:45:00Z",
    "updated_at": "2026-06-25T07:45:00Z"
  }
}
```

> **Tip:** Use `GET /patients/search?q=TEMP-EMG` to find all unresolved temporary profiles.

---

### 2.6 Promote Temporary Patient to Full Registration
Once the patient's identity is confirmed, this endpoint upgrades the temporary profile to a permanent record. The `TEMP-EMG-...` UHID is replaced with a permanent `PAT-YYYY-XXXX` UHID and `is_temporary` is set to `false`. All prior emergency visits and clinical records remain linked via the same patient UUID.

* **Endpoint:** `POST /patients/{patient_id}/promote`
* **Required Permission:** `patients:edit`
* **Path Parameters:**
  | Parameter | Type | Description |
  | :--- | :--- | :--- |
  | `patient_id` | UUID | The internal UUID of the temporary patient. |

* **Request Body:**
  | Field | Type | Required? | Description |
  | :--- | :--- | :--- | :--- |
  | `full_name` | String | **Mandatory** | Min 2, Max 100 chars. |
  | `phone` | String | **Mandatory** | Min 10, Max 50 chars. |
  | `age` | Integer | **Mandatory** | 0–150. |
  | `dob` | Date | Optional | `YYYY-MM-DD`. |
  | `gender` | String | Optional | `MALE`, `FEMALE`, `OTHER`, `UNKNOWN`. |
  | `blood_group` | String | Optional | Standard blood group codes. |
  | `address` | Object | Optional | `{ line1, city, state, pincode }`. |
  | `abha_number` | String | Optional | ABHA ID. Max 100 chars. |
  | `abha_address` | String | Optional | PHR address. Max 255 chars. |
  | `state_name` | String | Optional | State of residence. |
  | `district_name` | String | Optional | District of residence. |

* **Error Cases:**
  - `400` — Patient is already a full registration (not temporary).
  - `404` — Patient not found.

* **Example Request:**
```json
{
  "full_name": "Rajesh Kumar",
  "phone": "+919876543210",
  "age": 35,
  "gender": "MALE",
  "blood_group": "B+",
  "address": {
    "line1": "12 Gandhi Nagar",
    "city": "Delhi",
    "state": "Delhi",
    "pincode": "110001"
  }
}
```
* **Example Response (200 OK):**
```json
{
  "success": true,
  "message": "Patient promoted to full registration successfully",
  "data": {
    "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "uhid": "PAT-2026-0042",
    "patient_number": "PAT-2026-0042",
    "full_name": "Rajesh Kumar",
    "phone": "+919876543210",
    "age": 35,
    "gender": "MALE",
    "blood_group": "B+",
    "is_temporary": false,
    "is_active": true,
    "created_at": "2026-06-25T07:45:00Z",
    "updated_at": "2026-06-25T08:10:00Z"
  }
}
```

---

## 3. Patient Vitals

### 3.1 Record Vitals
Submit physiological parameters for a patient.

* **Endpoint:** `POST /patients/{patient_id}/vitals`
* **Required Permission:** `patients:edit`
* **Request Body:**
  | Field | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `recorded_at` | DateTime | **Mandatory** | Time of measurement | ISO 8601 |
  | `bp_systolic` | Integer | Optional | Systolic Blood Pressure | > 50, < 250 |
  | `bp_diastolic` | Integer | Optional | Diastolic Blood Pressure | > 30, < 150 |
  | `pulse` | Integer | Optional | Heart/Pulse Rate | > 30, < 250 |
  | `weight` | Float | Optional | Weight in Kilograms | > 0.0 |
  | `temperature` | Float | Optional | Temperature in °C | > 30.0, < 45.0 |
  | `spo2` | Integer | Optional | Oxygen Saturation % | ge 0, le 100 |
  | `respiratory_rate` | Integer | Optional | Breaths per minute | > 5, < 50 |
  | `blood_sugar` | Integer | Optional | Blood glucose level (mg/dL) | ge 10, le 999 |
  | `pain_score` | Integer | Optional | Visual Pain Scale (0-10) | ge 0, le 10 |
  | `notes` | String | Optional | General observations | Max 1000 characters |

* **Example Request:**
```json
{
  "recorded_at": "2026-06-23T17:30:00Z",
  "bp_systolic": 120,
  "bp_diastolic": 80,
  "pulse": 72,
  "temperature": 37.0,
  "spo2": 98,
  "respiratory_rate": 18,
  "weight": 70.5
}
```
* **Example Response (200 OK):**
```json
{
  "success": true,
  "message": "Vitals signs recorded successfully",
  "data": {
    "id": "vit-uuid-8789-fefa",
    "patient_id": "e302008e-5b12-421c-a111-9a99fcd23b89",
    "bp_systolic": 120,
    "bp_diastolic": 80,
    "pulse": 72,
    "temperature": 37.0,
    "spo2": 98,
    "respiratory_rate": 18,
    "weight": 70.5,
    "notes": null,
    "recorded_by": "doc-uuid-1111",
    "recorded_at": "2026-06-23T17:30:00Z"
  }
}
```

---

### 3.2 Fetch Vital Logs
* **Endpoint:** `GET /patients/{patient_id}/vitals`
* **Required Permission:** `patients:view`
* **Example Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": "vit-uuid-8789-fefa",
      "patient_id": "e302008e-5b12-421c-a111-9a99fcd23b89",
      "bp_systolic": 120,
      "bp_diastolic": 80,
      "pulse": 72,
      "temperature": 37.0,
      "spo2": 98,
      "recorded_by": "doc-uuid-1111",
      "recorded_at": "2026-06-23T17:30:00Z"
    }
  ]
}
```

---

## 4. Patient Insurance

### 4.1 Link Insurance Policy
* **Endpoint:** `POST /patients/{patient_id}/insurance`
* **Required Permission:** `patients:edit`
* **Request Body:**
  | Field | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `provider_name` | String | **Mandatory** | Insurance company | Min 2, max 255 chars |
  | `policy_number` | String | **Mandatory** | Policy identification number | Min 2, max 100 chars |
  | `scheme_type` | String | Optional | Scheme class (`ROHINI`, `TPA`, `CGHS`, `ESI`, `OTHER`) | Defaults to `OTHER` |
  | `valid_from` | Date | Optional | Start Date | `YYYY-MM-DD` |
  | `valid_till` | Date | Optional | Expiration Date | `YYYY-MM-DD` |
  | `notes` | String | Optional | Policy limitations or coverage details | Max 1000 chars |

* **Example Request:**
```json
{
  "provider_name": "Star Health Insurance",
  "policy_number": "POL-998877",
  "scheme_type": "TPA",
  "valid_from": "2026-01-01",
  "valid_till": "2026-12-31"
}
```
* **Example Response (201 Created):**
```json
{
  "success": true,
  "message": "Insurance details saved successfully",
  "data": {
    "id": "ins-uuid-4444",
    "patient_id": "e302008e-5b12-421c-a111-9a99fcd23b89",
    "provider_name": "Star Health Insurance",
    "policy_number": "POL-998877",
    "scheme_type": "TPA",
    "is_active": true,
    "valid_from": "2026-01-01",
    "valid_till": "2026-12-31"
  }
}
```

---

### 4.2 Fetch Insurance Policies
* **Endpoint:** `GET /patients/{patient_id}/insurance`
* **Required Permission:** `patients:view`
* **Example Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": "ins-uuid-4444",
      "patient_id": "e302008e-5b12-421c-a111-9a99fcd23b89",
      "provider_name": "Star Health Insurance",
      "policy_number": "POL-998877",
      "scheme_type": "TPA",
      "is_active": true
    }
  ]
}
```

---

### 4.3 Update Insurance Policy
* **Endpoint:** `PATCH /patients/{patient_id}/insurance/{insurance_id}`
* **Required Permission:** `patients:edit`
* **Request Body:**
```json
{
  "is_active": false,
  "notes": "Policy expired due to non-payment of premium"
}
```
* **Example Response (200 OK):**
```json
{
  "success": true,
  "message": "Insurance details updated successfully",
  "data": {
    "id": "ins-uuid-4444",
    "is_active": false,
    "notes": "Policy expired due to non-payment of premium"
  }
}
```

---

## 5. Attendants & Medical History

### 5.1 Attendant CRUD

#### Link / Update Attendant Details
* **Endpoint:** `POST /patients/{patient_id}/attendant`
* **Required Permission:** `patients:edit`
* **Request Body:**
  | Field | Type | Required? | Description |
  | :--- | :--- | :--- | :--- |
  | `full_name` | String | **Mandatory** | Attendant name (min 2, max 255 chars) |
  | `relationship` | String | **Mandatory** | e.g. Spouse, Parent, Child |
  | `phone` | String | **Mandatory** | Contact phone (10-20 chars) |
  | `alternate_phone`| String | Optional | Alternate contact |

* **Example Request:**
```json
{
  "full_name": "John Doe",
  "relationship": "Spouse",
  "phone": "+918888888887"
}
```
* **Example Response (200 OK):**
```json
{
  "success": true,
  "message": "Attendant details updated successfully",
  "data": {
    "id": "att-uuid-1111",
    "patient_id": "e302008e-5b12-421c-a111-9a99fcd23b89",
    "full_name": "John Doe",
    "relationship": "Spouse",
    "phone": "+918888888887"
  }
}
```

#### Fetch Attendants
* **Endpoint:** `GET /patients/{patient_id}/attendant`
* **Required Permission:** `patients:view`

---

### 5.2 Medical History CRUD

#### Add Medical History Item
* **Endpoint:** `POST /patients/{patient_id}/history`
* **Required Permission:** `patients:edit`
* **Request Body:**
  | Field | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `history_type` | String | **Mandatory** | Category of medical history | `PAST_MEDICAL`, `FAMILY`, `SURGICAL`, `ALLERGY`, `SOCIAL` |
  | `condition` | String | **Mandatory** | Diagnosis or allergen details | Min 2, max 500 chars |
  | `details` | Object | Optional | Structural details about history | JSON |
  | `relation` | String | Optional | Affected relative (e.g. Mother) | For `FAMILY` history type |
  | `is_active` | Boolean | Optional | Active toggle | Defaults to `True` |
  | `is_chronic` | Boolean | Optional | Is the issue chronic? | Defaults to `False` |
  | `diagnosed_date` | Date | Optional | Diagnostic date | `YYYY-MM-DD` |
  | `severity` | String | Optional | severity class | `MILD`, `MODERATE`, `SEVERE`, `CRITICAL` |

* **Example Request:**
```json
{
  "history_type": "ALLERGY",
  "condition": "Penicillin",
  "severity": "SEVERE",
  "is_active": true,
  "details": {
    "reaction": "Anaphylaxis"
  }
}
```
* **Example Response (201 Created):**
```json
{
  "success": true,
  "message": "Medical history details logged successfully",
  "data": {
    "id": "hist-uuid-777",
    "patient_id": "e302008e-5b12-421c-a111-9a99fcd23b89",
    "history_type": "ALLERGY",
    "condition": "Penicillin",
    "severity": "SEVERE",
    "is_active": true,
    "details": {
      "reaction": "Anaphylaxis"
    }
  }
}
```

#### Fetch Medical History
* **Endpoint:** `GET /patients/{patient_id}/history`
* **Required Permission:** `patients:view`
* **Query Parameters:**
  * `history_type` (optional): Filter history items by type (`ALLERGY`, `PAST_MEDICAL`, etc.)

#### Update Medical History Item
* **Endpoint:** `PATCH /patients/{patient_id}/history/{history_id}`
* **Required Permission:** `patients:edit`
* **Request Body:**
```json
{
  "is_active": false
}
```
* **Example Response (200 OK):**
```json
{
  "success": true,
  "message": "Medical history record updated successfully",
  "data": {
    "id": "hist-uuid-777",
    "is_active": false
  }
}
```

---

## 6. Patient Diagnostic, Prescription & OPD History

### 6.1 Patient OPD History
Retrieve all past OPD check-ins and appointments.

* **Endpoint:** `GET /patients/{patient_id}/opd/history`
* **Required Permission:** `opd:view`
* **Query Parameters:**
  * `page` (optional): defaults to `1`
  * `page_size` (optional): defaults to `20`
* **Example Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "opd_visit_id": "visit-uuid-1234",
        "appointment_id": "appt-uuid-5678",
        "visit_date": "2026-06-15",
        "doctor_name": "Dr. Priya Sharma",
        "department_name": "Cardiology",
        "visit_status": "COMPLETED"
      }
    ],
    "total": 1,
    "page": 1,
    "page_size": 20
  }
}
```

---

### 6.2 Patient Prescription History
Retrieve prescription orders assigned to this patient.

* **Endpoint:** `GET /patients/{patient_id}/pharmacy`
* **Required Permission:** `patients:view`
* **Query Parameters:**
  * `context_type` (optional): `OPD_VISIT` or `IPD_ADMISSION`
  * `context_id` (optional): corresponding visit or admission UUID
  * `source_module` (optional): `OPD` or `IPD`
  * `status` (optional): Filter status (`DRAFT`, `SIGNED`, `CANCELLED`)

* **Example Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": "rx-uuid-8878",
      "prescription_id": "RX-2026-0042",
      "status": "SIGNED",
      "created_at": "2026-06-15T11:00:00Z",
      "items": [
        {
          "medicine_name": "Amoxicillin 500mg",
          "dosage": "1 capsule",
          "frequency": "Three times daily"
        }
      ]
    }
  ]
}
```

---

### 6.3 Patient Diagnostic Orders History
Retrieve laboratory or imaging orders requested for this patient.

* **Endpoint:** `GET /patients/{patient_id}/diagnostic-orders`
* **Required Permission:** `patients:view`
* **Query Parameters:**
  * `context_type` (optional): `OPD_VISIT` or `IPD_ADMISSION`
  * `context_id` (optional): corresponding UUID
  * `source_module` (optional): `OPD` or `IPD`
  * `status` (optional): Filter status (`ORDERED`, `COLLECTED`, `COMPLETED`, `CANCELLED`)

* **Example Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": "order-uuid-999",
      "order_type": "LAB",
      "status": "COMPLETED",
      "created_at": "2026-06-15T11:30:00Z",
      "lab_items": [
        {
          "test_name": "Complete Blood Count (CBC)",
          "status": "VERIFIED"
        }
      ]
    }
  ]
}
```
