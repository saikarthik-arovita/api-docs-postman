# HMS — Patient History & Timeline API Specification

This document provides the complete API specification for Patient History, Timeline, Summary, and Admission records, including exact request URLs, query parameters, header requirements, and response schemas with explicit pagination metadata (`has_next_page`, `has_prev_page`).

---

## Global Requirements

### Base URL
```
https://<api-base-url>/dev
```

### Universal Headers
| Header | Value | Description |
| :--- | :--- | :--- |
| `Authorization` | `Bearer <access_token>` | **Mandatory** JWT Access Token |
| `Content-Type` | `application/json` | **Mandatory** Request payload format |
| `x-tenant-id` | `<branch_uuid>` | Optional context header for branch scoping |

---

## 1. Patient OPD Visit History

Retrieves the paginated list of Outpatient Department (OPD) visit records for a patient.

* **Method**: `GET`
* **Endpoint**: `/patients/{patient_id}/opd/history`
* **Required Permission**: `opd:view`
* **Query Parameters**:
  | Parameter | Type | Required? | Default | Description |
  | :--- | :--- | :--- | :--- | :--- |
  | `page` | Integer | Optional | `1` | Page number to retrieve |
  | `page_size` | Integer | Optional | `20` | Records per page (Max 100) |

### Sample Response (`200 OK`)
```json
{
    "success": true,
    "code": 200,
    "data": {
        "patient_id": "8b5d7493-e9e9-43e5-99a4-e0467993911d",
        "total_visits": 16,
        "last_visit_date": "2026-08-08",
        "visits": [
            {
                "id": "d108b40e-ea7e-43c8-9b0d-e74f2223898a",
                "tenant_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
                "patient_id": "8b5d7493-e9e9-43e5-99a4-e0467993911d",
                "patient_name": "arpita arovita",
                "appointment_id": "4142c103-a1e4-4da2-a36e-e4c919263f32",
                "visit_number": "OPD-20260808-0005",
                "visit_date": "2026-08-08",
                "department_id": "61817d5e-49af-4b0c-b108-4691f450f9ba",
                "doctor_id": "1d5a7ebe-fe54-4f30-94d3-699809c45670",
                "chief_complaint": null,
                "follow_up_date": null,
                "follow_up_notes": null,
                "status": "COMPLETED",
                "symptoms": null,
                "registered_by": "5d9fcd72-1496-4eae-b362-aada88997ceb",
                "created_at": "2026-08-08 14:39:05.236671+05:30",
                "updated_at": "2026-08-08 14:43:42.641792+05:30",
                "consultation_id": "4120e704-afeb-4d92-b4a3-f1cc9aeaad9a",
                "decision": "MEDICAL",
                "diagnosis": "Consultation completed",
                "admission_id": "4b88744c-0024-4035-b579-f0e65b113a46",
                "admission_number": "IP-2607-2607",
                "admit_ward_id": "f1c00f90-ec55-497b-b9ef-e07b3c417214",
                "admit_bed_id": "859c4316-ee95-4674-b555-5da98729b637",
                "admission_type": "ELECTIVE",
                "admit_reason": "Acute Myocardial Infarction - Inpatient Monitoring",
                "ipd_status": "UNDER_TREATMENT"
            }
        ],
        "page": 1,
        "page_size": 20,
        "total_pages": 1,
        "has_next_page": false,
        "has_prev_page": false
    }
}
```

---

## 2. Patient Clinical Summary

Retrieves the patient profile summary including demographics, last attending doctor, next scheduled appointment, active insurance, active medical history, active medications, and pending lab orders.

* **Method**: `GET`
* **Endpoint**: `/patients/{patient_id}/summary`
* **Required Permission**: `patients:view`
* **Query Parameters**: *None*

