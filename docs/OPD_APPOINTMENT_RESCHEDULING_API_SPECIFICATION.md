# OPD Appointment Rescheduling API Specification

## 1. Overview

The **OPD Appointment Rescheduling API** allows receptionists, front desk staff, and administrators to modify the date, time slot, assigned doctor, and/or department of an existing OPD appointment. 

This PATCH endpoint preserves the original appointment history and patient assignment while releasing the previously held slot, reserving the requested slot, recalculating the queue position, and logging a complete audit entry in `opd.appointment_reschedule_history`.

---

## 2. Endpoint Metadata

- **HTTP Method**: `PATCH`
- **Path**: `/opd/appointments/{appointment_id}/reschedule`
- **Auth Required**: `Yes` (`Bearer <JWT Token>`)
- **Required Permission**: `appointments:edit`
- **Supported Roles**: `ADM-001` (Hospital Admin), `ADM-002` (Receptionist), `ADM-005` (Front Desk), `MED-001` (Doctor), `MED-002` (Nurse), `MED-003` (Head Nurse)

---

## 3. Headers & Parameters

### Request Headers

| Header | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `Authorization` | String | **Yes** | Bearer JWT access token (e.g. `Bearer eyJhbGci...`) |
| `x-tenant-id` | String (UUID) | **Yes** | Active tenant / branch UUID |
| `Content-Type` | String | **Yes** | Must be `application/json` |

### Path Parameters

| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `appointment_id` | String (UUID) | **Yes** | The unique identifier of the appointment to reschedule |

### Query Parameters

*None.* All parameters for rescheduling are supplied in the JSON request body.

---

## 4. Request Body Specification

### Field Matrix

| Field Name | Type | Required | Description / Rules |
| :--- | :--- | :--- | :--- |
| `appointment_date` | String (`YYYY-MM-DD`) | **Yes** | Target date for the rescheduled appointment. Must be today or a future date. |
| `slot_time` | String | **Yes** | Target slot time (e.g. `"11:00 AM"`, `"02:30 PM"`, or `"11:00"`). |
| `doctor_id` | String (UUID) | Optional | Target doctor UUID. If omitted, the existing doctor is retained. |
| `department_id` | String (UUID) | Optional | Target department UUID. If omitted, the existing department is retained. |
| `reason` | String | Optional | Reason for rescheduling (max 500 characters). |

---

### Request Body Examples

#### Example 1: Full Reschedule Request (Date, Slot, Doctor, Department & Reason)

```json
{
  "appointment_date": "2026-08-03",
  "slot_time": "11:00 AM",
  "doctor_id": "ba865601-e3b7-4bfc-9b0e-0706ce894dd5",
  "department_id": "902d2bc4-f5ee-45df-98bd-674cd7bb0e11",
  "reason": "Patient requested morning appointment with specialist"
}
```

#### Example 2: Minimal Reschedule Request (Only Date & Slot Time)

```json
{
  "appointment_date": "2026-08-03",
  "slot_time": "11:00 AM"
}
```

---

## 5. Response Specification

### HTTP 200 OK Response Payload

```json
{
  "success": true,
  "code": 200,
  "data": {
    "appointment_id": "ba865601-e3b7-4bfc-9b0e-0706ce894dd5",
    "appointment_number": "OPD-2026-00018",
    "patient_name": "Rahul Verma",
    "old_schedule": {
      "appointment_date": "2026-08-01",
      "slot_time": "10:30 AM",
      "doctor": "Dr. Priya Verma"
    },
    "new_schedule": {
      "appointment_date": "2026-08-03",
      "slot_time": "11:00 AM",
      "doctor": "Dr. Priya Verma"
    },
    "queue_position": 4,
    "token_number": "T-18",
    "status": "RESCHEDULED",
    "updated_at": "2026-08-03T11:05:20Z"
  }
}
```

### Response Field Descriptions

| Field | Type | Description |
| :--- | :--- | :--- |
| `success` | Boolean | `true` if operation completed successfully |
| `code` | Integer | HTTP status code (`200`) |
| `data.appointment_id` | String (UUID) | Unique ID of the appointment |
| `data.appointment_number` | String | Formatted appointment tracking number |
| `data.patient_name` | String | Full name of the patient |
| `data.old_schedule` | Object | Previous scheduling details prior to update |
| `data.old_schedule.appointment_date` | String | Previous appointment date |
| `data.old_schedule.slot_time` | String | Previous slot time |
| `data.old_schedule.doctor` | String | Previous doctor name |
| `data.new_schedule` | Object | Updated scheduling details |
| `data.new_schedule.appointment_date` | String | Updated appointment date |
| `data.new_schedule.slot_time` | String | Updated slot time |
| `data.new_schedule.doctor` | String | Updated doctor name |
| `data.queue_position` | Integer | Recalculated queue position for the doctor/date |
| `data.token_number` | String | Formatted token sequence number (e.g. `"T-18"`) |
| `data.status` | String | `"RESCHEDULED"` |
| `data.updated_at` | String (ISO-8601) | Timestamp of modification in UTC |

---

## 6. Business Rules & State Constraints

| Current Appointment Status | Reschedule Allowed? | HTTP Status Code | Behavior / Error Message |
| :--- | :---: | :---: | :--- |
| `PENDING_PAYMENT` | **Yes** | `200 OK` | Schedule updated. Token generated upon confirmation. |
| `SCHEDULED` | **Yes** | `200 OK` | Schedule updated, queue position recalculated. |
| `CONFIRMED` | **Yes** | `200 OK` | Schedule updated, slot released/booked, queue recalculated. |
| `CHECKED_IN` | **No** | `409 Conflict` | Cannot reschedule appointment with status 'CHECKED_IN' |
| `IN_CONSULTATION` | **No** | `409 Conflict` | Cannot reschedule appointment with status 'IN_CONSULTATION' |
| `COMPLETED` | **No** | `409 Conflict` | Cannot reschedule appointment with status 'COMPLETED' |
| `CANCELLED` | **No** | `409 Conflict` | Cannot reschedule appointment with status 'CANCELLED' |

---

## 7. Error Responses

### 400 Bad Request (Validation Error)

Returned when `appointment_date` is in the past or `slot_time` format is invalid.

```json
{
  "success": false,
  "code": 400,
  "error": "Validation error: Appointment date cannot be in the past"
}
```

### 401 Unauthorized

Returned when `Authorization` header is missing, invalid, or expired.

```json
{
  "success": false,
  "code": 401,
  "error": "Unauthorized — missing Authorization header"
}
```

### 404 Not Found

Returned when the specified `appointment_id`, `doctor_id`, or `department_id` does not exist.

```json
{
  "success": false,
  "code": 404,
  "error": "Appointment 'ba865601-e3b7-4bfc-9b0e-0706ce894dd5' not found"
}
```

### 409 Conflict (State Conflict)

Returned when attempting to reschedule an appointment that is already checked in, completed, or cancelled.

```json
{
  "success": false,
  "code": 409,
  "error": "Cannot reschedule appointment with status 'COMPLETED'"
}
```
