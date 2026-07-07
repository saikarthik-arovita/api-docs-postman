# HMS API Reference for Unit Testing — RECEPTIONIST Module

This document outlines the API endpoints, request bodies, and response structures for all fields relevant to the **Receptionist** role across the HMS modules. This serves as the blueprint for writing unit tests.

---

## 1. Authentication Module (Receptionist Actions)

### 1.1 Login (`POST /auth/login`)
* **Description:** Receptionist log in to get access token.
* **Request Body:**
```json
{
  "email": "receptionist@arovita.com",
  "password": "SecurePassword123"
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "access_token": "eyJhbGciOi...",
    "refresh_token": "eyJhbGciOi...",
    "user": {
      "id": "reception-uuid-1111",
      "full_name": "Ramesh Kumar",
      "email": "receptionist@arovita.com",
      "role": "RECEPTIONIST"
    }
  }
}
```

### 1.2 Switch Tenant / Branch (`POST /auth/switch-branch`)
* **Description:** Switch between configured branches.
* **Request Body:**
```json
{
  "branch_id": "b2c3d4e5-f6a7-8b9c-0d1e-2f3a4b5c6d7e"
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "access_token": "new-jwt-token-with-new-claims",
    "branch_id": "b2c3d4e5-f6a7-8b9c-0d1e-2f3a4b5c6d7e"
  }
}
```

---

## 2. Patients Module (Receptionist Actions)

### 2.1 Register New Patient (`POST /patients/register`)
* **Request Body:**
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
  },
  "abha_number": "12-3456-7890-1234"
}
```
* **Success Response (201 Created):**
```json
{
  "success": true,
  "message": "Patient registered successfully",
  "data": {
    "id": "e302008e-5b12-421c-a111-9a99fcd23b89",
    "uhid": "PAT-2026-0010",
    "full_name": "Jane Doe",
    "phone": "+919999999999",
    "age": 29,
    "gender": "FEMALE",
    "is_temporary": false,
    "is_active": true,
    "created_at": "2026-06-23T12:00:00Z"
  }
}
```

### 2.2 Search Patients (`GET /patients/search`)
* **Query Parameters:** `q=Jane&page=1&page_size=20`
* **Success Response (200 OK):**
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

## 3. Doctors Module (Receptionist Actions)

### 3.1 Get Doctor Slot Availability (`GET /opd/doctors/{doctor_id}/slots`)
* **Query Parameters:** `date=2026-06-25`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "doctor_id": "902d2bc4-f5ee-45df-98bd-674cd7bb0eef",
    "date": "2026-06-25",
    "slots": [
      { "time": "10:00", "is_available": true },
      { "time": "10:15", "is_available": false }
    ]
  }
}
```

---

## 4. Nurse Module (Receptionist Actions)

Note: Receptionist role is not authorized to manage nurse duty rotas directly. However, they can read nurse shift assignments for patient stay lookups:

### 4.1 Get Patient Nursing Assignments (`GET /ipd/admissions/{admission_id}/nursing`)
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "admission_id": "admit-uuid-1111",
    "current_assignments": [
      {
        "id": "assignment-uuid-9999",
        "nurse_id": "nurse-uuid-8888",
        "nurse_name": "Nurse Priya Shetty",
        "shift": "MORNING",
        "is_active": true
      }
    ]
  }
}
```

---

## 5. Clinical Module (Receptionist Actions)

Note: Due to EMR data privacy regulations, Receptionist users do not have access to view or edit clinical files. They check-in the patients for EMR.

---

## 6. OPD Module (Receptionist Actions)

### 6.1 Book Appointment (`POST /opd/appointments`)
* **Request Body:**
```json
{
  "patient_id": "e302008e-5b12-421c-a111-9a99fcd23b89",
  "doctor_id": "902d2bc4-f5ee-45df-98bd-674cd7bb0eef",
  "appointment_date": "2026-06-25",
  "appointment_time": "10:00:00",
  "appointment_type": "CONSULTATION"
}
```
* **Success Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "id": "appt-uuid-1111",
    "patient_id": "e302008e-5b12-421c-a111-9a99fcd23b89",
    "doctor_id": "902d2bc4-f5ee-45df-98bd-674cd7bb0eef",
    "appointment_date": "2026-06-25",
    "appointment_time": "10:00:00",
    "status": "SCHEDULED",
    "token_number": 5
  }
}
```