### Sample Response (`200 OK`)
```json
{
    "success": true,
    "code": 200,
    "data": {
        "patient": {
            "id": "8b5d7493-e9e9-43e5-99a4-e0467993911d",
            "uhid": "PAT-2026-0106",
            "patient_number": "PAT-2026-0106",
            "full_name": "arpita arovita",
            "phone": "9867866997",
            "dob": "1993-08-13",
            "age": 24,
            "gender": "MALE",
            "blood_group": "B+",
            "address": null,
            "abha_number": null,
            "abha_address": "678876678876",
            "is_temporary": false,
            "is_active": true,
            "created_at": "2026-07-04 15:22:59.887694+05:30",
            "updated_at": "2026-08-07 18:36:59.742013+05:30",
            "patient_type": "OPD",
            "current_status": null,
            "insurance": null,
            "emergency_contact": null,
            "last_visit": null,
            "assigned_doctor": null,
            "last_vitals": null,
            "next_appointment": null,
            "pending_labs_count": 19,
            "active_medications_count": 9
        },
        "patient_type": "IPD",
        "attending_doctor": {
            "doctor_id": "1d5a7ebe-fe54-4f30-94d3-699809c45670",
            "doctor_name": "Dr. kartik",
            "department_id": "61817d5e-49af-4b0c-b108-4691f450f9ba",
            "department_name": "Cardiology",
            "consultation_date": "2026-08-08 14:39:30.474240+05:30"
        },
        "recent_vitals": [
            {
                "id": "c43f4b31-46e8-4916-8115-3edbbe9749c1",
                "patient_id": "8b5d7493-e9e9-43e5-99a4-e0467993911d",
                "bp_systolic": 102,
                "bp_diastolic": 80,
                "pulse": 90,
                "weight": null,
                "temperature": 33.0,
                "spo2": 98,
                "respiratory_rate": 18,
                "blood_sugar": 77,
                "pain_score": 0,
                "notes": null,
                "recorded_by": "1d5a7ebe-fe54-4f30-94d3-699809c45670",
                "recorded_at": "2026-08-08 14:40:05.154000+05:30"
            }
        ],
        "active_insurance": [
            {
                "id": "7d724b13-4a64-4908-b797-acf40809d788",
                "patient_id": "8b5d7493-e9e9-43e5-99a4-e0467993911d",
                "provider_name": "Star Health Insurance",
                "policy_number": "STAR-912",
                "scheme_type": "TPA",
                "is_active": true,
                "valid_from": "2026-01-01",
                "valid_till": "2026-12-31",
                "notes": "Active Star Health Policy STAR-912"
            }
        ],
        "active_history": [
            {
                "id": "11e2b05a-7bd0-475f-9ba9-f9e1461bfc35",
                "patient_id": "8b5d7493-e9e9-43e5-99a4-e0467993911d",
                "history_type": "ALLERGY",
                "condition": "peanut",
                "details": {
                    "notes": "done",
                    "reaction": ["hives"],
                    "severity": "MILD"
                },
                "is_active": true,
                "is_chronic": false,
                "severity": "MILD"
            }
        ],
        "medications": [
            {
                "id": "e1e4c3c4-1028-487d-8e90-327107afc0d6",
                "prescription_id": "9121908b-0354-466b-bade-fad49097cab6",
                "medicine_name": "Paracetamol 500mg",
                "dosage": "500mg",
                "frequency": "TDS",
                "duration": "10",
                "instructions": "",
                "status": "active",
                "created_at": "2026-08-08 14:43:15.636262+05:30"
            }
        ],
        "next_appointment": null,
        "pending_labs_count": 19,
        "pending_labs": [
            {
                "id": "f4d5b09b-8aa4-4a14-b34c-f8ae83524edc",
                "order_type": "LAB",
                "status": "PENDING",
                "test_name": "AFP (Alpha-Fetoprotein)",
                "created_at": "2026-07-29 14:49:32.782738+05:30"
            }
        ]
    }
}
```

---

## 3. Patient Activity Timeline

Retrieves a paginated event stream combining registration, visits, vitals, admissions, prescriptions, and diagnostic orders sorted by timestamp descending.

