# HMS Nursing Module — API Endpoints Documentation

This document describes all API endpoints exposed by the **HMS Nursing Module**. 

All endpoints require a valid JWT token passed in the `Authorization` header (`Bearer <token>`), except where noted.
* **Write Operations (POST, PATCH, DELETE)**: Require a user with role `NURSE` (`MED-002`) or `HEAD_NURSE` (`MED-003`).
* **Read Operations (GET)**: Permitted for roles `NURSE`, `HEAD_NURSE`, `DOCTOR`, or `ADMIN`.

---

## 📋 Table of Contents
1. [General & Health Check](#1-general--health-check)
2. [Patient Census & Overview](#2-patient-census--overview)
3. [Nurse Management & Assignments](#3-nurse-management--assignments)
4. [Shift & Handovers](#4-shift--handovers)
5. [Nursing Notes](#5-nursing-notes)
6. [Discharge Preparation](#6-discharge-preparation)
7. [Bed & Ward Management](#7-bed--ward-management)

---

## 1. General & Health Check

### GET `/nurses/health`
* **Description**: Verify the health and status of the Nurse microservice.
* **Authentication**: None (Public)
* **Query Parameters**: None
* **Response (200 OK)**:
  ```json
  {
    "success": true,
    "code": 200,
    "data": {
      "status": "ok",
      "service": "hms-nurse"
    }
  }
  ```

### GET `/nurses/me`
* **Description**: Retrieve active profile, assignments, and permissions for the logged-in nurse user.
* **Authentication**: Required (`Bearer <token>`)
* **Query Parameters**: None
* **Response (200 OK)**:
  ```json
  {
    "success": true,
    "code": 200,
    "data": {
      "user": {
        "user_id": "364d3daa-470d-41bf-890b-3947d9285a9c",
        "employee_id": "HNR-GDGHUT",
        "full_name": "Meera Joshi",
        "email": "meera.joshi@arovita.com",
        "phone": "+919876541004",
        "role": "HEAD_NURSE",
        "role_id": "MED-003",
        "tenant_id": "add22d47-2bc7-4bd2-9221-75fb580289d0",
        "department_id": "3f4d7b9e-8d2a-4d0b-a76f-29a8f2d31b44",
        "availability_status": "ASSIGNED",
        "assignments": [
          {
            "assignment_id": "4318a2c2-761c-4e02-8622-8686f889ba13",
            "work_area": "ICU",
            "work_area_id": "57df5ac1-fbf6-4ad8-b617-75a5133c2c56",
            "ward_id": "0c1c6410-7ed4-49e7-8c57-92e60421f3e7",
            "ward_name": "ICU",
            "shift_type": "EVENING"
          }
        ]
      },
      "permissions": ["nurse:beds:manage", "nurse:notes:create", "nurse:vitals:create"],
      "work_areas": [
        {
          "work_area": "IPD",
          "work_area_id": "266f2545-64a9-45b5-a8f5-f3eee48db109"
        }
      ]
    }
  }
  ```

---

## 2. Patient Census & Overview

### GET `/nurses/patients`
* **Description**: Fetch all patients currently admitted in a specified care area (IPD/ICU/OPD/OT).
* **Query Parameters**:
  * `care_area` (Optional, String): Filter by care area. Options: `IPD`, `ICU`, `OT`, `OPD`.
  * `nurse_user_id` (Optional, UUID): Filter patients assigned to a specific nurse.
  * `queue_date` (Optional, Date `YYYY-MM-DD`): Default is today's date (primarily used for OPD queues).
  * `page` (Optional, Integer): Default is `1`.
  * `page_size` (Optional, Integer): Default is `20`, Max is `100`.
* **Response (200 OK)**:
  ```json
  {
    "success": true,
    "code": 200,
    "data": {
      "patients": [
        {
          "patient_id": "1799deab-e735-4061-8222-458d728d663f",
          "uid": "MRN-202605-0285",
          "full_name": "Rakshan Dodmani",
          "age": 25,
          "gender": "MALE",
          "admission_id": "1fc666a1-719b-4fb8-a4a5-d4ee1b3d79ce",
          "emergency_visit_id": "8b51d023-e18e-49b0-a612-42173f27f29f",
          "admission_status": "ADMITTED",
          "ipd_status": "UNDER_TREATMENT",
          "patient_status": "MONITORING",
          "care_area": "IPD",
          "ward_id": "830023d4-7375-4e3a-9f12-d725a63d8f3a",
          "ward_name": "General Ward A",
          "bed": "GEN-05",
          "doctor_id": "cedb19f5-29da-4917-bf63-de47e2463d84",
          "doctor": "Dr. KS Walia",
          "diagnosis": "Post-op observation",
          "available_actions": ["RECORD_VITALS", "ADMINISTER_MEDICATION", "ADD_NOTE"],
          "assigned_nurse": null
        }
      ],
      "total": 1,
      "page": 1,
      "page_size": 20,
      "total_pages": 1
    }
  }
  ```

### GET `/nurses/patients/{patient_id}`
* **Description**: Retrieve a detailed clinical snapshot of an active patient's admission.
* **Path Parameters**:
  * `patient_id` (Mandatory, UUID)
* **Response (200 OK)**:
  ```json
  {
    "success": true,
    "code": 200,
    "data": {
      "patient_id": "1799deab-e735-4061-8222-458d728d663f",
      "uid": "0285",
      "full_name": "Rakshan Dodmani",
      "age": 25,
      "gender": "M",
      "bed": "GEN-05",
      "department": "General Ward A",
      "diagnosis": "Post-op observation",
      "assigned_nurse": null,
      "admitted_date": "22 May",
      "attending_doctor": "Dr. KS Walia",
      "status": "MONITORING",
      "admission_status": "ADMITTED",
      "patient_status": "MONITORING",
      "day_of_admission": 31,
      "latest_vitals": {
        "bp": "120/80",
        "spo2": 98,
        "hr": 72,
        "recorded_at": "18:25"
      },
      "pending_meds": 2,
      "pending_tasks": 1,
      "nursing_notes_count": 5
    }
  }
  ```

### GET `/nurses/my-patients`
* **Description**: Fetch the list of patients currently assigned to the logged-in nurse (based on their active ward/unit shift assignment).
* **Query Parameters**:
  * `nurse_user_id` (Optional, UUID): Retrieve for a specific nurse (Head Nurse permission required).
  * `page` (Optional, Integer): Default is `1`.
  * `page_size` (Optional, Integer): Default is `20`.
* **Response (200 OK)**: Same structure as `GET /nurses/patients`.

### GET `/nurses/patients/assignable`
* **Description**: List all currently admitted patients in a ward who have not yet been assigned to any nurse.
* **Query Parameters**:
  * `work_area` (Optional/Contextual, String): Options: `ICU`, `IPD`, `OT`.
  * `ward_id` (Mandatory, UUID): The UUID of the ward.
* **Response (200 OK)**:
  ```json
  {
    "success": true,
    "code": 200,
    "data": [
      {
        "patient_id": "9b45af50-4589-4fb7-8c08-d6c06ddc6cf1",
        "admission_id": "b7616183-852b-47b4-93c1-8592c0f4d57e",
        "uid": "MRN-202605-0259",
        "full_name": "Sai Karthik",
        "ward_id": "830023d4-7375-4e3a-9f12-d725a63d8f3a",
        "ward_name": "General Ward A",
        "bed_number": "GWA-06",
        "diagnosis": "Fissure",
        "admitted_at": "2026-05-15T10:00:00Z"
      }
    ]
  }
  ```

---

## 3. Nurse Management & Assignments

### GET `/nurses/list` (or `/nurses`)
* **Description**: Retrieve a list of nurses in the system, filtered by shift, work area, status, or search query.
* **Query Parameters**:
  * `search` (Optional, String): Search by nurse name or employee code.
  * `department_id` (Optional, UUID)
  * `ward_id` (Optional, UUID)
  * `unit_id` (Optional, UUID)
  * `work_area` (Optional, String): `ICU`, `IPD`, `OT`, `OPD`.
  * `shift_type` (Optional, String): `MORNING`, `AFTERNOON`, `EVENING`, `NIGHT`.
  * `status` (Optional, String): `ACTIVE`, `INACTIVE`, `ON_DUTY`, `OFF_DUTY`, `ON_LEAVE`.
  * `page` (Optional, Integer): Default is `1`.
  * `page_size` (Optional, Integer): Default is `20`.
* **Response (200 OK)**:
  ```json
  {
    "success": true,
    "code": 200,
    "data": {
      "summary": {
        "total_nurses": 45,
        "on_duty": 12,
        "on_leave": 2,
        "off_duty": 31,
        "critical_alerts": 0
      },
      "items": [
        {
          "user_id": "364d3daa-470d-41bf-890b-3947d9285a9c",
          "employee_id": "HNR-GDGHUT",
          "full_name": "Meera Joshi",
          "email": "meera.joshi@arovita.com",
          "phone": "+919876541004",
          "designation": "HEAD_NURSE",
          "is_active": true,
          "availability_status": "ASSIGNED",
          "active_assignment_count": 1,
          "max_assignment_capacity": 5,
          "assignments": [...]
        }
      ],
      "meta": {
        "page": 1,
        "page_size": 20,
        "total_items": 45,
        "total_pages": 3
      }
    }
  }
  ```

### GET `/nurses/{nurse_user_id}`
* **Description**: Retrieve a detailed profile of a nurse including active shift assignments.
* **Path Parameters**:
  * `nurse_user_id` (Mandatory, UUID)
* **Response (200 OK)**: Same structure as a single item in the `GET /nurses/list` response.

### POST `/nurses/assignments`
* **Description**: Create a shift assignment mapping a nurse to a specific work area, ward/unit, and optionally patient list.
* **Request Body (JSON)**:
  * `nurse_user_id` (Mandatory, UUID): Nurse receiving the assignment.
  * `work_area` (Optional, String): `ICU`, `IPD`, `OT`, `OPD` (Mandatory if `work_area_id` is missing).
  * `work_area_id` (Optional, UUID)
  * `ward_id` (Optional, UUID): Mandatory for `ICU`, `IPD`, `OT`.
  * `department_id` (Optional, UUID): Mandatory for `OPD` (maps to department).
  * `unit_id` (Optional, UUID)
  * `shift_type` (Mandatory, String): `MORNING`, `AFTERNOON`, `EVENING`, `NIGHT`.
  * `start_date` (Mandatory, Date `YYYY-MM-DD`)
  * `end_date` (Mandatory, Date `YYYY-MM-DD`)
  * `patient_ids` (Optional, Array of UUIDs)
  * `notes` (Optional, String, Max length 2000)
* **Response (201 Created)**:
  ```json
  {
    "success": true,
    "code": 201,
    "message": "Nurse assignment created",
    "data": {
      "assignment_id": "e0a2948b-302a-434b-b29d-92a184efbcda",
      "nurse_user_id": "364d3daa-470d-41bf-890b-3947d9285a9c",
      "work_area": "IPD",
      "ward_id": "830023d4-7375-4e3a-9f12-d725a63d8f3a",
      "shift_type": "MORNING",
      "start_date": "2026-06-22",
      "end_date": "2026-06-22",
      "assigned_patient_count": 0,
      "is_active": true
    }
  }
  ```

### DELETE `/nurses/assignments/{assignment_id}`
* **Description**: Cancel or deactivate an active nurse assignment.
* **Path Parameters**:
  * `assignment_id` (Mandatory, UUID)
* **Request Body (JSON)**:
  * `nurse_user_id` (Mandatory, UUID)
  * `reason` (Optional, String, Max length 1000)
* **Response (200 OK)**:
  ```json
  {
    "success": true,
    "code": 200,
    "message": "Nurse assignment cancelled",
    "data": true
  }
  ```

### GET `/nurses/assignments`
* **Description**: Search and query all active or historical nurse assignments across facilities.
* **Query Parameters**: Same filters as `GET /nurses/list` (e.g. `nurse_user_id`, `work_area`, `ward_id`, `shift_type`, `is_active`, `page`, `page_size`).
* **Response (200 OK)**: JSON list containing paginated assignment objects.

---

## 4. Shift & Handovers

### GET `/nurses/my-handovers`
* **Description**: Retrieve a history of shift handovers submitted or received by the logged-in nurse.
* **Query Parameters**: None
* **Response (200 OK)**:
  ```json
  {
    "success": true,
    "code": 200,
    "data": {
      "handovers": [
        {
          "handover_id": "d094f82c-473d-4c32-b29e-9184cdefbcda",
          "assignment_id": "4318a2c2-761c-4e02-8622-8686f889ba13",
          "from_nurse_id": "364d3daa-470d-41bf-890b-3947d9285a9c",
          "from_nurse_name": "Meera Joshi",
          "to_nurse_id": "e0a2948b-302a-434b-b29d-92a184efbcda",
          "to_nurse_name": "Ravi Kumar",
          "work_area": "ICU",
          "ward_id": "0c1c6410-7ed4-49e7-8c57-92e60421f3e7",
          "ward_name": "ICU",
          "shift_type": "EVENING",
          "notes": "Patients stable. Meds administered.",
          "status": "COMPLETED",
          "created_at": "2026-06-22T14:00:00Z"
        }
      ]
    }
  }
  ```

### GET `/nurses/shift/handover/available-nurses`
* **Description**: Get a list of on-duty nurses eligible to receive patient handovers for a specific shift transition.
* **Query Parameters**:
  * `assignment_id` (Mandatory, UUID): The active assignment ID of the nurse handoff.
* **Response (200 OK)**:
  ```json
  {
    "success": true,
    "code": 200,
    "data": {
      "available_nurses": [
        {
          "user_id": "e0a2948b-302a-434b-b29d-92a184efbcda",
          "full_name": "Ravi Kumar",
          "employee_id": "NR-0921"
        }
      ]
    }
  }
  ```

### POST `/nurses/shift/handover`
* **Description**: Submit a new shift handover record transferring patient care duty to the next nurse.
* **Request Body (JSON)**:
  * `assignment_id` (Mandatory, UUID): The active assignment ID of the outgoing nurse.
  * `to_nurse_id` (Mandatory, UUID): The user ID of the incoming nurse.
  * `notes` (Mandatory, String): Shift transition instructions, handoff status summary (Min length: 1, Max: 5000).
* **Response (201 Created)**:
  ```json
  {
    "success": true,
    "code": 201,
    "data": {
      "handover_id": "d094f82c-473d-4c32-b29e-9184cdefbcda",
      "assignment_id": "4318a2c2-761c-4e02-8622-8686f889ba13",
      "submitted_at": "2026-06-22T18:55:00Z",
      "from_nurse_id": "364d3daa-470d-41bf-890b-3947d9285a9c",
      "to_nurse_id": "e0a2948b-302a-434b-b29d-92a184efbcda",
      "work_area": "ICU",
      "shift_type": "EVENING"
    }
  }
  ```

---

## 5. Nursing Notes

### GET `/nurses/admissions/{admission_id}/notes`
* **Description**: Retrieve a paginated chronological feed of nursing notes added for a patient.
* **Path Parameters**:
  * `admission_id` (Mandatory, UUID)
* **Query Parameters**:
  * `note_type` (Optional, String): Filter by type (e.g. `SHIFT_HANDOVER`, `WOUND_NOTE`, `INTAKE_OUTPUT`, `GENERAL`).
  * `shift` (Optional, String): `DAY`, `EVENING`, `NIGHT`.
  * `page` (Optional, Integer): Default `1`.
  * `page_size` (Optional, Integer): Default `20`.
* **Response (200 OK)**:
  ```json
  {
    "success": true,
    "code": 200,
    "data": {
      "admission_id": "1fc666a1-719b-4fb8-a4a5-d4ee1b3d79ce",
      "items": [
        {
          "id": "n09da8b1-309a-4fd2-8b29-b2a8fbcd92c0",
          "admission_id": "1fc666a1-719b-4fb8-a4a5-d4ee1b3d79ce",
          "patient_id": "1799deab-e735-4061-8222-458d728d663f",
          "nurse_id": "364d3daa-470d-41bf-890b-3947d9285a9c",
          "note_type": "GENERAL",
          "content": "Patient resting comfortably. Pulse regular.",
          "shift": "DAY",
          "intake_ml": null,
          "output_ml": null,
          "is_escalation": false,
          "escalated_to": null,
          "created_at": "2026-06-22T18:50:00Z"
        }
      ],
      "total": 1,
      "page": 1,
      "page_size": 20
    }
  }
  ```

### POST `/nurses/admissions/{admission_id}/notes`
* **Description**: Add a new clinical progress note, wound care chart, intake-output sheet, or escalation memo for a patient.
* **Path Parameters**:
  * `admission_id` (Mandatory, UUID)
* **Request Body (JSON)**:
  * `patient_id` (Mandatory, UUID)
  * `note_type` (Mandatory, String): Options: `SHIFT_HANDOVER`, `WOUND_NOTE`, `INTAKE_OUTPUT`, `CATHETER_CARE`, `DRESSING`, `ESCALATION`, `GENERAL`, `SBAR`, `IV_LINE_CARE`.
  * `content` (Mandatory, String, Min length 5, Max 5000)
  * `shift` (Optional, String): `DAY`, `EVENING`, `NIGHT`. Default: `DAY`.
  * `intake_ml` (Optional, Integer, Range >= 0)
  * `output_ml` (Optional, Integer, Range >= 0)
  * `wound_location` (Optional, String)
  * `wound_size_cm` (Optional, String)
  * `wound_description` (Optional, String)
  * `is_escalation` (Optional, Boolean): If `true`, must specify `escalated_to` doctor.
  * `escalated_to` (Optional, UUID): The UUID of the doctor being alerted.
* **Response (201 Created)**: Same structure as a note item returned in GET notes list, with newly allocated UUID.

---

## 6. Discharge Preparation

### GET `/nurses/admissions/{admission_id}/discharge-checklist`
* **Description**: Retrieve the status of clinical and billing tasks on the pre-discharge checklist for a patient.
* **Path Parameters**:
  * `admission_id` (Mandatory, UUID)
* **Response (200 OK)**:
  ```json
  {
    "success": true,
    "code": 200,
    "data": {
      "id": "c09da8b1-302a-4fd2-892b-b28dafb92cda",
      "admission_id": "1fc666a1-719b-4fb8-a4a5-d4ee1b3d79ce",
      "patient_id": "1799deab-e735-4061-8222-458d728d663f",
      "nurse_id": "364d3daa-470d-41bf-890b-3947d9285a9c",
      "vitals_stable": true,
      "meds_explained": false,
      "discharge_education_done": false,
      "wound_instructions_given": false,
      "belongings_handed": false,
      "billing_ready": false,
      "pharmacy_ready": false,
      "transport_arranged": false,
      "all_complete": false,
      "notes": "Awaiting final clearance from pharmacy."
    }
  }
  ```

### PATCH `/nurses/admissions/{admission_id}/discharge-checklist`
* **Description**: Check off or update items in the discharge checklist.
* **Path Parameters**:
  * `admission_id` (Mandatory, UUID)
* **Request Body (JSON)**:
  * `vitals_stable` (Optional, Boolean)
  * `meds_explained` (Optional, Boolean)
  * `discharge_education_done` (Optional, Boolean)
  * `wound_instructions_given` (Optional, Boolean)
  * `belongings_handed` (Optional, Boolean)
  * `billing_ready` (Optional, Boolean)
  * `pharmacy_ready` (Optional, Boolean)
  * `transport_arranged` (Optional, Boolean)
  * `notes` (Optional, String, Max length 1000)
  * *Note*: At least one checklist field must be provided.
* **Response (200 OK)**: Same structure as the GET response, with updated checklist statuses.

### GET `/nurses/discharge/cases`
* **Description**: Fetch all active patients prepared or preparing for discharge (restricted to/optimized for Head Nurses).
* **Query Parameters**:
  * `discharge_stage` (Optional, String)
  * `ward_id` (Optional, UUID)
  * `search` (Optional, String)
  * `discharge_type` (Optional, String)
  * `from_date` (Optional, Date `YYYY-MM-DD`)
  * `to_date` (Optional, Date `YYYY-MM-DD`)
* **Response (200 OK)**:
  ```json
  {
    "success": true,
    "code": 200,
    "data": {
      "cases": [
        {
          "discharge_case_id": "dc9da8b1-309a-4fd2-89b2-b2a8fc092cda",
          "admission_id": "1fc666a1-719b-4fb8-a4a5-d4ee1b3d79ce",
          "patient": {
            "id": "1799deab-e735-4061-8222-458d728d663f",
            "name": "Rakshan Dodmani",
            "mrn": "MRN-202605-0285"
          },
          "ward": {
            "id": "830023d4-7375-4e3a-9f12-d725a63d8f3a",
            "name": "General Ward A",
            "bed_number": "GEN-05"
          },
          "doctor": {
            "id": "cedb19f5-29da-4917-bf63-de47e2463d84",
            "name": "Dr. KS Walia"
          },
          "discharge_type": "REGULAR",
          "discharge_stage": "CHECKLIST_PREPARATION",
          "all_gates_cleared": false,
          "billing_status": "PENDING"
        }
      ]
    }
  }
  ```

### GET `/nurses/discharge/cases/{admission_id}`
* **Description**: Retrieve granular workflow checklists and gateway clearance milestones for a specific discharge case.
* **Path Parameters**:
  * `admission_id` (Mandatory, UUID)
* **Response (200 OK)**: Returns full details of checklist statuses and clinical, pharmacy, and billing gate clearances.

### GET `/nurses/admissions/{admission_id}/consumables`
* **Description**: Retrieve checklist summary of consumables billed to a patient for pre-discharge clearance verification.
* **Path Parameters**:
  * `admission_id` (Mandatory, UUID)
* **Response (200 OK)**: JSON list of billed consumables.

---

## 7. Bed & Ward Management

### PATCH `/nurses/beds`
* **Description**: Update the status/availability of a physical bed in the ward.
* **Request Body (JSON)**:
  * `bed_id` (Mandatory, UUID)
  * `to_status` (Mandatory, String): Options: `AVAILABLE`, `OCCUPIED`, `RESERVED`, `MAINTENANCE`.
  * `reason` (Optional, String, Max length 200)
* **Response (200 OK)**:
  ```json
  {
    "success": true,
    "code": 200,
    "data": {
      "bed_id": "b0a294d1-309a-434b-b29d-92a8fcb92cda",
      "bed_number": "GEN-05",
      "bed_type": "STANDARD",
      "status": "MAINTENANCE",
      "ward_id": "830023d4-7375-4e3a-9f12-d725a63d8f3a",
      "ward_name": "General Ward A",
      "changed_at": "2026-06-22T19:00:00Z"
    }
  }
  ```

### POST `/nurses/patients`
* **Description**: Transactional quick registration and bed allocation directly from the Nurse Portal (IPD admissions only).
* **Request Body (JSON)**:
  * `full_name` (Mandatory, String, Min length 2, Max 200)
  * `age` (Mandatory, Integer, Range: 0-130)
  * `gender` (Mandatory, String): `M`, `F`, `MALE`, `FEMALE`, or `OTHER`.
  * `contact_phone` (Mandatory, String, Min length 6, Max 20)
  * `diagnosis` (Optional, String, Max length 2000)
  * `registration_type` (Mandatory, String): Must be `IPD`.
  * `ward_id` (Mandatory, UUID)
  * `bed_id` (Mandatory, UUID)
  * `doctor_id` (Optional, UUID)
  * `diet` (Optional, String, Max length 100)
  * `admitted_by` (Mandatory, UUID)
* **Response (201 Created)**:
  ```json
  {
    "success": true,
    "code": 201,
    "data": {
      "patient_id": "p09da8b1-302a-434a-9fd2-92a8fbcd92c0",
      "admission_id": "a09da8b1-302a-434a-9fd2-92a8fbcd92c0",
      "uid": "0286",
      "bed": "GEN-06"
    }
  }
  ```
