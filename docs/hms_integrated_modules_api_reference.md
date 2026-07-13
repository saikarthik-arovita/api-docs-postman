# HMS Unified Modules API Reference Guide

This document is a master API guide for the Hospital Management System (HMS) covering:
1. **Outpatient (OPD) & Reception Module** (12 screens)
2. **Inpatient (IPD), Ward, Nursing & OT Module** (8 screens)
3. **Pharmacy & Dispensing Module** (4 screens)

---

## 🔑 1. Global API Conventions

### Headers
All API calls require authentication and branch isolation headers:
```http
Authorization: Bearer <access_token>
Content-Type: application/json
X-Tenant-Id: <tenant-uuid>
```

### Standard Response Envelope
* **Success (200 OK / 201 Created):**
  ```json
  { "success": true, "code": 200, "message": "Success", "data": {} }
  ```
* **Error (400 / 401 / 403 / 404 / 409 / 500):**
  ```json
  { "success": false, "code": 400, "error": "Error details" }
  ```

---

## 🗂️ 2. OPD & Reception Module (12 Screens)

### **Screen 1 & 3: Patient Registration & Profile Details**

#### **1. Register Patient**
* **HTTP Method:** `POST`
* **Endpoint:** `/patients/register`
* **Purpose:** Saves patient demographics and returns a unique UHID.
* **Request Body:**
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
* **Success Response (`201 Created`):**
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "e2c07a01-2092-4876-8889-112233445566",
    "uhid": "PAT-2026-0089",
    "full_name": "Priyanka Nair",
    "is_active": true
  }
}
```

#### **2. Get Patient Details**
* **HTTP Method:** `GET`
* **Endpoint:** `/patients/{patient_id}`
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
    "phone": "+919781186435"
  }
}
```

#### **3. Get Patient Timeline**
* **HTTP Method:** `GET`
* **Endpoint:** `/patients/{patient_id}/timeline`
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
    }
  ]
}
```

---

### **Screen 2: Receptionist Dashboard**

#### **4. Get Summary Statistics**
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
    "bed_occupancy_percent": 84.5
  }
}
```

#### **5. Unified Patient Lookup**
* **HTTP Method:** `GET`
* **Endpoint:** `/patients/search`
* **Query Parameters:**
  * `q` (String, Required): Search query (Name, phone, or UHID).
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
      "phone": "+919781186435"
    }
  ]
}
```

---

### **Screen 4, 5, 6 & 12: Appointment Booking & Scheduling**

#### **6. Get Specialties**
* **HTTP Method:** `GET`
* **Endpoint:** `/opd/departments`
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": [
    { "department_id": "0fbec-14", "department_name": "Orthopaedics" }
  ]
}
```

#### **7. Get Doctors in Specialty**
* **HTTP Method:** `GET`
* **Endpoint:** `/opd/departments/{id}/doctors`
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": [
    { "doctor_id": "97d2a-30", "full_name": "Dr. Tanvi Shrivastava", "consultation_fee": 600.00 }
  ]
}
```

#### **8. Get Doctor Slots**
* **HTTP Method:** `GET`
* **Endpoint:** `/opd/doctors/{id}/slots`
* **Query Parameters:**
  * `date` (String, Required): Format: `YYYY-MM-DD`.
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "slots": [
      { "time": "09:00", "available": true },
      { "time": "09:30", "available": false }
    ]
  }
}
```

#### **9. Book OPD Appointment**
* **HTTP Method:** `POST`
* **Endpoint:** `/opd/appointments`
* **Request Body:**
```json
{
  "patient_id": "e2c07a01-2092-4876-8889-112233445566",
  "doctor_id": "97d2a10d-1b8a-40b4-a37a-5daf4e35f630",
  "department_id": "0fbec6ac-b221-413b-bd63-9bff86f4ce14",
  "appointment_time": "2026-07-13T09:00:00Z",
  "notes": "Severe knee pain."
}
```
* **Success Response (`201 Created`):**
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "a9a9b8b8-c7c7-6d6d-5e5e-4f4f3f3f2e2e",
    "token_number": "T-104",
    "status": "SCHEDULED"
  }
}
```

#### **10. Get Today's Appointments**
* **HTTP Method:** `GET`
* **Endpoint:** `/opd/appointments/today`
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
      "status": "SCHEDULED"
    }
  ]
}
```

---

