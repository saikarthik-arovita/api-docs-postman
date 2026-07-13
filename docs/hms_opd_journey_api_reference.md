# HMS Outpatient (OPD) Journey API Reference

This document provides the technical API contracts, query parameters, validation rules, and JSON payloads for the entire Patient Registration, Receptionist Dashboard, and Doctor Consultation (OPD) workflows, mapping directly to your 12 frontend layout screens.

---

## 🔑 1. Global Conventions

### Required Request Headers
Every request must include the following headers for authorization and tenant isolation:
```http
Authorization: Bearer <access_token>
Content-Type: application/json
X-Tenant-Id: <tenant-uuid>
```

### Standard Response Envelope

#### Success Response
```json
{
  "success": true,
  "code": 200,
  "message": "Operation completed successfully",
  "data": {}
}
```

#### Error Response
```json
{
  "success": false,
  "code": 400,
  "error": "Detailed business or validation error message"
}
```

---

## 📋 2. Screen-by-Screen API Documentation

### **Screen 1: Patient Profile & Details**
Fetches demographics, clinical timelines, and active files for a patient.

#### **API: Get Patient Details**
* **HTTP Method:** `GET`
* **Endpoint:** `/patients/{patient_id}`
* **Path Parameters:**
  * `patient_id` (UUID, Mandatory): Patient record identifier.
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "e2c07a01-2092-4876-8889-112233445566",
    "uhid": "PAT-2026-0089",
    "full_name": "Priyanka Nair",
    "gender": "FEMALE",
    "dob": "1994-04-18",
    "age": 32,
    "phone": "+919781186435",
    "email": "priyankanair.b1@gmail.com",
    "blood_group": "B+",
    "is_active": true
  }
}
```

#### **API: Get Patient Clinical Timeline**
* **HTTP Method:** `GET`
* **Endpoint:** `/patients/{patient_id}/timeline`
* **Path Parameters:**
  * `patient_id` (UUID, Mandatory): Patient identifier.
* **Query Parameters:**
  * `limit` (Integer, Optional): Max records to retrieve (default: 20).
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "event_type": "REGISTRATION",
      "description": "Patient profile created",
      "timestamp": "2026-07-13T04:00:00Z"
    },
    {
      "event_type": "OPD_VISIT",
      "description": "Consultation with Dr. Vikram Shah (Paediatrics)",
      "timestamp": "2026-07-13T04:30:00Z"
    }
  ]
}
```

---

### **Screen 2: Receptionist Dashboard**
Provides statistics widgets and active patient search capabilities.

#### **API: Get Receptionist Dashboard Summary**
* **HTTP Method:** `GET`
* **Endpoint:** `/opd/dashboard`
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "appointments_today": 48,
    "checked_in_count": 22,
    "waiting_in_queue": 12,
    "completed_consults": 14,
    "active_doctors_count": 8,
    "bed_occupancy_percent": 84.5
  }
}
```

#### **API: Unified Patient Search**
* **HTTP Method:** `GET`
* **Endpoint:** `/patients/search`
* **Query Parameters:**
  * `q` (String, Mandatory): Search term (UHID, Name, or Phone Number). Min length: 3 characters.
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "id": "e2c07a01-2092-4876-8889-112233445566",
      "uhid": "PAT-2026-0089",
      "full_name": "Priyanka Nair",
      "phone": "+919781186435",
      "gender": "FEMALE"
    }
  ]
}
```

---

### **Screen 3: Patient Registration Form**
Saves new demographic sheets to the patient database.

#### **API: Register Patient**
* **HTTP Method:** `POST`
* **Endpoint:** `/patients/register`
* **Request Body Parameters:**

| Parameter | Type | Required? | Description | Constraints |
| :--- | :--- | :--- | :--- | :--- |
| `full_name` | String | **Mandatory** | Patient's legal name | 2 to 100 characters |
| `phone` | String | **Mandatory** | Contact phone number | Valid format (min 10 digits) |
| `dob` | String | **Mandatory** | Date of birth | YYYY-MM-DD format |
| `age` | Integer | **Mandatory** | Age in years | 0 to 150 |
| `gender` | String | **Mandatory** | Gender identity | `MALE`, `FEMALE`, `OTHER`, `UNKNOWN` |
| `blood_group`| String | Optional | Blood group | `A+`, `A-`, `B+`, `B-`, `AB+`, `AB-`, `O+`, `O-` |
| `email` | String | Optional | Primary email address | Must be a valid email format |
| `address` | Object | Optional | Primary address block | Embedded object keys: `line1`, `city`, `state`, `pincode` |

