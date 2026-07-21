# Lab Order Lifecycle & Addendum Workflow — E2E API Testing Reference Guide

This reference guide documents the actual, live API request payloads and JSON responses captured during the complete E2E execution of a lab order registration through to results entry, pathologist approval, report generation, and the subsequent post-release clinical addendum/amendment lifecycle.

---

## Authentication Roles & Test Credentials

Use these credentials to authenticate via `POST /auth/login` to obtain access tokens for each stage:

| Role | Username / Email | Password | Primary Lifecycle Action |
| :--- | :--- | :--- | :--- |
| **RECEPTIONIST** | `sunitha.nair.b1@hospital.com` | `Temp@123` | Create Registration |
| **LAB_ADMIN** | `amit.verma@arovita.com` | `Test@123` | Accept, Collect, Confirm, Process, Submit Results |
| **PATHOLOGIST** | `pathologist@arovita.com` | `Test@123` | Approval, Generate Report, Addendum Actions |

---

## Step-by-Step API Request & Response Log

### 1. Create Registration (PENDING Stage)
* **Role**: `RECEPTIONIST`
* **Method**: `POST`
* **Path**: `/diagnostic-orders/orders`
* **Payload**:
```json
{
  "patient_id": "93643ddd-0d0d-491d-83d6-37f3bcde518d",
  "doctor_id": "b1da0859-f222-4a4a-83d6-785b779295fb",
  "department_id": "46d19155-09a1-462e-b256-ce2a5741b091",
  "priority": "Routine",
  "action": "REGISTER_AND_PAY",
  "selected_tests": [
    {
      "test_id": "e5e82d23-96f8-4981-b068-52a12dc10de4"
    }
  ],
  "remarks": "E2E Full Addendum Test 0f15ee5b"
}
```
* **Response (201 Created)**:
```json
{
  "success": true,
  "code": 201,
  "message": "Registration completed successfully",
  "data": {
    "id": "d13ee80d-1a58-4d5c-968e-1a0fc4573b83",
    "patient_id": "93643ddd-0d0d-491d-83d6-37f3bcde518d",
    "visit_type": "Walk-in",
    "doctor_id": "b1da0859-f222-4a4a-83d6-785b779295fb",
    "department_id": "46d19155-09a1-462e-b256-ce2a5741b091",
    "status": "REGISTERED",
    "total_amount": 367.5,
    "payment_status": "PAID"
  }
}
```

---

### 2. Accept Order (PENDING -> SCHEDULED)
* **Role**: `LAB_ADMIN`
* **Method**: `POST`
* **Path**: `/diagnostic-orders/orders/b3c677e1-93a7-4f07-b569-8194f5ec76f8/actions`
* **Payload**:
```json
{
  "action": "accept",
  "reason": "Accepted for processing"
}
```
* **Response (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "message": "Order accepted successfully",
  "data": {
    "id": "b3c677e1-93a7-4f07-b569-8194f5ec76f8",
    "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "diagnostic_order_id": "d13ee80d-1a58-4d5c-968e-1a0fc4573b83",
    "test_id": "e5e82d23-96f8-4981-b068-52a12dc10de4",
    "status": "SCHEDULED",
    "sample_collected_at": null,
    "sample_collected_by": null,
    "processed_at": null,
    "processed_by": null,
    "barcode": "BC-10080",
    "notes": "E2E Full Addendum Test 0f15ee5b",
    "created_at": "2026-07-20 14:27:23.786841+05:30",
    "updated_at": "2026-07-20 14:27:26.197929+05:30",
    "barcode_id": "b3c677e1-93a7-4f07-b569-8194f5ec76f8",
    "token_number": "LAB-1011",
    "registration_notes": null,
    "priority": "ROUTINE",
    "fasting_required": false,
    "sample_type": null,
    "draft_flag": false,
    "technician_id": null,
    "collector_id": null,
    "report_status": "PENDING",
    "assigned_analyzer": null,
    "received_at": null,
    "received_by": null,
    "machine_id": null
  }
}
```

---

### 3. Collect Sample (SCHEDULED -> SAMPLE_COLLECTED)
* **Role**: `LAB_ADMIN`
* **Method**: `POST`
* **Path**: `/diagnostic-orders/orders/b3c677e1-93a7-4f07-b569-8194f5ec76f8/actions`
* **Payload**:
```json
{
  "action": "collect-sample",
  "barcode": "BC-E2E-AB1124",
  "notes": "EDTA tube, left arm"
}
```
* **Response (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "message": "Sample collected successfully",
  "data": {
    "id": "b3c677e1-93a7-4f07-b569-8194f5ec76f8",
    "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "diagnostic_order_id": "d13ee80d-1a58-4d5c-968e-1a0fc4573b83",
    "test_id": "e5e82d23-96f8-4981-b068-52a12dc10de4",
    "status": "SAMPLE_COLLECTED",
    "sample_collected_at": "2026-07-20 14:27:27.601359+05:30",
    "sample_collected_by": "e35c4fff-e52b-4b2d-ac62-b567e7826e0c",
    "processed_at": null,
    "processed_by": null,
    "barcode": "BC-E2E-AB1124",
    "notes": "EDTA tube, left arm",
    "created_at": "2026-07-20 14:27:23.786841+05:30",
    "updated_at": "2026-07-20 14:27:27.601359+05:30",
    "barcode_id": "b3c677e1-93a7-4f07-b569-8194f5ec76f8",
    "token_number": "LAB-1011",
    "registration_notes": null,
    "priority": "ROUTINE",
    "fasting_required": false,
    "sample_type": null,
    "draft_flag": false,
    "technician_id": null,
    "collector_id": null,
    "report_status": "PENDING",
    "assigned_analyzer": null,
    "received_at": null,
    "received_by": null,
    "machine_id": null
  }
}
```

