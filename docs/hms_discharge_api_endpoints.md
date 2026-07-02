# HMS Discharge Module — API Endpoints Reference

This document provides a comprehensive reference of the API endpoints involved in the Patient Discharge and DAMA (Discharge Against Medical Advice) workflows, including request/response bodies, query parameters, and authorized roles.

---

## 1. Discharge Operations (`ops` service)

### POST `/ops/discharge/initiate`
* **Description**: Transitions the patient admission status from `UNDER_TREATMENT` to `READY_FOR_DISCHARGE`.
* **Required Permission**: `ipd:manage`
* **Authorized Roles**: Doctor (`MED-001`), Head Nurse (`MED-003`), Hospital Admin (`ADM-001`)
* **Request Headers**:
  ```http
  Authorization: Bearer <access_token>
  ```
* **Request Body**:
  ```json
  {
    "admission_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
  }
  ```
* **Response Body (`200 OK`)**:
  ```json
  {
    "success": true,
    "status": 200,
    "message": "Discharge initiated — admission moved to READY_FOR_DISCHARGE",
    "data": {
      "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
      "tenant_id": "d3b07384-d113-4c4f-9e7b-eb1df0b89f55",
      "patient_id": "7a5c898b-24b3-4616-92d3-138374d6c41b",
      "ipd_status": "READY_FOR_DISCHARGE",
      "status": "ADMITTED"
    }
  }
  ```

---

### POST `/ops/discharge/complete`
* **Description**: Finalizes a normal discharge by verifying that all gates/clearances in `ops.discharge_clearances` are `CLEARED`, updating the admission lifecycle status to `DISCHARGED`, releasing the bed, and triggering auto-billing finalizations.
* **Required Permission**: `ipd:manage`
* **Authorized Roles**: Doctor (`MED-001`), Head Nurse (`MED-003`), Hospital Admin (`ADM-001`)
* **Request Body**:
  ```json
  {
    "admission_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "discharge_summary": "Patient is stable and fit for discharge. Follow-up in 1 week.",
    "follow_up_date": "2026-06-09",
    "discharge_type": "NORMAL"
  }
  ```
* **Response Body (`200 OK`)**:
  ```json
  {
    "success": true,
    "status": 200,
    "message": "Discharge completed successfully",
    "data": {
      "admission_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
      "discharge_type": "NORMAL",
      "status": "DISCHARGED",
      "bed_released": true
    }
  }
  ```
* **Error Response (`409 Conflict` - Gates Pending)**:
  ```json
  {
    "success": false,
    "status": 409,
    "error": "ConflictError",
    "message": "Cannot complete discharge: pending clearances for gates: BILLING, NURSING"
  }
  ```

---

### GET `/ops/discharge/{admission_id}`
* **Description**: Retrieves the complete discharge summary details for an admission, including the nursing checklist state and clearance checkpoints.
* **Required Permission**: `ipd:view`
* **Authorized Roles**: Doctor (`MED-001`), Nurse (`MED-002`), Head Nurse (`MED-003`), Receptionist (`ADM-002`), Hospital Admin (`ADM-001`)
* **Response Body (`200 OK`)**:
  ```json
  {
    "success": true,
    "status": 200,
    "data": {
      "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
      "tenant_id": "d3b07384-d113-4c4f-9e7b-eb1df0b89f55",
      "patient_id": "7a5c898b-24b3-4616-92d3-138374d6c41b",
      "patient_name": "Rajan Pillai",
      "mrn": "MRN-202604-A1B2C3",
      "date_of_birth": "1978-03-12",
      "blood_group": "B+",
      "ward_name": "General Ward A",
      "bed_number": "GW-102",
      "ipd_status": "READY_FOR_DISCHARGE",
      "status": "ADMITTED",
      "checklist_complete": true,
      "billing_ready": true,
      "pharmacy_ready": true
    }
  }
  ```

---

## 2. Nursing & Case Management Operations (`nurse` service)