#### **Request Example:**
```json
{
  "full_name": "Priyanka Nair",
  "phone": "+919781186435",
  "dob": "1994-04-18",
  "age": 32,
  "gender": "FEMALE",
  "blood_group": "B+",
  "email": "priyankanair.b1@gmail.com",
  "address": {
    "line1": "Flat 304, Green Heights",
    "city": "Mumbai",
    "state": "Maharashtra",
    "pincode": "400053"
  }
}
```

#### **Success Response (`201 Created`):**
```json
{
  "success": true,
  "code": 201,
  "message": "Patient registered successfully",
  "data": {
    "id": "e2c07a01-2092-4876-8889-112233445566",
    "uhid": "PAT-2026-0089",
    "full_name": "Priyanka Nair",
    "created_at": "2026-07-13T05:00:00Z"
  }
}
```

---

### **Screen 4: Appointment Scheduling Form**
Schedules appointments by dynamically querying specialists, slot rosters, and calendars.

#### **API: Get Department Specialties**
* **HTTP Method:** `GET`
* **Endpoint:** `/opd/departments`
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "department_id": "0fbec6ac-b221-413b-bd63-9bff86f4ce14",
      "department_name": "Orthopaedics",
      "is_active": true
    },
    {
      "department_id": "281e4472-a127-4ea0-864c-f83a0cf6189f",
      "department_name": "Gynaecology",
      "is_active": true
    }
  ]
}
```

#### **API: Get Doctors in Department**
* **HTTP Method:** `GET`
* **Endpoint:** `/opd/departments/{id}/doctors`
* **Path Parameters:**
  * `id` (UUID, Mandatory): Selected department ID.
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "doctor_id": "97d2a10d-1b8a-40b4-a37a-5daf4e35f630",
      "full_name": "Dr. Tanvi Shrivastava",
      "qualification": "MS - Orthopaedics",
      "consultation_fee": 600.00
    }
  ]
}
```

#### **API: Get Doctor Slots**
* **HTTP Method:** `GET`
* **Endpoint:** `/opd/doctors/{id}/slots`
* **Path Parameters:**
  * `id` (UUID, Mandatory): Doctor ID.
* **Query Parameters:**
  * `date` (String, Mandatory): Target consultation date (`YYYY-MM-DD`).
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "doctor_id": "97d2a10d-1b8a-40b4-a37a-5daf4e35f630",
    "date": "2026-07-13",
    "slots": [
      { "time": "09:00", "available": true },
      { "time": "09:30", "available": false },
      { "time": "10:00", "available": true }
    ]
  }
}
```

#### **API: Book Appointment**
* **HTTP Method:** `POST`
* **Endpoint:** `/opd/appointments`
* **Request Body Parameters:**

| Parameter | Type | Required? | Description | Constraints |
| :--- | :--- | :--- | :--- | :--- |
| `patient_id` | UUID | **Mandatory** | Patient ID | Must be registered |
| `doctor_id` | UUID | **Mandatory** | Assigned doctor ID | Must have active schedule |
| `department_id`| UUID | Optional | Department ID | |
| `appointment_time`| String | **Mandatory** | Time slot | ISO 8601 (YYYY-MM-DDTHH:MM:SSZ) |
| `appointment_type`| String | Optional | Consultation type | `NEW`, `FOLLOW_UP` (default: `NEW`) |
| `notes` | String | Optional | Reason for consult | Max 500 characters |

#### **Request Example:**
```json
{
  "patient_id": "e2c07a01-2092-4876-8889-112233445566",
  "doctor_id": "97d2a10d-1b8a-40b4-a37a-5daf4e35f630",
  "department_id": "0fbec6ac-b221-413b-bd63-9bff86f4ce14",
  "appointment_time": "2026-07-13T09:00:00Z",
  "appointment_type": "NEW",
  "notes": "Severe knee pain when walking."
}
```

#### **Success Response (`201 Created`):**
```json
{
  "success": true,
  "code": 201,
  "message": "OPD Appointment booked",
  "data": {
    "id": "a9a9b8b8-c7c7-6d6d-5e5e-4f4f3f3f2e2e",
    "token_number": "T-104",
    "status": "SCHEDULED",
    "created_at": "2026-07-13T05:05:00Z"
  }
}
```

---

### **Screen 5: Appointment List Queue**
Loads the master list of all bookings for reception management.

#### **API: Get Today's Appointments**
* **HTTP Method:** `GET`
* **Endpoint:** `/opd/appointments/today`
* **Query Parameters:**
  * `status` (String, Optional): Filter by booking status (`SCHEDULED`, `CONFIRMED`, `IN_PROGRESS`, `COMPLETED`, `CANCELLED`).
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "appointment_id": "a9a9b8b8-c7c7-6d6d-5e5e-4f4f3f3f2e2e",
      "token_number": "T-104",
      "patient_name": "Priyanka Nair",
      "uhid": "PAT-2026-0089",
      "doctor_name": "Dr. Tanvi Shrivastava",
      "appointment_time": "2026-07-13T09:00:00Z",
      "status": "SCHEDULED"
    }
  ]
}
```

