# Arovita HMS — Clinical Domain Schema Flows

This document details the database schema flows and table relationships for the three core clinical domains: **Outpatient Department (OPD)**, **Inpatient Department (IPD)**, and **Surgery / Operating Theatre (OT) Procedures**.

---

## 1. Outpatient Department (OPD) Flow

The OPD flow manages scheduling, receptionist check-in, doctor consultation, prescription drafting, and diagnostic orders.

### 1.1 Table Relationships & Entities
```
 [scheduling.appointments]
          │ (1:1 / 1:N)
          ▼
 [clinical.opd_visits] ──(1:N)──► [clinical.opd_symptoms]
          │ (1:1)
          ▼
 [clinical.consultations]
          ├──(1:N)──► [pharmacy.prescriptions] ──► [pharmacy.prescribed_medicines]
          └──(1:N)──► [diagnostic.diagnostic_orders]
                            ├──► [diagnostic.lab_order_items]
                            └──► [diagnostic.imaging_order_items]
```

### 1.2 Step-by-Step Flow

1. **Appointment Creation**:
   - Record created in `scheduling.appointments`.
   - Links `patient_id` (`patient.patients`) and `doctor_id` (`public.sys_users`).
   - Default status set to `'SCHEDULED'`.

2. **Receptionist Check-In**:
   - When the patient arrives, the receptionist changes the appointment status to `'ARRIVED'`.
   - Instantiates a record in `clinical.opd_visits` with a unique, sequential `visit_number`. Sets status to `'SCHEDULED'` (meaning scheduled for doctor review).

3. **Symptom Log**:
   - Symptoms are logged via `clinical.opd_symptoms` (linked via `opd_visit_id`).

4. **Doctor Consultation**:
   - The doctor moves the visit status to `'IN_PROGRESS'`.
   - Creates a record in `clinical.consultations` to capture `chief_complaint`, `diagnosis`, and `icd_code`.
   - Prescriptions are written to `pharmacy.prescriptions` and mapped to individual line items in `pharmacy.prescribed_medicines` (linked to `pharmacy.medicine_master` IDs).
   - Lab/radiology orders are written to `diagnostic.diagnostic_orders`, which spawn rows in `diagnostic.lab_order_items` or `diagnostic.imaging_order_items`.

5. **Visit Completion**:
   - Doctor signs the consultation (setting `signed_at` and `signed_by`).
   - The `clinical.opd_visits` status transitions to `'COMPLETED'`.
   - The billing module is now clear to generate the final outpatient invoice.

---

## 2. Inpatient Department (IPD) Flow

The IPD flow manages ward assignments, bed allocations, daily clinical progression notes, and structured discharge clearances.

### 2.1 Table Relationships & Entities
```
 [clinical.opd_visits] / [ipd.emergency_visits]  (Admission recommended)
                       │
                       ▼
             [ipd.ipd_admissions] ◄──(RLS Scoped)──► [ipd.beds] / [ipd.wards]
                       │
         ┌─────────────┴─────────────┐
         ▼                           ▼
 [ipd.ipd_daily_notes]      [clinical.vitals]
 (Doctor/Nurse entries)    (Ward vital logs)
```

### 2.2 Step-by-Step Flow

1. **Admission Recommendation**:
   - Triggered from an OPD consultation or Emergency visit where `decision = 'ADMIT'`.

2. **Bed Allocation**:
   - Receptionist queries available rooms in `ipd.beds` filtering by `ward_id` where `status = 'AVAILABLE'`.
   - Reserves the bed by updating `ipd.beds.status = 'RESERVED'`.

3. **Admitting the Patient**:
   - A new row is inserted into `ipd.ipd_admissions` mapping `patient_id`, `admission_number`, `bed_id`, `ward_id`, and `admitting_doctor_id`.
   - The bed status is updated to `ipd.beds.status = 'OCCUPIED'`.

4. **Inpatient Ward Progression**:
   - Vitals are recorded periodically in `clinical.vitals` with a reference to the `ipd_admission_id`.
   - Daily doctor rounds and nursing shifts record progression text in `ipd.ipd_daily_notes` (using enum `note_type` of `'DOCTOR'` or `'NURSE'`).

5. **Discharge Process**:
   - The doctor marks the admission status as `'READY_FOR_DISCHARGE'`.
   - Clearance records are verified across multiple gates (Billing, Nursing, Pharmacy, Insurance) to ensure all payments are collected and medications returned.
   - Upon all clearances returning `status = 'CLEARED'`, the receptionist executes the discharge.
   - Updates `ipd.ipd_admissions` with `discharge_date` and status to `'DISCHARGED'`.
   - Releases the bed: `ipd.beds.status = 'AVAILABLE'`.

---

## 3. Surgery & Operating Theatre (OT) Flow

Surgeries and procedures flow from orders inside a consultation, pre-surgical clearance checklists, execution inside the Operating Theatre, and postoperative recovery transfers.

### 3.1 Table Relationships & Entities
```
 [clinical.consultations]
          │
          ▼
 [clinical.service_orders] (type = 'SURGERY' / 'PROCEDURE')
          │
          ▼
  [ipd.ot_sessions] ◄──► [clinical.surgery_procedure_catalogue]
          │ (Surgeon, Anaesthetist, Nurse, Room ID)
          │
          └──► Post-Op Recovery ──► Ward Transfer ([ipd.beds])
```

### 3.2 Step-by-Step Flow

1. **Surgery Ordering**:
   - During a consultation, the physician creates a surgical request by inserting a row in `clinical.service_orders`.
   - Sets `order_type = 'SURGERY'` (or `'PROCEDURE'`), mapping to a lookup ID in `clinical.surgery_procedure_catalogue` to pull estimated costs.

2. **PAC (Pre-Anaesthetic Checkup) Clearance**:
   - An anaesthetist reviews the patient.
   - Logs checkup metrics using the custom form templates (`clinical.dept_consultation_forms` or `clinical.patient_notes`).
   - Clears the patient for surgery by updating the booked OT session or service order.

3. **OT Scheduling**:
   - Creates a record in `ipd.ot_sessions` linking `service_order_id`, `patient_id`, `surgeon_id`, `anaesthetist_id`, `scrub_nurse_id`, `scheduled_start`, `scheduled_end`, and `status = 'SCHEDULED'`.

4. **Surgical Checklist & Checklist Verification**:
   - Immediately prior to incision, the nursing team runs the pre-op checklist.
   - Updates `ipd.ot_sessions` setting `pac_cleared = TRUE`, updating `pre_op_checklist` JSONB parameters, and setting `status = 'PRE_OP'`.

5. **Intra-Operative Stage**:
   - The surgical team sets `status = 'IN_PROGRESS'` and enters `surgery_start_at`.
   - Real-time details are written in `intra_op_notes` and any critical vitals recorded.

6. **Post-Operative Recovery & Transfer**:
   - The surgery completes: `status = 'POST_OP'`. Sets `surgery_end_at` and `recovery_start_at`.
   - Upon recovery completion, `recovery_notes` are finalized, and the patient is transferred back to a ward.
   - Updates `transferred_to_ward_id` and `transferred_to_bed_id`.
   - Bed status is changed to `OCCUPIED`.
   - OT Session transitions to `status = 'COMPLETED'`.