### GET `/nurses/discharge/cases`
* **Description**: Returns active enterprise discharge workflows/cases to be managed.
* **Authorized Roles**: Nurse (`MED-002`), Head Nurse (`MED-003`), Receptionist (`ADM-002`), Hospital Admin (`ADM-001`)
* **Query Parameters** (All optional):
  * `discharge_stage` (e.g. `BILLING_PENDING`, `SUMMARY_PENDING`)
  * `ward_id` (UUID)
  * `search` (patient name / MRN)
  * `discharge_type` (e.g. `NORMAL`, `DAMA`)
  * `from_date` (YYYY-MM-DD)
  * `to_date` (YYYY-MM-DD)
* **Response Body (`200 OK`)**:
  ```json
  {
    "success": true,
    "status": 200,
    "data": {
      "cases": [
        {
          "discharge_case_id": "8f8a1234-abcd-4f5e-9999-e68b19a77777",
          "admission_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
          "patient": {
            "id": "7a5c898b-24b3-4616-92d3-138374d6c41b",
            "name": "Rajan Pillai",
            "mrn": "MRN-202604-A1B2C3",
            "gender": "MALE",
            "age": 48
          },
          "ward": {
            "id": "c88b0222-3a3a-4444-8888-dddddddddddd",
            "name": "General Ward A",
            "bed_number": "GW-102"
          },
          "doctor": {
            "id": "d00a1111-2b2b-3c3c-4d4d-eeeeeeeeeeee",
            "name": "Dr. Suresh Kumar"
          },
          "patient_status": "READY_FOR_DISCHARGE",
          "discharge_type": "NORMAL",
          "discharge_stage": "BILLING_PENDING",
          "all_gates_cleared": false,
          "billing_status": "PENDING",
          "summary_submitted": false,
          "initiated_at": "2026-06-02T10:30:00Z",
          "delay_minutes": 45
        }
      ]
    }
  }
  ```

---

### GET `/nurses/discharge/cases/{admission_id}`
* **Description**: Returns detailed workflow, clearance checklist, and status for a single active discharge case.
* **Authorized Roles**: Nurse (`MED-002`), Head Nurse (`MED-003`), Receptionist (`ADM-002`), Hospital Admin (`ADM-001`)
* **Response Body (`200 OK`)**:
  ```json
  {
    "success": true,
    "status": 200,
    "data": {
      "discharge_case_id": "8f8a1234-abcd-4f5e-9999-e68b19a77777",
      "admission_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
      "patient": {
        "id": "7a5c898b-24b3-4616-92d3-138374d6c41b",
        "name": "Rajan Pillai",
        "mrn": "MRN-202604-A1B2C3",
        "gender": "MALE",
        "age": 48
      },
      "doctor": {
        "id": "d00a1111-2b2b-3c3c-4d4d-eeeeeeeeeeee",
        "name": "Dr. Suresh Kumar"
      },
      "ward": {
        "id": "c88b0222-3a3a-4444-8888-dddddddddddd",
        "name": "General Ward A",
        "bed_number": "GW-102"
      },
      "discharge": {
        "discharge_type": "NORMAL",
        "current_stage": "BILLING_PENDING",
        "patient_status": "READY_FOR_DISCHARGE",
        "initiated_at": "2026-06-02T10:30:00Z",
        "summary_submitted": false,
        "all_gates_cleared": false
      },
      "clearances": [
        {
          "gate_name": "BILLING",
          "gate_status": "PENDING",
          "cleared_by": null,
          "cleared_at": null,
          "blocking_reason": null
        },
        {
          "gate_name": "NURSING",
          "gate_status": "CLEARED",
          "cleared_by": "nurs-7777-8888-9999-101010101010",
          "cleared_at": "2026-06-02T10:45:00Z",
          "blocking_reason": null
        }
      ],
      "checklist": {
        "vitals_stable": true,
        "meds_explained": true,
        "discharge_education_done": true,
        "billing_ready": false,
        "all_complete": false
      }
    }
  }
  ```