---

### **Screen 6 & 12: Doctor Schedules & Availability lookup**
Displays doctor availability calendars and schedules.

#### **API: Check Doctor Availability**
* **HTTP Method:** `GET`
* **Endpoint:** `/opd/doctors/availability`
* **Query Parameters:**
  * `date` (String, Optional): Query target date (`YYYY-MM-DD`, defaults to today).
  * `specialty_id` (UUID, Optional): Filter by department category.
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "doctor_id": "97d2a10d-1b8a-40b4-a37a-5daf4e35f630",
      "doctor_name": "Dr. Tanvi Shrivastava",
      "is_available": true,
      "current_status": "ACTIVE_ONCALL",
      "next_slot": "09:00"
    }
  ]
}
```

---

### **Screen 7: Check-In/MFA / Status Changes**
Handles checking in a patient (verifying fee payment and activating the visit queue).

#### **API: Check-in Patient (Update Appointment Status)**
* **HTTP Method:** `POST`
* **Endpoint:** `/opd/appointments/{id}/confirm`
* **Path Parameters:**
  * `id` (UUID, Mandatory): Target appointment ID.
* **Request Body Parameters:**

| Parameter | Type | Required? | Description | Constraints |
| :--- | :--- | :--- | :--- | :--- |
| `payment_status` | String | **Mandatory** | Payment clearance flag | Must be `PAID` |
| `payment_mode` | String | **Mandatory** | Billing channel | `CASH`, `CARD`, `UPI`, `INSURANCE` |
| `amount_paid` | Number | **Mandatory** | Cost of consultation | Must match doctor's fee |
| `payment_ref` | String | Optional | Receipt reference | Transaction ID |

#### **Request Example:**
```json
{
  "payment_status": "PAID",
  "payment_mode": "UPI",
  "amount_paid": 600.00,
  "payment_ref": "UPI-TXN-11223344"
}
```

#### **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "message": "Appointment confirmed and visit generated",
  "data": {
    "appointment_id": "a9a9b8b8-c7c7-6d6d-5e5e-4f4f3f3f2e2e",
    "opd_visit_id": "f5f5e4e4-d3d3-2c2c-1b1b-0a0a9a9a8a8a",
    "status": "CONFIRMED"
  }
}
```

---

### **Screen 9: Doctor's Consultation Queue**
Returns a list of patients waiting to see a specific doctor.

#### **API: Get Doctor's Active Queue**
* **HTTP Method:** `GET`
* **Endpoint:** `/opd/queue/{doctor_id}`
* **Path Parameters:**
  * `doctor_id` (UUID, Mandatory): The ID of the doctor whose queue is being requested.
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "opd_visit_id": "f5f5e4e4-d3d3-2c2c-1b1b-0a0a9a9a8a8a",
      "token_number": "T-104",
      "patient_name": "Priyanka Nair",
      "uhid": "PAT-2026-0089",
      "status": "ARRIVED",
      "waiting_time_minutes": 15
    }
  ]
}
```

---

### **Screen 8: Doctor Consultation Workbench**
Enables the doctor to start the consult, record vitals, place service orders, and complete the visit.

#### **API: Start Consultation**
* **HTTP Method:** `POST`
* **Endpoint:** `/opd/visits/{id}/start`
* **Path Parameters:**
  * `id` (UUID, Mandatory): OPD Visit ID.
* **Request Example:**
```json
{
  "chief_complaint": "Severe knee pain since last night; inflammation observed."
}
```
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "message": "Consultation started",
  "data": {
    "visit_id": "f5f5e4e4-d3d3-2c2c-1b1b-0a0a9a9a8a8a",
    "status": "IN_PROGRESS",
    "started_at": "2026-07-13T09:15:00Z"
  }
}
```