## 7, 8, 9 & 10: Check-In & Consultation Workbench

#### **11. Confirm Check-In & Payment**
* **HTTP Method:** `POST`
* **Endpoint:** `/opd/appointments/{id}/confirm`
* **Request Body:**
```json
{
  "payment_status": "PAID",
  "payment_mode": "UPI",
  "amount_paid": 600.00,
  "payment_ref": "UPI-TXN-11223344"
}
```
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "opd_visit_id": "f5f5e4e4-d3d3-2c2c-1b1b-0a0a9a9a8a8a",
    "status": "CONFIRMED"
  }
}
```

#### **12. Get Doctor Queue**
* **HTTP Method:** `GET`
* **Endpoint:** `/opd/queue/{doctor_id}`
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
      "status": "ARRIVED"
    }
  ]
}
```

#### **13. Start Consultation**
* **HTTP Method:** `POST`
* **Endpoint:** `/opd/visits/{id}/start`
* **Request Body:**
```json
{
  "chief_complaint": "Severe knee pain since last night."
}
```
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": { "status": "IN_PROGRESS" }
}
```

#### **14. Record Vitals**
* **HTTP Method:** `POST`
* **Endpoint:** `/opd/vitals`
* **Request Body:**
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
{ "success": true, "code": 201 }
```

#### **15. Complete Consultation**
* **HTTP Method:** `POST`
* **Endpoint:** `/opd/visits/{id}/complete`
* **Request Body:**
```json
{
  "decision": "FOLLOW_UP",
  "diagnosis": "Left Knee Meniscal Tear (Grade II)",
  "follow_up_date": "2026-07-27",
  "follow_up_notes": "Avoid heavy workouts."
}
```
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": { "status": "COMPLETED" }
}
```

---

### **Screen 11: OPD Billing Receipt**

#### **16. Get Billing Receipt Summary**
* **HTTP Method:** `GET`
* **Endpoint:** `/opd/billing/summary`
* **Query Parameters:**
  * `opd_visit_id` (UUID, Required)
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
    "grand_total": 735.00
  }
}
```

---

## 🏥 3. IPD, Ward, Nursing & OT Module (8 Screens)

### **Screen 1 & 2: Nurse Ward Board & Inpatient Lists**

#### **1. Get Ward Board Summary**
* **HTTP Method:** `GET`
* **Endpoint:** `/ipd/dashboard/wards`
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "total_beds": 120,
    "occupied_beds": 98,
    "available_beds": 22
  }
}
```

#### **2. List Wards**
* **HTTP Method:** `GET`
* **Endpoint:** `/ipd/wards`
* **Query Parameters:**
  * `floor_id` (UUID, Optional)
  * `block_id` (UUID, Optional)
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "id": "ward-uuid-1",
      "name": "General Ward A",
      "block_id": "block-uuid-1",
      "capacity": 20
    }
  ]
}
```

#### **3. List Beds**
* **HTTP Method:** `GET`
* **Endpoint:** `/ipd/beds`
* **Query Parameters:**
  * `ward_id` (UUID, Required)
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": [
    { "id": "bed-uuid-1", "bed_number": "B-201", "status": "AVAILABLE" }
  ]
}
```

---

### **Screen 3: Admission Request Intake**

#### **4. Get Pending Admission Requests**
* **HTTP Method:** `GET`
* **Endpoint:** `/ipd/admission-requests`
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "id": "req-uuid-1",
      "patient_name": "Ramesh Sharma",
      "attending_doctor_id": "doc-uuid-1",
      "admission_reason": "Elective Laparoscopy"
    }
  ]
}
```

#### **5. Create Inpatient Admission**
* **HTTP Method:** `POST`
* **Endpoint:** `/ipd/admissions`
* **Request Body:**
```json
{
  "patient_id": "e2c07a01-2092-4876-8889-112233445566",
  "ward_id": "ward-uuid-1",
  "bed_id": "bed-uuid-1",
  "attending_doctor_id": "doc-uuid-1",
  "admission_reason": "Arthroscopic Meniscectomy",
  "insurance_details": null
}
```
* **Success Response (`201 Created`):**
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "ipd-case-uuid-1234",
    "status": "ADMITTED"
  }
}
```

---

### **Screen 4 & 5: Inpatient Details & Clinical Logs**

#### **6. Get Admission Details**
* **HTTP Method:** `GET`
* **Endpoint:** `/ipd/admissions/{id}`
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "ipd-case-uuid-1234",
    "patient_id": "e2c07a01-2092-4876-8889-112233445566",
    "uhid": "PAT-2026-0089",
    "bed_number": "B-201",
    "ward_name": "General Ward A"
  }
}
```

