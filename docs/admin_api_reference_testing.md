# HMS API Reference for Unit Testing — ADMIN Module

This document outlines the API endpoints, request bodies, and response structures for all fields relevant to the **Admin** role across the HMS modules. This serves as the blueprint for writing unit tests.

---

## 1. Authentication & HRMS Module

### 1.1 Create Staff Record (`POST /auth/staff`)
* **Description:** HR or Admin provisions a new staff user.
* **Headers:** `Authorization: Bearer <token>`
* **Request Body:**
```json
{
  "full_name": "Dr. Priya Sharma",
  "email": "priya.sharma@arovita.com",
  "phone": "9123456789",
  "role": "DOCTOR",
  "department_id": "8b7f50c4-fa3a-4aef-ab27-f42b7fb0e54f",
  "specialization_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
  "registration_number": "MCI/2022/98765",
  "qualification": "MBBS, MS (Ortho)",
  "experience_years": 4,
  "joining_date": "2025-07-01",
  "gender": "FEMALE",
  "date_of_birth": "1990-05-20"
}
```
* **Success Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "user_id": "902d2bc4-f5ee-45df-98bd-674cd7bb0eef",
    "employee_id": "DOC-00456",
    "email": "priya.sharma@arovita.com",
    "role": "DOCTOR",
    "onboarding_status": "NOT_ACTIVATED",
    "temp_password": "Temp@Password2026",
    "message": "Staff record created and credentials provisioned."
  }
}
```

### 1.2 Update Staff Profile (`PATCH /auth/staff/{user_id}/profile`)
* **Description:** Updates specific profile metadata.
* **Request Body:**
```json
{
  "full_name": "Dr. Priya Sharma",
  "phone": "+919123456789",
  "department_id": "8b7f50c4-fa3a-4aef-ab27-f42b7fb0e54f",
  "specialization_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
  "qualification": "MBBS, MD",
  "experience_years": 5
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "user_id": "902d2bc4-f5ee-45df-98bd-674cd7bb0eef",
    "full_name": "Dr. Priya Sharma",
    "phone": "+919123456789",
    "department_id": "8b7f50c4-fa3a-4aef-ab27-f42b7fb0e54f",
    "specialization_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "qualification": "MBBS, MD",
    "experience_years": 5,
    "updated_at": "2026-06-23T12:05:00Z"
  }
}
```

### 1.3 Activate Staff Member (`POST /hrms/staff/{user_id}/activate`)
* **Description:** Enables login and marks the profile status as active.
* **Request Body:** None
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "user_id": "902d2bc4-f5ee-45df-98bd-674cd7bb0eef",
    "employee_id": "DOC-00456",
    "onboarding_status": "ACTIVE",
    "login_enabled": true
  }
}
```

---

## 2. Patients Module (Admin Actions)

### 2.1 View Patient Details (`GET /patients/{patient_id}`)
* **Description:** Retrieve comprehensive demographic profile and health record context.
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "e302008e-5b12-421c-a111-9a99fcd23b89",
    "uhid": "PAT-2026-0010",
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
    "is_temporary": false,
    "is_active": true,
    "created_at": "2026-06-23T12:00:00Z"
  }
}
```

---

## 3. Doctors Module (Admin Actions)

### 3.1 Adjust Doctor Consultation Fees (`PATCH /admin/staff/{user_id}/fee`)
* **Description:** Updates the OPD base fee.
* **Request Body:**
```json
{
  "consultation_fee": 600.00
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "user_id": "902d2bc4-f5ee-45df-98bd-674cd7bb0eef",
    "consultation_fee": 600.00,
    "message": "Consultation fee updated successfully"
  }
}
```

### 3.2 Doctor Performance Analytics (`GET /admin/analytics/doctor-performance`)
* **Query Params:** `from_date=2026-06-01&to_date=2026-06-13&limit=5`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "from_date": "2026-06-01",
    "to_date": "2026-06-13",
    "doctors": [
      {
        "doctor_id": "902d2bc4-f5ee-45df-98bd-674cd7bb0eef",
        "doctor_name": "Dr. Priya Sharma",
        "consultations_count": 84,
        "revenue_generated": 42000.00,
        "avg_rating": 4.8
      }
    ]
  }
}
```

---

## 4. Nurse Module (Admin Actions)

### 4.1 Assign Nurse to Admission (`POST /ipd/admissions/{admission_id}/nursing/assign`)
* **Description:** Binds a nurse to a patient stay.
* **Request Body:**
```json
{
  "nurse_id": "nurse-uuid-8888",
  "assignment_type": "PRIMARY",
  "shift": "MORNING",
  "handover_notes": "Patient is a fall risk. Maintain bed rails raised."
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Nurse assigned successfully",
  "data": {
    "admission_id": "admit-uuid-1111",
    "nurse_id": "nurse-uuid-8888",
    "shift": "MORNING",
    "is_active": true
  }
}
```

---

## 5. Clinical Module (Admin Actions)