* **Method**: `GET`
* **Endpoint**: `/patients/{patient_id}/timeline`
* **Required Permission**: `patients:view`
* **Query Parameters**:
  | Parameter | Type | Required? | Default | Description |
  | :--- | :--- | :--- | :--- | :--- |
  | `page` | Integer | Optional | `1` | Page number |
  | `page_size` | Integer | Optional | `20` | Items per page (Max 100) |

### Sample Response (`200 OK`)
```json
{
    "success": true,
    "code": 200,
    "data": {
        "items": [
            {
                "event_id": "d108b40e-ea7e-43c8-9b0d-e74f2223898a",
                "event_type": "OPD_VISIT_COMPLETED",
                "title": "OPD Visit Completed",
                "description": "Consultation with Dr. kartik. Diagnosis: Consultation completed",
                "timestamp": "2026-08-08T14:43:42.641792+05:30",
                "recorded_by": "1d5a7ebe-fe54-4f30-94d3-699809c45670",
                "details": {
                    "visit_number": "OPD-20260808-0005",
                    "doctor_name": "Dr. kartik",
                    "department_name": "Cardiology",
                    "diagnosis": "Consultation completed",
                    "status": "COMPLETED"
                }
            },
            {
                "event_id": "8b5d7493-e9e9-43e5-99a4-e0467993911d",
                "event_type": "REGISTRATION",
                "title": "Patient Registered",
                "description": "Demographic details registered. UHID: PAT-2026-0106",
                "timestamp": "2026-07-04T15:22:59.887694+05:30",
                "recorded_by": null,
                "details": {
                    "uhid": "PAT-2026-0106"
                }
            }
        ],
        "total": 24,
        "page": 1,
        "page_size": 20,
        "total_pages": 2,
        "has_next_page": true,
        "has_prev_page": false
    }
}
```

---

## 4. Patient IPD Admissions History

Retrieves Inpatient Department (IPD) hospital stay history, ward & bed details, attending doctor, and bed transfer logs for a patient.

* **Method**: `GET`
* **Endpoint**: `/patients/{patient_id}/admissions`
* **Required Permission**: `patients:view`
* **Query Parameters**: *None*

### Sample Response (`200 OK`)
```json
{
    "success": true,
    "code": 200,
    "data": [
        {
            "id": "4b88744c-0024-4035-b579-f0e65b113a46",
            "admission_number": "IP-2607-2607",
            "status": "ADMITTED",
            "ipd_status": "UNDER_TREATMENT",
            "admission_reason": "Acute Myocardial Infarction - Inpatient Monitoring",
            "admitted_at": "2026-07-31 12:23:16.982626+05:30",
            "actual_discharge_at": null,
            "ward_name": "Premium Private Ward",
            "bed_number": "B-PVT-05",
            "doctor_name": "Kiran Patil",
            "bed_transfers": []
        },
        {
            "id": "528b0284-1cb3-4042-a292-2548ea7de17b",
            "admission_number": "IP-2607-2607",
            "status": "DISCHARGED",
            "ipd_status": "DISCHARGED",
            "admission_reason": "Consultation completed",
            "admitted_at": "2026-07-04 15:50:50.933612+05:30",
            "actual_discharge_at": null,
            "ward_name": "Emergency Ward D",
            "bed_number": "B-ERD-08",
            "doctor_name": "Dr. Vikram Shah",
            "bed_transfers": [
                {
                    "id": "e1bfd8a6-5563-4c94-b0d4-0ee61afb6878",
                    "transfer_type": "ICU",
                    "transfer_reason": "done",
                    "notes": null,
                    "transferred_at": "2026-07-04 15:54:43.202470+05:30",
                    "from_ward_name": "General Medical Ward",
                    "from_bed_number": "B-GEN-14",
                    "to_ward_name": "Super Intensive Care Unit (SICU)",
                    "to_bed_number": "B-ICU-12",
                    "transferred_by_name": "Sarah Jenkins"
                }
            ]
        }
    ]
}
```