---

### PATCH `/nurses/admissions/{admission_id}/discharge-checklist`
* **Description**: Updates individual checklist items. Once all items are marked `true`, the system automatically moves `all_complete` to `true` and triggers notifications to the attending doctor.
* **Authorized Roles**: Nurse (`MED-002`), Head Nurse (`MED-003`), Receptionist (`ADM-002`), Hospital Admin (`ADM-001`)
* **Request Body**:
  ```json
  {
    "vitals_stable": true,
    "meds_explained": true,
    "discharge_education_done": true,
    "wound_instructions_given": true,
    "belongings_handed": true,
    "billing_ready": true,
    "pharmacy_ready": true,
    "transport_arranged": true,
    "notes": "Belongings handed over to wife. Patient understands medication schedules."
  }
  ```
* **Response Body (`200 OK`)**:
  ```json
  {
    "success": true,
    "status": 200,
    "data": {
      "id": "chk-9999-8888-7777-666666666666",
      "admission_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
      "patient_id": "7a5c898b-24b3-4616-92d3-138374d6c41b",
      "nurse_id": "nurs-7777-8888-9999-101010101010",
      "vitals_stable": true,
      "meds_explained": true,
      "discharge_education_done": true,
      "wound_instructions_given": true,
      "belongings_handed": true,
      "billing_ready": true,
      "pharmacy_ready": true,
      "transport_arranged": true,
      "all_complete": true,
      "notes": "Belongings handed over to wife. Patient understands medication schedules.",
      "completed_at": "2026-06-02T10:45:00Z",
      "created_at": "2026-06-02T10:35:00Z",
      "updated_at": "2026-06-02T10:45:00Z"
    }
  }
  ```

---

### GET `/nurses/admissions/{admission_id}/discharge-checklist`
* **Description**: Returns the current checklist progress for a patient's admission.
* **Authorized Roles**: Doctor (`MED-001`), Nurse (`MED-002`), Head Nurse (`MED-003`), Receptionist (`ADM-002`), Hospital Admin (`ADM-001`)
* **Response Body (`200 OK`)**:
  *(Same structure as response body of PATCH `/nurses/admissions/{admission_id}/discharge-checklist`)*

---

## 3. DAMA Operations (`ops` service)

### POST `/ops/dama/initiate`
* **Description**: Initiates a DAMA (Discharge Against Medical Advice) workflow for an active admission.
* **Required Permission**: `ipd:manage`
* **Authorized Roles**: Doctor (`MED-001`), Head Nurse (`MED-003`), Hospital Admin (`ADM-001`)
* **Request Body**:
  ```json
  {
    "admission_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "reason": "Family wants to travel back to home town immediately",
    "clinical_summary": "Patient is in stable condition but post-op recovery is incomplete.",
    "requires_approval": true
  }
  ```
* **Response Body (`201 Created`)**:
  ```json
  {
    "success": true,
    "status": 201,
    "message": "DAMA request initiated",
    "data": {
      "id": "dama-1111-2222-3333-444444444444",
      "admission_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
      "patient_id": "7a5c898b-24b3-4616-92d3-138374d6c41b",
      "initiated_by": "doc-0000-1111-2222-333333333333",
      "reason": "Family wants to travel back to home town immediately",
      "clinical_summary": "Patient is in stable condition but post-op recovery is incomplete.",
      "requires_approval": true,
      "status": "INITIATED",
      "consent_obtained": false
    }
  }
  ```

---

### POST `/ops/dama/consent`
* **Description**: Records the family member or patient consent and signature details for DAMA.
* **Required Permission**: `appointments:manage`
* **Authorized Roles**: Receptionist (`ADM-002`), Hospital Admin (`ADM-001`), Head Nurse (`MED-003`)
* **Request Body**:
  ```json
  {
    "admission_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "consent_by": "Priya Venkataraman",
    "consent_relation": "SPOUSE",
    "consent_document_key": "dama/consents/3fa85f64-5717-4562-b3fc-2c963f66afa6.pdf"
  }
  ```