---

### 4. Confirm Sample Arrival (SAMPLE_COLLECTED -> SCHEDULED)
* **Role**: `LAB_ADMIN`
* **Method**: `PATCH`
* **Path**: `/diagnostic-orders/lab/samples/b3c677e1-93a7-4f07-b569-8194f5ec76f8/confirm-arrival`
* **Payload**: *None*
* **Response (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "message": "Sample arrival confirmed successfully",
  "data": {
    "id": "b3c677e1-93a7-4f07-b569-8194f5ec76f8",
    "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "diagnostic_order_id": "d13ee80d-1a58-4d5c-968e-1a0fc4573b83",
    "test_id": "e5e82d23-96f8-4981-b068-52a12dc10de4",
    "status": "SCHEDULED",
    "sample_collected_at": "2026-07-20 14:27:27.601359+05:30",
    "sample_collected_by": "e35c4fff-e52b-4b2d-ac62-b567e7826e0c",
    "processed_at": null,
    "processed_by": null,
    "barcode": "BC-E2E-AB1124",
    "notes": "EDTA tube, left arm",
    "created_at": "2026-07-20 14:27:23.786841+05:30",
    "updated_at": "2026-07-20 14:27:28.989988+05:30",
    "barcode_id": "b3c677e1-93a7-4f07-b569-8194f5ec76f8",
    "token_number": "LAB-1011",
    "registration_notes": null,
    "priority": "ROUTINE",
    "fasting_required": false,
    "sample_type": null,
    "draft_flag": false,
    "technician_id": null,
    "collector_id": null,
    "report_status": "PENDING",
    "assigned_analyzer": null,
    "received_at": "2026-07-20 14:27:28.989988+05:30",
    "received_by": "e35c4fff-e52b-4b2d-ac62-b567e7826e0c",
    "machine_id": null
  }
}
```

---

### 5. Start Processing (SCHEDULED -> IN_PROGRESS)
* **Role**: `LAB_ADMIN`
* **Method**: `POST`
* **Path**: `/diagnostic-orders/orders/b3c677e1-93a7-4f07-b569-8194f5ec76f8/actions`
* **Payload**:
```json
{
  "action": "process"
}
```
* **Response (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "message": "Sample processing started",
  "data": {
    "id": "b3c677e1-93a7-4f07-b569-8194f5ec76f8",
    "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "diagnostic_order_id": "d13ee80d-1a58-4d5c-968e-1a0fc4573b83",
    "test_id": "e5e82d23-96f8-4981-b068-52a12dc10de4",
    "status": "IN_PROGRESS",
    "sample_collected_at": "2026-07-20 14:27:27.601359+05:30",
    "sample_collected_by": "e35c4fff-e52b-4b2d-ac62-b567e7826e0c",
    "processed_at": "2026-07-20 14:27:30.435058+05:30",
    "processed_by": "e35c4fff-e52b-4b2d-ac62-b567e7826e0c",
    "barcode": "BC-E2E-AB1124",
    "notes": "EDTA tube, left arm",
    "created_at": "2026-07-20 14:27:23.786841+05:30",
    "updated_at": "2026-07-20 14:27:30.435058+05:30",
    "barcode_id": "b3c677e1-93a7-4f07-b569-8194f5ec76f8",
    "token_number": "LAB-1011",
    "registration_notes": null,
    "priority": "ROUTINE",
    "fasting_required": false,
    "sample_type": null,
    "draft_flag": false,
    "technician_id": null,
    "collector_id": null,
    "report_status": "PENDING",
    "assigned_analyzer": null,
    "received_at": "2026-07-20 14:27:28.989988+05:30",
    "received_by": "e35c4fff-e52b-4b2d-ac62-b567e7826e0c",
    "machine_id": null
  }
}
```

