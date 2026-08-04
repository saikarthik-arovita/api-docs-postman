# Postman Testing Guide: OPD Appointment Rescheduling — End-to-End Flow

This guide walks you through a **real, executable Postman flow** — from patient registration to appointment creation, confirmation, and finally rescheduling — using actual API gateway URLs and request/response bodies.

---

## 1. Environment Setup

### Base URL (API Gateway)

```
https://4t1eo222cl.execute-api.ap-south-1.amazonaws.com/dev
```

### Authentication

Login as **Receptionist** to get a JWT token:

**URL**: `POST https://4t1eo222cl.execute-api.ap-south-1.amazonaws.com/dev/auth/login`

**Request Body**:
```json
{
  "username": "sunitha.nair.b1@hospital.com",
  "password": "Temp@123"
}
```

**Response**: Copy the `access_token` from the response and use it as `Bearer {{access_token}}` in all subsequent requests.

### Headers (All Requests)

```
Authorization: Bearer <paste_access_token_here>
Content-Type: application/json
```

---

## 2. Complete Flow: Registration → Booking → Confirmation → Reschedule

---

### STEP 1: Register a New Patient

**Method**: `POST`
**URL**: `https://4t1eo222cl.execute-api.ap-south-1.amazonaws.com/dev/patients/register`

#### Request Body
```json
{
  "full_name": "Rahul Verma",
  "phone": "+919876543210",
  "age": 34,
  "gender": "MALE",
  "blood_group": "B+",
  "address": {
    "line1": "Flat 201, Emerald Heights",
    "city": "Bengaluru",
    "state": "Karnataka",
    "pincode": "560034"
  }
}
```

#### Expected Response (200 / 201)
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "<<patient_id_uuid>>",
    "full_name": "Rahul Verma",
    "phone": "+919876543210",
    "uhid": "UHID-2026-00XXX",
    "age": 34,
    "gender": "MALE",
    "blood_group": "B+",
    "address": {
      "line1": "Flat 201, Emerald Heights",
      "city": "Bengaluru",
      "state": "Karnataka",
      "pincode": "560034"
    },
    "created_at": "2026-08-01T03:30:00Z"
  }
}
```

> ⚠️ **Copy `data.id`** from the response — this is your `patient_id` for the next step.

---

### STEP 2: Fetch Available Doctors (Optional — to get real doctor_id)

**Method**: `GET`
**URL**: `https://4t1eo222cl.execute-api.ap-south-1.amazonaws.com/dev/opd/doctors`

#### Expected Response (200 OK)
```json
{
  "success": true,
  "data": [
    {
      "doctor_id": "902d2bc4-f5ee-45df-98bd-674cd7bb0eef",
      "doctor_name": "Dr. Priya Sharma",
      "department_id": "8b7f50c4-fa3a-4aef-ab27-f42b7fb0e54f",
      "department_name": "General Medicine"
    }
  ]
}
```

> Use `doctor_id` = `902d2bc4-f5ee-45df-98bd-674cd7bb0eef` and `department_id` = `8b7f50c4-fa3a-4aef-ab27-f42b7fb0e54f` from the response.

---

### STEP 3: Create Draft Appointment

**Method**: `POST`
**URL**: `https://4t1eo222cl.execute-api.ap-south-1.amazonaws.com/dev/opd/appointments`

#### Request Body
```json
{
  "patient_id": "<<patient_id from Step 1>>",
  "doctor_id": "902d2bc4-f5ee-45df-98bd-674cd7bb0eef",
  "department_id": "8b7f50c4-fa3a-4aef-ab27-f42b7fb0e54f",
  "appointment_time": "2026-08-05T10:30:00+05:30",
  "appointment_type": "NEW",
  "is_emergency": false,
  "priority": 0,
  "notes": "Initial OPD consultation"
}
```

#### Expected Response (200 OK)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "<<appointment_id_uuid>>",
    "tenant_id": "<<tenant_uuid>>",
    "patient_id": "<<patient_id>>",
    "patient_name": "Rahul Verma",
    "doctor_id": "902d2bc4-f5ee-45df-98bd-674cd7bb0eef",
    "doctor_name": "Dr. Priya Sharma",
    "department_id": "8b7f50c4-fa3a-4aef-ab27-f42b7fb0e54f",
    "department_name": "General Medicine",
    "appointment_time": "2026-08-05T05:00:00Z",
    "status": "PENDING_PAYMENT",
    "appointment_type": "NEW",
    "token_number": null,
    "is_emergency": false,
    "priority": 0,
    "payment_status": "PENDING",
    "notes": "Initial OPD consultation",
    "created_at": "2026-08-01T03:35:00Z"
  }
}
```

> ⚠️ **Copy `data.id`** — this is your `appointment_id` for all subsequent steps.

---

### STEP 4: Confirm Appointment (Payment)

**Method**: `POST`
**URL**: `https://4t1eo222cl.execute-api.ap-south-1.amazonaws.com/dev/opd/appointments/<<appointment_id>>/confirm`

#### Request Body
```json
{
  "payment_status": "PAID",
  "payment_mode": "CASH",
  "amount_paid": 500.00,
  "payment_ref": "CASH-REC-001"
}
```