* **Response Body (`200 OK`)**:
  ```json
  {
    "success": true,
    "status": 200,
    "message": "DAMA consent recorded",
    "data": {
      "id": "dama-1111-2222-3333-444444444444",
      "status": "PENDING_APPROVAL",
      "consent_obtained": true,
      "consent_by": "Priya Venkataraman",
      "consent_relation": "SPOUSE",
      "consent_document_key": "dama/consents/3fa85f64-5717-4562-b3fc-2c963f66afa6.pdf"
    }
  }
  ```

---

### POST `/ops/dama/approve`
* **Description**: Approves a DAMA request that required senior doctor or management approval.
* **Required Permission**: `appointments:manage`
* **Authorized Roles**: Senior Doctor, Hospital Admin (`ADM-001`)
* **Request Body**:
  ```json
  {
    "admission_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "approval_notes": "Consent document verified. Risks explained to spouse. Approved."
  }
  ```
* **Response Body (`200 OK`)**:
  ```json
  {
    "success": true,
    "status": 200,
    "message": "DAMA approved",
    "data": {
      "id": "dama-1111-2222-3333-444444444444",
      "status": "APPROVED",
      "approved_by": "admin-9999-8888-7777-666666666666",
      "approved_at": "2026-06-02T11:15:00Z",
      "approval_notes": "Consent document verified. Risks explained to spouse. Approved."
    }
  }
  ```

---

### POST `/ops/dama/complete`
* **Description**: Completes the DAMA discharge, setting the patient status to `DISCHARGED`, releasing the bed, and generating the unbilled bed and clinical charges.
* **Required Permission**: `appointments:manage`
* **Authorized Roles**: Doctor (`MED-001`), Head Nurse (`MED-003`), Hospital Admin (`ADM-001`), Receptionist (`ADM-002`)
* **Request Body**:
  ```json
  {
    "admission_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "discharge_notes": "Patient left AMA with spouse. Discharge summary provided."
  }
  ```
* **Response Body (`200 OK`)**:
  ```json
  {
    "success": true,
    "status": 200,
    "message": "DAMA discharge completed — patient left AMA",
    "data": {
      "id": "dama-1111-2222-3333-444444444444",
      "status": "COMPLETED",
      "completed_by": "doc-0000-1111-2222-333333333333",
      "completed_at": "2026-06-02T11:30:00Z"
    }
  }
  ```

---

### POST `/ops/dama/cancel`
* **Description**: Cancels an active DAMA request when the patient or family decides to stay.
* **Required Permission**: `appointments:manage`
* **Authorized Roles**: Doctor (`MED-001`), Head Nurse (`MED-003`), Hospital Admin (`ADM-001`), Receptionist (`ADM-002`)
* **Request Body**:
  ```json
  {
    "admission_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "cancellation_reason": "Spouse agreed to continue treatment after counselling."
  }
  ```
* **Response Body (`200 OK`)**:
  ```json
  {
    "success": true,
    "status": 200,
    "message": "DAMA cancelled",
    "data": {
      "id": "dama-1111-2222-3333-444444444444",
      "status": "CANCELLED",
      "cancellation_reason": "Spouse agreed to continue treatment after counselling."
    }
  }
  ```

---

## 4. Billing Operations (`billing` service)

### GET `/billing/clearance/{admission_id}`
* **Description**: Checks whether the patient's outstanding balance is zero and returns the clearance status.
* **Authorized Roles**: Receptionist (`ADM-002`), Hospital Admin (`ADM-001`), Nurse (`MED-002`), Head Nurse (`MED-003`), Doctor (`MED-001`)
* **Response Body (`200 OK`)**:
  ```json
  {
    "success": true,
    "status": 200,
    "data": {
      "admission_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
      "patient_id": "7a5c898b-24b3-4616-92d3-138374d6c41b",
      "outstanding": 0.00,
      "cleared": true
    }
  }
  ```