---

### 6. Submit Result Values (IN_PROGRESS -> COMPLETED)
* **Role**: `LAB_ADMIN`
* **Method**: `POST`
* **Path**: `/diagnostic-orders/orders/b3c677e1-93a7-4f07-b569-8194f5ec76f8/actions`
* **Payload**:
```json
{
  "action": "submit-values",
  "result_values": {
    "param-uuid-1111": "14.2"
  },
  "remarks": "Hemoglobin typed"
}
```
* **Response (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "message": "Results submitted successfully",
  "data": {
    "result_header": {
      "id": "7fc70e45-7936-45e1-99b2-44a106c5aeb6",
      "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
      "lab_order_item_id": "b3c677e1-93a7-4f07-b569-8194f5ec76f8",
      "result_value": "Completed",
      "result_json": null,
      "report_url": null,
      "abnormal_flag": false,
      "critical_flag": false,
      "entered_by": "e35c4fff-e52b-4b2d-ac62-b567e7826e0c",
      "validated_by": null,
      "validated_at": null,
      "comments": "Hemoglobin typed",
      "created_at": "2026-07-20 14:27:31.842666+05:30",
      "updated_at": "2026-07-20 14:27:31.842666+05:30",
      "status": "READY_FOR_REVIEW",
      "digital_signature": false
    },
    "values": [
      {
        "id": "53ca9119-5ed6-4a6e-bf7f-f03f21c9caba",
        "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
        "lab_order_item_id": "b3c677e1-93a7-4f07-b569-8194f5ec76f8",
        "parameter_id": "11111111-1111-1111-1111-111111111111",
        "result_value": "14.2",
        "abnormal_flag": false,
        "critical_flag": false,
        "created_at": "2026-07-20 14:27:31.842666+05:30"
      }
    ]
  }
}
```

---

### 7. Pathologist Approval (READY_FOR_REVIEW -> APPROVED)
* **Role**: `PATHOLOGIST`
* **Method**: `PATCH`
* **Path**: `/diagnostic-orders/lab/results/7fc70e45-7936-45e1-99b2-44a106c5aeb6/status`
* **Payload**:
```json
{
  "status": "APPROVED",
  "digitalSignature": true
}
```
* **Response (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "message": "Result status updated to APPROVED successfully",
  "data": {
    "id": "7fc70e45-7936-45e1-99b2-44a106c5aeb6",
    "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "lab_order_item_id": "b3c677e1-93a7-4f07-b569-8194f5ec76f8",
    "result_value": "Completed",
    "result_json": null,
    "report_url": null,
    "abnormal_flag": false,
    "critical_flag": false,
    "entered_by": "e35c4fff-e52b-4b2d-ac62-b567e7826e0c",
    "validated_by": "143b85f9-0073-4b51-8357-bf5ccfcffcea",
    "validated_at": "2026-07-20 14:27:33.915486+05:30",
    "comments": "Hemoglobin typed\nPathologist Remarks (APPROVED): No comments",
    "created_at": "2026-07-20 14:27:31.842666+05:30",
    "updated_at": "2026-07-20 14:27:33.915486+05:30",
    "status": "APPROVED",
    "digital_signature": true
  }
}
```