#### **7. Get Clinical EMR Summary**
* **HTTP Method:** `GET`
* **Endpoint:** `/ipd/admissions/{id}/clinical-summary`
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "active_medications": [],
    "recent_vitals": [],
    "lab_orders": []
  }
}
```

#### **8. Add Daily clinical Note**
* **HTTP Method:** `POST`
* **Endpoint:** `/ipd/admissions/{id}/notes`
* **Request Body:**
```json
{
  "note_type": "PROGRESS_NOTE",
  "content": "Patient reports minor discomfort. Knee flexion is recovering."
}
```
* **Success Response (`201 Created`):**
```json
{ "success": true, "code": 201 }
```

---

### **Screen 6: Inpatient Discharge Clearance**

#### **9. Initiate Discharge (Doctor Action)**
* **HTTP Method:** `POST`
* **Endpoint:** `/ipd/admissions/{id}/discharge/initiate`
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "message": "Discharge workflow successfully initiated."
}
```

#### **10. Save Draft Discharge Summary (Nurse Action)**
* **HTTP Method:** `PATCH`
* **Endpoint:** `/ipd/admissions/{id}/discharge-summary`
* **Request Body:**
```json
{
  "discharge_summary": "Successful arthroscopic repair of left knee meniscus.",
  "follow_up_date": "2026-07-27",
  "follow_up_instructions": "Review in orthopaedic clinic.",
  "medications_on_discharge": "Paracetamol 650mg TDS PRN, Pantocid 40mg OD",
  "diet_instructions": "General high-protein diet."
}
```
* **Success Response (`200 OK`):**
```json
{ "success": true, "code": 200, "message": "Discharge summary draft saved" }
```

#### **11. Submit Gate Clearances**
* **HTTP Method:** `POST`
* **Endpoint:** `/ipd/admissions/{id}/discharge/clearance`
* **Request Body:**
```json
{
  "clearance_gate": "PHARMACY",
  "status": "CLEARED",
  "notes": "No pending pharmacy dues."
}
```
* **Success Response (`200 OK`):**
```json
{ "success": true, "code": 200 }
```

#### **12. Finalize Case Discharge**
* **HTTP Method:** `POST`
* **Endpoint:** `/ipd/admissions/{id}/discharge/complete`
* **Request Body:** `{}`
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "message": "Patient discharged successfully. Bed released."
}
```

---

### **Screen 7: OT Surgery Scheduler & Consent**

#### **13. Schedule OT Surgery Case**
* **HTTP Method:** `POST`
* **Endpoint:** `/ipd/ot-sessions`
* **Request Body:**
```json
{
  "procedure_id": "procedure-uuid-1",
  "ot_room_id": "ot-room-uuid-1",
  "primary_surgeon_id": "doc-uuid-1",
  "scheduled_start_time": "2026-07-15T08:00:00Z"
}
```
* **Success Response (`201 Created`):**
```json
{
  "success": true,
  "code": 201,
  "data": { "ot_session_id": "ot-session-uuid-12" }
}
```

#### **14. Clear PAC (Pre-Anaesthetic Checkup)**
* **HTTP Method:** `POST`
* **Endpoint:** `/ipd/ot-sessions/{id}/pac`
* **Request Body:**
```json
{
  "clearance_status": "CLEARED",
  "asa_grade": "ASA_1",
  "notes": "Lungs clear. Good cardiac tolerance."
}
```
* **Success Response (`200 OK`):**
```json
{ "success": true, "code": 200 }
```

---

### **Screen 8: IPD/OT Admin Analytics Dashboard**

#### **15. Get Occupancy & ALOS Metrics**
* **HTTP Method:** `GET`
* **Endpoint:** `/ipd/dashboard`
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "active_inpatients_count": 98,
    "average_length_of_stay": 4.2,
    "daily_admissions_count": 12
  }
}
```

---

## 💊 4. Pharmacy & Dispensing Module (4 Screens)