#### **API: Record Vitals**
* **HTTP Method:** `POST`
* **Endpoint:** `/opd/vitals`
* **Request Body Parameters:**

| Parameter | Type | Required? | Description | Constraints |
| :--- | :--- | :--- | :--- | :--- |
| `patient_id` | UUID | **Mandatory** | Patient ID | |
| `opd_visit_id` | UUID | **Mandatory** | Active visit ID | |
| `bp_systolic` | Integer | Optional | Blood pressure (Systolic) | 50 to 250 mmHg |
| `bp_diastolic`| Integer | Optional | Blood pressure (Diastolic)| 30 to 150 mmHg |
| `pulse` | Integer | Optional | Pulse rate | 30 to 200 bpm |
| `temperature` | Number | Optional | Body temperature | 90 to 110 °F |
| `spo2` | Integer | Optional | Oxygen saturation | 50 to 100 % |

#### **Request Example:**
```json
{
  "patient_id": "e2c07a01-2092-4876-8889-112233445566",
  "opd_visit_id": "f5f5e4e4-d3d3-2c2c-1b1b-0a0a9a9a8a8a",
  "bp_systolic": 120,
  "bp_diastolic": 80,
  "pulse": 72,
  "temperature": 98.6,
  "spo2": 98
}
```
* **Success Response (`201 Created`):**
```json
{
  "success": true,
  "code": 201,
  "message": "Vitals recorded successfully"
}
```

#### **API: Order Surgery / Procedure**
* **HTTP Method:** `POST`
* **Endpoint:** `/opd/surgery`
* **Request Example:**
```json
{
  "patient_id": "e2c07a01-2092-4876-8889-112233445566",
  "opd_visit_id": "f5f5e4e4-d3d3-2c2c-1b1b-0a0a9a9a8a8a",
  "procedure_name": "Arthroscopic Meniscectomy",
  "priority": "ELECTIVE",
  "notes": "Recommend OT scheduling within two weeks."
}
```
* **Success Response (`201 Created`):**
```json
{
  "success": true,
  "code": 201,
  "data": {
    "order_id": "11223344-5566-7788-9900-aabbccddeeff",
    "status": "ORDERED"
  }
}
```

#### **API: Complete Consultation (Sign-off)**
* **HTTP Method:** `POST`
* **Endpoint:** `/opd/visits/{id}/complete`
* **Path Parameters:**
  * `id` (UUID, Mandatory): OPD Visit ID.
* **Request Body Parameters:**

| Parameter | Type | Required? | Description | Constraints |
| :--- | :--- | :--- | :--- | :--- |
| `decision` | String | **Mandatory** | Clinical conclusion decision | `MEDICAL`, `SURGERY`, `PROCEDURE`, `FOLLOW_UP`, `ADMIT`, `REFERRAL`, `OTHER` |
| `diagnosis` | String | Optional | Final EMR diagnostic notes | Max 2000 characters |
| `follow_up_date`| String | Optional | Future follow-up limit | Date (YYYY-MM-DD) in the future |
| `follow_up_notes`| String | Optional | Additional details for patient | Max 2000 characters |

#### **Request Example:**
```json
{
  "decision": "FOLLOW_UP",
  "diagnosis": "Left Knee Meniscal Tear (Grade II)",
  "follow_up_date": "2026-07-27",
  "follow_up_notes": "Avoid running; apply cold pack; continue medication."
}
```
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "message": "Consultation completed",
  "data": {
    "visit_id": "f5f5e4e4-d3d3-2c2c-1b1b-0a0a9a9a8a8a",
    "status": "COMPLETED",
    "completed_at": "2026-07-13T09:30:00Z"
  }
}
```

---

### **Screen 11: Billing / Receipt Details Table**
Retrieves pricing calculations for checkout services.

#### **API: Get Billing Receipt Summary**
* **HTTP Method:** `GET`
* **Endpoint:** `/opd/billing/summary`
* **Query Parameters:**
  * `opd_visit_id` (UUID, Mandatory): Target consult visit ID.
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "consultation_fee": 600.00,
    "registration_fee": 100.00,
    "subtotal": 700.00,
    "tax_amount": 35.00,
    "discount_amount": 0.00,
    "grand_total": 735.00
  }
}
```