---

### 8. Generate Report (APPROVED -> REPORT_GENERATED)
* **Role**: `PATHOLOGIST`
* **Method**: `PATCH`
* **Path**: `/diagnostic-orders/lab/results/7fc70e45-7936-45e1-99b2-44a106c5aeb6/status`
* **Payload**:
```json
{
  "status": "REPORT_GENERATED"
}
```
* **Response (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "message": "Result status updated to REPORT_GENERATED successfully",
  "data": {
    "id": "7fc70e45-7936-45e1-99b2-44a106c5aeb6",
    "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "lab_order_item_id": "b3c677e1-93a7-4f07-b569-8194f5ec76f8",
    "result_value": "Completed",
    "result_json": null,
    "report_url": null,
    "abnormal_flag": false,
    "critical_flag": false,
    "entered_by": "e35c4fff-e52b-4b2d-ac62-b567e7826e0c",
    "validated_by": "143b85f9-0073-4b51-8357-bf5ccfcffcea",
    "validated_at": "2026-07-20 14:27:33.915486+05:30",
    "comments": "Hemoglobin typed\nPathologist Remarks (APPROVED): No comments\nPathologist Remarks (REPORT_GENERATED): No comments",
    "created_at": "2026-07-20 14:27:31.842666+05:30",
    "updated_at": "2026-07-20 14:27:35.480019+05:30",
    "status": "REPORT_GENERATED",
    "digital_signature": false
  }
}
```

---

### 10. Create Addendum
* **Role**: `PATHOLOGIST`
* **Method**: `POST`
* **Path**: `/diagnostic-orders/lab/addendums`
* **Payload**:
```json
{
  "lab_result_id": "7fc70e45-7936-45e1-99b2-44a106c5aeb6",
  "patient_name": "E2E Simulation Patient",
  "patient_uhid": "PAT-2026-0001",
  "test_type": "Hematology",
  "amendment_type": "Correction",
  "original_report_date": "2026-07-20T12:00:00Z",
  "requested_by": "143b85f9-0073-4b51-8357-bf5ccfcffcea",
  "updated_by": "143b85f9-0073-4b51-8357-bf5ccfcffcea",
  "reason": "Correcting typo in hemoglobin value",
  "clinical_notes": "Technician input 14.2 g/dL but standard reading confirms 12.4 g/dL. Corrected here.",
  "status": "Pending"
}
```
* **Response (201 Created)**:
```json
{
  "success": true,
  "code": 201,
  "message": "Addendum report created successfully",
  "data": {
    "id": "bdd49d86-63a9-4d87-9c97-e2b43b30c225",
    "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "lab_result_id": "7fc70e45-7936-45e1-99b2-44a106c5aeb6",
    "patient_name": "E2E Simulation Patient",
    "patient_uhid": "PAT-2026-0001",
    "test_type": "Hematology",
    "amendment_type": "Correction",
    "original_report_date": "2026-07-20 17:30:00+05:30",
    "addendum_date": "2026-07-20 14:27:37.375859+05:30",
    "requested_by": "143b85f9-0073-4b51-8357-bf5ccfcffcea",
    "updated_by": "143b85f9-0073-4b51-8357-bf5ccfcffcea",
    "reason": "Correcting typo in hemoglobin value",
    "clinical_notes": "Technician input 14.2 g/dL but standard reading confirms 12.4 g/dL. Corrected here.",
    "status": "Pending",
    "created_at": "2026-07-20 14:27:37.375859+05:30",
    "updated_at": "2026-07-20 14:27:37.375859+05:30"
  }
}
```

---

### 11. Get Addendum by ID
* **Role**: `PATHOLOGIST`
* **Method**: `GET`
* **Path**: `/diagnostic-orders/lab/addendums/bdd49d86-63a9-4d87-9c97-e2b43b30c225`
* **Payload**: *None*
* **Response (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "bdd49d86-63a9-4d87-9c97-e2b43b30c225",
    "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "lab_result_id": "7fc70e45-7936-45e1-99b2-44a106c5aeb6",
    "patient_name": "E2E Simulation Patient",
    "patient_uhid": "PAT-2026-0001",
    "test_type": "Hematology",
    "amendment_type": "Correction",
    "original_report_date": "2026-07-20 17:30:00+05:30",
    "addendum_date": "2026-07-20 14:27:37.375859+05:30",
    "requested_by": "143b85f9-0073-4b51-8357-bf5ccfcffcea",
    "updated_by": "143b85f9-0073-4b51-8357-bf5ccfcffcea",
    "reason": "Correcting typo in hemoglobin value",
    "clinical_notes": "Technician input 14.2 g/dL but standard reading confirms 12.4 g/dL. Corrected here.",
    "status": "Pending",
    "created_at": "2026-07-20 14:27:37.375859+05:30",
    "updated_at": "2026-07-20 14:27:37.375859+05:30"
  }
}
```