### 5.1 Save/Modify Department Consultation Form (`PUT /clinical/consultations/{consultation_id}/forms/{form_type}`)
* **Request Body:**
```json
{
  "department_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
  "form_data": {
    "chief_complaints": "Acute toothache in lower right molar",
    "duration": "3 days",
    "pain_character": "Sharp, radiating"
  }
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Consultation form saved successfully",
  "data": {
    "consultation_id": "c8542c47-f3e5-4ec0-8f24-b9f7e58c32f2",
    "form_type": "CURRENT_SYMPTOMS",
    "version": 1,
    "form_data": {
      "chief_complaints": "Acute toothache in lower right molar",
      "duration": "3 days",
      "pain_character": "Sharp, radiating"
    },
    "updated_at": "2026-06-23T17:35:00Z"
  }
}
```

---

## 6. OPD Module (Admin Actions)

### 6.1 View OPD Dashboard Summary (`GET /opd/dashboard`)
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "date": "2026-06-25",
    "total_appointments": 42,
    "scheduled": 18,
    "in_progress": 5,
    "completed": 14,
    "no_show": 3,
    "cancelled": 2
  }
}
```

---

## 7. IPD Module (Admin Actions)

### 7.1 Create Ward (`POST /admin/wards`)
* **Request Body:**
```json
{
  "name": "ICU Block B",
  "ward_type": "ICU",
  "capacity": 12,
  "floor": "3",
  "work_area_id": "a1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d"
}
```
* **Success Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "id": "4c5d12d2-79dd-4650-867e-079ac3b85a5a",
    "tenant_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "name": "ICU Block B",
    "ward_type": "ICU",
    "floor": "3",
    "capacity": 12,
    "is_active": true,
    "created_at": "2026-07-01T09:00:00Z"
  }
}
```

### 7.2 Create Bed (`POST /admin/beds`)
* **Request Body:**
```json
{
  "ward_id": "4c5d12d2-79dd-4650-867e-079ac3b85a5a",
  "bed_number": "ICU-B-002",
  "bed_type": "ICU"
}
```
* **Success Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "id": "a1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d",
    "ward_id": "4c5d12d2-79dd-4650-867e-079ac3b85a5a",
    "bed_number": "ICU-B-002",
    "bed_type": "ICU",
    "status": "AVAILABLE",
    "is_active": true,
    "created_at": "2026-07-01T09:10:00Z"
  }
}
```

### 7.3 Update Bed Status (`PATCH /admin/beds/{bed_id}/status`)
* **Request Body:**
```json
{
  "status": "MAINTENANCE",
  "reason": "Deep cleaning scheduled after discharge."
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "84820843-9876-4321-9abc-1234567890ab",
    "bed_number": "ICU-B-001",
    "status": "MAINTENANCE",
    "is_active": true
  }
}
```

---

## 8. Billing Module (Admin Actions)

### 8.1 Hospital Settings Upsert (`POST /admin/settings/{category}`)
* **Request Body:**
```json
{
  "settings": {
    "OPD_REGISTRATION_FEE": "200",
    "MAX_DAILY_APPOINTMENTS_PER_DOC": "30"
  }
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "e8f77d33-4dfd-4b8c-8ef8-bde88de6d4f9",
        "category": "billing",
        "key": "OPD_REGISTRATION_FEE",
        "value": "200",
        "is_encrypted": false
      }
    ],
    "total": 2
  }
}
```

### 8.2 Manual Billing Clearance Override (`POST /billing/clearance/{admission_id}/clear`)
* **Request Body:**
```json
{
  "clearance_type": "CREDIT_GUARANTEE",
  "approved_by": "admin-uuid-9999",
  "notes": "Approved by MD. Corporate billing will settle post-discharge."
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Billing clearance status forced successfully",
  "data": {
    "admission_id": "admit-uuid-1111",
    "is_cleared": true,
    "outstanding_balance": 15000.00,
    "clearance_type": "CREDIT_GUARANTEE",
    "notes": "Approved by MD. Corporate billing will settle post-discharge."
  }
}
```

---

## 9. Pharmacy Module (Admin Actions)

### 9.1 Create Medical Inventory Stock (`POST /admin/inventory/medical`)
* **Request Body:**
```json
{
  "item_code": "MED-PAR-650",
  "item_name": "Paracetamol 650mg Tablets",
  "category": "TABLETS",
  "batch_number": "BATCH-2026-A",
  "expiry_date": "2028-05-31",
  "opening_balance": 5000,
  "reorder_level": 500,
  "mrp": 2.50
}
```
* **Success Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "id": "inv-uuid-123456",
    "item_code": "MED-PAR-650",
    "item_name": "Paracetamol 650mg Tablets",
    "status_level": "GOOD",
    "created_at": "2026-06-13T12:45:00Z"
  }
}
```

### 9.2 Create Purchase Order (PO) (`POST /admin/procurement/orders`)
* **Request Body:**
```json
{
  "vendor_id": "vendor-uuid-123456",
  "expected_delivery_date": "2026-06-25",
  "items": [
    { "item_code": "MED-PAR-650", "quantity": 10000, "unit_price": 2.10 }
  ]
}
```
* **Success Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "po_id": "po-uuid-123456",
    "po_number": "PO-2026-0089",
    "vendor_id": "vendor-uuid-123456",
    "status": "DRAFT",
    "total_amount": 21000.00,
    "created_at": "2026-06-13T12:50:00Z"
  }
}
```