### 6.2 Confirm & Check-In (`POST /opd/appointments/{id}/confirm`)
* **Request Body:** None
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "appt-uuid-1111",
    "status": "CONFIRMED",
    "checked_in_at": "2026-06-25T09:45:00Z"
  }
}
```

### 6.3 Register Walk-In Visit (`POST /opd/walkin`)
* **Request Body:**
```json
{
  "patient_id": "e302008e-5b12-421c-a111-9a99fcd23b89",
  "doctor_id": "902d2bc4-f5ee-45df-98bd-674cd7bb0eef",
  "department_id": "8b7f50c4-fa3a-4aef-ab27-f42b7fb0e54f"
}
```
* **Success Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "opd_visit_id": "visit-uuid-2222",
    "patient_id": "e302008e-5b12-421c-a111-9a99fcd23b89",
    "token_number": 8,
    "status": "CHECKED_IN"
  }
}
```

---

## 7. IPD Module (Receptionist Actions)

### 7.1 Wards & Bed Availability Summary (`GET /ipd/beds/availability`)
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "total_beds": 120,
    "available": 45,
    "occupied": 62,
    "reserved": 8,
    "maintenance": 5
  }
}
```

### 7.2 Create Admission Request (`POST /ipd/admission-requests`)
* **Request Body:**
```json
{
  "patient_id": "e302008e-5b12-421c-a111-9a99fcd23b89",
  "doctor_id": "902d2bc4-f5ee-45df-98bd-674cd7bb0eef",
  "recommended_ward_type": "GENERAL",
  "diagnosis_notes": "Surgical planning for cholecystectomy."
}
```
* **Success Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "id": "admit-req-uuid-3333",
    "patient_id": "e302008e-5b12-421c-a111-9a99fcd23b89",
    "status": "PENDING_APPROVAL"
  }
}
```

---

## 8. Billing Module (Receptionist Actions)

### 8.1 Collect Payment (`POST /billing/collect`)
* **Request Body:**
```json
{
  "patient_id": "e302008e-5b12-421c-a111-9a99fcd23b89",
  "amount": 250.00,
  "payment_mode": "UPI",
  "reference_number": "TXN9876543210",
  "billing_category": "OPD_REGISTRATION"
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "receipt_id": "rcpt-uuid-7777",
    "patient_id": "e302008e-5b12-421c-a111-9a99fcd23b89",
    "amount": 250.00,
    "payment_mode": "UPI",
    "status": "PAID",
    "received_at": "2026-06-25T10:15:00Z"
  }
}
```

### 8.2 Generate Invoice (`POST /billing/invoices`)
* **Request Body:**
```json
{
  "patient_id": "e302008e-5b12-421c-a111-9a99fcd23b89",
  "items": [
    { "item_name": "Consultation Fee", "quantity": 1, "unit_price": 200.00 },
    { "item_name": "Registration Fee", "quantity": 1, "unit_price": 50.00 }
  ]
}
```
* **Success Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "invoice_id": "inv-uuid-8888",
    "invoice_number": "INV-2026-0044",
    "total_amount": 250.00,
    "status": "UNPAID"
  }
}
```

---

## 9. Pharmacy Module (Receptionist Actions)

Note: Receptionists do not prepare, prescribe, or dispense drugs, but are able to view billing entries related to pharmacy purchases.

### 9.1 Get Invoice Detail (Including Pharmacy Line Items) (`GET /billing/invoices/{invoice_id}`)
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "invoice_id": "inv-uuid-8888",
    "invoice_number": "INV-2026-0044",
    "total_amount": 120.00,
    "status": "PAID",
    "items": [
      { "item_name": "Paracetamol 650mg", "quantity": 10, "unit_price": 2.50 },
      { "item_name": "Amoxicillin 500mg", "quantity": 10, "unit_price": 9.50 }
    ]
  }
}
```