> [!NOTE]
> For the comprehensive, detailed API specification of the Pharmacy module (covering dashboard metrics, patient/customer registrations, prescription versioning, dispensing carts, inventory batch FEFO allocations, returns, and procurement), please refer to the detailed [pharmacy_mgr_api_specs.md](file:///c:/Users/saika/OneDrive/Desktop/Arovita/ops-hms-ljb/docs/pharmacy_mgr_api_specs.md).

### **Screen 1: Dashboard Overview**

#### **1. Get Summary Stats**
* **HTTP Method:** `GET`
* **Endpoint:** `/pharmacy/dashboard/summary`
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "total_prescriptions": 340,
    "pending_prescriptions": 24,
    "completed_prescriptions": 112,
    "out_of_stock_medicines": 33
  }
}
```

#### **2. Get Prescription Queue**
* **HTTP Method:** `GET`
* **Endpoint:** `/pharmacy/prescriptions/queue`
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "prescription_id": "rx-uuid-4444",
      "token_number": "RX-802",
      "patient_name": "Priyanka Nair",
      "status": "PENDING"
    }
  ]
}
```

---

### **Screen 2: Prescription details & active cart**

#### **3. Get Prescription Details**
* **HTTP Method:** `GET`
* **Endpoint:** `/pharmacy/{prescription_id}`
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "rx-uuid-4444",
    "doctor_name": "Dr. Tanvi Shrivastava",
    "items": [
      { "medicine_name": "Amoxicillin 500mg", "dosage": "1 capsule", "frequency": "TDS" }
    ]
  }
}
```

#### **4. Initialize Dispense Cart**
* **HTTP Method:** `POST`
* **Endpoint:** `/pharmacy/carts`
* **Request Body:**
```json
{
  "prescription_id": "rx-uuid-4444",
  "cart_type": "PRESCRIPTION"
}
```
* **Success Response (`201 Created`):**
```json
{
  "success": true,
  "code": 201,
  "data": { "cart_id": "cart-uuid-9999" }
}
```

#### **5. Add Item to Cart**
* **HTTP Method:** `POST`
* **Endpoint:** `/pharmacy/carts/{id}/items`
* **Request Body:**
```json
{
  "medicine_id": "med-uuid-001",
  "quantity": 15,
  "batch_id": "batch-uuid-099"
}
```
* **Success Response (`201 Created`):**
```json
{ "success": true, "code": 201, "message": "Item added to cart" }
```

#### **6. Checkout / Finalize Dispensing**
* **HTTP Method:** `POST`
* **Endpoint:** `/pharmacy/carts/{id}/checkout`
* **Request Body:**
```json
{
  "payment_mode": "CASH"
}
```
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "message": "Checkout completed successfully. Receipt generated."
}
```

---

### **Screen 3: Catalogue search & Batch details**

#### **7. Search Medicines**
* **HTTP Method:** `GET`
* **Endpoint:** `/pharmacy/medicines`
* **Query Parameters:**
  * `q` (String, Optional): Search keyword.
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": [
    { "id": "med-uuid-001", "name": "Paracetamol 650mg", "unit_price": 15.50 }
  ]
}
```

#### **8. Get Batches for Medicine**
* **HTTP Method:** `GET`
* **Endpoint:** `/pharmacy/inventory/batches`
* **Query Parameters:**
  * `medicine_id` (UUID, Required)
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": [
    { "batch_id": "batch-uuid-099", "batch_number": "B-9023", "expiry_date": "2028-10-31", "stock_qty": 500 }
  ]
}
```

---

### **Screen 4: Billing History & Invoices**

#### **9. Get Billing Transactions (Report Filters Enabled)**
* **HTTP Method:** `GET`
* **Endpoint:** `/pharmacy/billing/invoices`
* **Query Parameters (All Optional):**
  * `date_from` / `date_to` (Date): Filter invoice registration timestamp.
  * `category` (String): Filter medicine category.
  * `department` (String): `OPD_VISIT` or `IPD_ADMISSION`.
  * `payment_modes` (String): Comma-separated list (e.g. `CASH,UPI`).
  * `shift_type` (String): `MORNING`, `EVENING`, `NIGHT`.
  * `return_reason` (String): Match return ticket rationale.
* **Success Response (`200 OK`):**
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "bill_no": "BILL-2026-904",
      "subtotal": 600.00,
      "gst_amount": 72.00,
      "total_amount": 672.00,
      "payment_mode": "UPI",
      "created_at": "2026-07-13T05:15:00Z"
    }
  ]
}
```