---

### 12. Approve Addendum (Pending -> Approved)
* **Role**: `PATHOLOGIST`
* **Method**: `PATCH`
* **Path**: `/diagnostic-orders/lab/addendums/bdd49d86-63a9-4d87-9c97-e2b43b30c225`
* **Payload**:
```json
{
  "status": "Approved",
  "clinical_notes": "Addendum finalized and approved by pathologist.",
  "updated_by": "143b85f9-0073-4b51-8357-bf5ccfcffcea"
}
```
* **Response (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "message": "Addendum report updated successfully",
  "data": {
    "id": "bdd49d86-63a9-4d87-9c97-e2b43b30c225",
    "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "lab_result_id": "7fc70e45-7936-45e1-99b2-44a106c5aeb6",
    "patient_name": "E2E Simulation Patient",
    "patient_uhid": "PAT-2026-0001",
    "test_type": "Hematology",
    "amendment_type": "Correction",
    "original_report_date": "2026-07-20 17:30:00+05:30",
    "addendum_date": "2026-07-20 14:27:37.375859+05:30",
    "requested_by": "143b85f9-0073-4b51-8357-bf5ccfcffcea",
    "updated_by": "143b85f9-0073-4b51-8357-bf5ccfcffcea",
    "reason": "Correcting typo in hemoglobin value",
    "clinical_notes": "Addendum finalized and approved by pathologist.",
    "status": "Approved",
    "created_at": "2026-07-20 14:27:37.375859+05:30",
    "updated_at": "2026-07-20 14:27:40.441741+05:30"
  }
}
```

---

### 13. List Addendums
* **Role**: `PATHOLOGIST`
* **Method**: `GET`
* **Path**: `/diagnostic-orders/lab/addendums`
* **Payload**: *None*
* **Response (200 OK)**:
```json
{
  "success": true,
  "code": 200,
  "data": {
    "total": 3,
    "page": 1,
    "limit": 10,
    "items": [
      {
        "addendum_id": "bdd49d86-63a9-4d87-9c97-e2b43b30c225",
        "lab_result_id": "7fc70e45-7936-45e1-99b2-44a106c5aeb6",
        "patient_name": "E2E Simulation Patient",
        "patient_uhid": "PAT-2026-0001",
        "test_type": "Hematology",
        "amendment_type": "Correction",
        "original_report_date": "2026-07-20 17:30:00+05:30",
        "addendum_date": "2026-07-20 14:27:37.375859+05:30",
        "requested_by": "143b85f9-0073-4b51-8357-bf5ccfcffcea",
        "updated_by": "143b85f9-0073-4b51-8357-bf5ccfcffcea",
        "reason": "Correcting typo in hemoglobin value",
        "clinical_notes": "Addendum finalized and approved by pathologist.",
        "status": "Approved",
        "created_at": "2026-07-20 14:27:37.375859+05:30",
        "updated_at": "2026-07-20 14:27:40.441741+05:30"
      }
    ]
  }
}
```