#### Expected Response (200 OK)
```json
{
  "success": true,
  "code": 200,
  "message": "Appointment confirmed",
  "data": {
    "id": "<<appointment_id>>",
    "tenant_id": "<<tenant_uuid>>",
    "patient_id": "<<patient_id>>",
    "patient_name": "Rahul Verma",
    "doctor_id": "902d2bc4-f5ee-45df-98bd-674cd7bb0eef",
    "doctor_name": "Dr. Priya Sharma",
    "department_id": "8b7f50c4-fa3a-4aef-ab27-f42b7fb0e54f",
    "department_name": "General Medicine",
    "appointment_time": "2026-08-05T05:00:00Z",
    "status": "SCHEDULED",
    "token_number": 18,
    "payment_status": "PAID",
    "payment_mode": "CASH",
    "amount_paid": 500.00,
    "opd_visit_id": "<<opd_visit_uuid>>",
    "created_at": "2026-08-01T03:35:00Z"
  }
}
```

> At this point, the appointment is **SCHEDULED** with a token assigned. It is now eligible for rescheduling.

---

### STEP 5: Reschedule Appointment ✅ (Main Endpoint)

**Method**: `PATCH`
**URL**: `https://4t1eo222cl.execute-api.ap-south-1.amazonaws.com/dev/opd/appointments/<<appointment_id>>/reschedule`

---

#### Scenario A: Full Reschedule (Date + Slot + Same Doctor + Reason)

##### Request Body
```json
{
  "appointment_date": "2026-08-06",
  "slot_time": "11:00 AM",
  "doctor_id": "902d2bc4-f5ee-45df-98bd-674cd7bb0eef",
  "department_id": "8b7f50c4-fa3a-4aef-ab27-f42b7fb0e54f",
  "reason": "Patient requested morning slot"
}
```

##### Expected Response (200 OK)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "appointment_id": "<<appointment_id>>",
    "appointment_number": "OPD-2026-00018",
    "patient_name": "Rahul Verma",
    "old_schedule": {
      "appointment_date": "2026-08-05",
      "slot_time": "10:30 AM",
      "doctor": "Dr. Priya Sharma"
    },
    "new_schedule": {
      "appointment_date": "2026-08-06",
      "slot_time": "11:00 AM",
      "doctor": "Dr. Priya Sharma"
    },
    "queue_position": 4,
    "token_number": "T-18",
    "status": "RESCHEDULED",
    "updated_at": "2026-08-01T03:40:00Z"
  }
}
```

---

#### Scenario B: Minimal Reschedule (Only Date & Slot — Doctor/Dept Unchanged)

##### Request Body
```json
{
  "appointment_date": "2026-08-07",
  "slot_time": "03:30 PM"
}
```

##### Expected Response (200 OK)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "appointment_id": "<<appointment_id>>",
    "appointment_number": "OPD-2026-00018",
    "patient_name": "Rahul Verma",
    "old_schedule": {
      "appointment_date": "2026-08-06",
      "slot_time": "11:00 AM",
      "doctor": "Dr. Priya Sharma"
    },
    "new_schedule": {
      "appointment_date": "2026-08-07",
      "slot_time": "03:30 PM",
      "doctor": "Dr. Priya Sharma"
    },
    "queue_position": 2,
    "token_number": "T-18",
    "status": "RESCHEDULED",
    "updated_at": "2026-08-01T03:45:00Z"
  }
}
```

---

### STEP 6: Verify Rescheduled Appointment (GET by ID)

**Method**: `GET`
**URL**: `https://4t1eo222cl.execute-api.ap-south-1.amazonaws.com/dev/opd/appointments/<<appointment_id>>`

#### Expected Response (200 OK)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "<<appointment_id>>",
    "patient_name": "Rahul Verma",
    "doctor_name": "Dr. Priya Sharma",
    "department_name": "General Medicine",
    "appointment_time": "2026-08-07T10:00:00Z",
    "status": "SCHEDULED",
    "token_number": 18,
    "payment_status": "PAID",
    "is_emergency": false
  }
}
```

---

## 3. Error Scenarios to Test

### ❌ Past Date (400 Bad Request)

```
PATCH /opd/appointments/<<appointment_id>>/reschedule
```

```json
{
  "appointment_date": "2025-01-01",
  "slot_time": "10:00 AM"
}
```

**Response**:
```json
{
  "success": false,
  "code": 400,
  "error": "Validation error: Appointment date cannot be in the past"
}
```

---

### ❌ Non-existent Doctor (404 Not Found)

```json
{
  "appointment_date": "2026-08-10",
  "slot_time": "10:00 AM",
  "doctor_id": "00000000-0000-0000-0000-000000000000"
}
```

**Response**:
```json
{
  "success": false,
  "code": 404,
  "error": "Doctor '00000000-0000-0000-0000-000000000000' not found or inactive"
}
```

---

## 4. Quick Reference Table

| Step | Method | Endpoint | Purpose |
| :--- | :--- | :--- | :--- |
| Login | `POST` | `/auth/login` | Get JWT access token |
| Register Patient | `POST` | `/patients/register` | Create new patient → get `patient_id` |
| List Doctors | `GET` | `/opd/doctors` | Fetch doctor UUIDs |
| Create Appointment | `POST` | `/opd/appointments` | Draft appointment → get `appointment_id` |
| Confirm Payment | `POST` | `/opd/appointments/{id}/confirm` | Pay & assign token |
| **Reschedule** | `PATCH` | `/opd/appointments/{id}/reschedule` | Change date/slot/doctor |
| Verify | `GET` | `/opd/appointments/{id}` | Confirm updated schedule |

---

## 5. Field Reference: Reschedule Request Body

| Field | Type | Required | Notes |
| :--- | :--- | :--- | :--- |
| `appointment_date` | `YYYY-MM-DD` | **Yes** | Must be today or future |
| `slot_time` | String | **Yes** | `"11:00 AM"`, `"03:30 PM"`, `"14:00"` |
| `doctor_id` | UUID | No | Omit to keep current doctor |
| `department_id` | UUID | No | Omit to keep current department |
| `reason` | String | No | Max 500 chars |
