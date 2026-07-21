# HMS Lab Module — Workflow & API Integration Guide

This guide details the laboratory operations, sample management, test result input, and validation workflow in the HMS **Lab Module**. It outlines how states transition in the database and provides a step-by-step reference for integrating and executing the technician and pathologist APIs with actual request and response payloads.

---

## 1. Workflow & Status Engine Lifecycle

Lab order items go through a strict state machine from the moment they are ordered until final patient delivery.

```mermaid
stateflow
    [*] --> PENDING : Lab Order Created (NEW)
    PENDING --> SAMPLE_COLLECTED : Draw Specimen (IN_TRANSIT)
    SAMPLE_COLLECTED --> SCHEDULED : Confirm Sample Arrival (AWAITING_PROCESSING)
    SCHEDULED --> IN_PROGRESS : Register Specimen in Analyzer (ON_ANALYZER)
    IN_PROGRESS --> COMPLETED : Submit Result Values (READY_FOR_REVIEW)
    COMPLETED --> APPROVED : Pathologist Signs Off (APPROVED)
    COMPLETED --> REPROCESSING : Returned for Correction (REPROCESSING)
    REPROCESSING --> COMPLETED : Re-submit Corrected Values (READY_FOR_REVIEW)
    APPROVED --> REPORT_GENERATED : Generate PDF Report
    REPORT_GENERATED --> REPORT_DELIVERED : Deliver Final Report
    PENDING --> CANCELLED : Cancel Order (REJECTED)
```

### Database vs. Domain Status Mapping
The backend uses a standard status lifecycle mapping to translate physical database statuses (`lab_order_items.status`) to user-facing dashboard statuses:

| Database Status | Domain/Dashboard Status | Description |
| :--- | :--- | :--- |
| `PENDING` | `NEW` or `REPEAT_SAMPLE_REQUIRED` | Created, waiting for sample collection (If notes start with `"REPEAT:"`, maps to `REPEAT_SAMPLE_REQUIRED`). |
| `SAMPLE_COLLECTED` | `IN_TRANSIT` | Sample collected from patient; container barcode assigned. |
| `SCHEDULED` | `AWAITING_PROCESSING` | Sample arrived at the lab; confirmed by technician. |
| `IN_PROGRESS` | `ON_ANALYZER` | Sample is loaded/running on the diagnostics analyzer. |
| `COMPLETED` | `READY_FOR_REVIEW` | Technician has entered results; awaiting pathologist verification. |
| `APPROVED` | `APPROVED` | Clinician/Pathologist signed off on the results. |
| `REPORT_GENERATED` | `REPORT_GENERATED` | Final PDF report generated with pathologist's digital signature. |
| `REPORT_DELIVERED` | `REPORT_DELIVERED` | Patient/Doctor has received the final report. |
| `CANCELLED` | `REJECTED` | Test cancelled. |

---

## 2. Step-by-Step API Execution Reference

Follow this sequence of HTTP requests to execute the laboratory flow end-to-end.

---

### Step 1: Create Lab Order
Creates a new diagnostic order containing one or more laboratory items.

* **Method:** `POST`
* **URL:** `/diagnostic-orders`
* **Headers:**
  * `Authorization`: `Bearer <token>`
  * `Content-Type`: `application/json`
* **Request Body:**
  | Field | Type | Mandatory? | Description |
  | :--- | :--- | :--- | :--- |
  | `patient_id` | UUID string | **Yes** | The patient identifier |
  | `doctor_id` | UUID string | **Yes** | The ordering physician |
  | `order_type` | String | **Yes** | Must be `'LAB'` |
  | `priority` | String | **Yes** | `'ROUTINE'` or `'EMERGENCY'` |
  | `clinical_notes`| String | No | Notes from the doctor |
  | `context_type` | String | **Yes** | Must be `'LAB'` or a valid DB enum |
  | `context_id` | UUID string | No | Consultation or admission identifier |
  | `encounter_type`| String | **Yes** | e.g. `'WALKIN'` or `'CONSULTATION'` |
  | `source_module` | String | **Yes** | Must be `'LAB'` |
  | `lab_items` | Array of Dicts | **Yes** | List of tests to order |
  | `lab_items[].test_id` | UUID string | **Yes** | The lab test catalogue ID |
  | `lab_items[].notes` | String | No | Instructions per item |

* **Request Example:**
  ```json
  {
    "patient_id": "93643ddd-0d0d-491d-83d6-37f3bcde518d",
    "doctor_id": "cab7cc4e-3ec7-49d9-a393-bbdb93bf58c2",
    "order_type": "LAB",
    "priority": "ROUTINE",
    "clinical_notes": "Draft registration details",
    "context_type": "LAB",
    "context_id": null,
    "source_module": "LAB",
    "encounter_type": "WALKIN",
    "lab_items": [
      {
        "test_id": "e5e82d23-96f8-4981-b068-52a12dc10de4",
        "notes": "Fast track blood culture"
      }
    ]
  }
  ```
* **Success Response (`201 Created`):**
  ```json
  {
    "success": true,
    "code": 200,
    "data": {
      "id": "ddde2263-6386-4358-8271-a85c4a6dc3f3",
      "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
      "patient_id": "93643ddd-0d0d-491d-83d6-37f3bcde518d",
      "doctor_id": "cab7cc4e-3ec7-49d9-a393-bbdb93bf58c2",
      "department_id": "46d19155-09a1-462e-b256-ce2a5741b091",
      "order_type": "LAB",
      "status": "PENDING",
      "priority": "ROUTINE",
      "clinical_notes": "Draft registration details",
      "created_at": "2026-07-13 12:54:21.591687+05:30",
      "updated_at": "2026-07-13 12:54:21.591687+05:30",
      "context_type": "LAB",
      "context_id": null,
      "source_module": "LAB",
      "encounter_type": "WALKIN",
      "created_by": "5221a442-e75f-49d5-aa42-860dc76a76de",
      "version_no": 1,
      "ipd_admission_id": null,
      "clinical_escalation_id": null,
      "lab_items": [
        {
          "id": "799ec1df-b8a5-4e01-a271-86d58a66fa8d",
          "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
          "diagnostic_order_id": "ddde2263-6386-4358-8271-a85c4a6dc3f3",
          "test_id": "e5e82d23-96f8-4981-b068-52a12dc10de4",
          "status": "PENDING",
          "barcode": null,
          "notes": "Fast track blood culture",
          "created_at": "2026-07-13 12:54:21.611124+05:30",
          "updated_at": "2026-07-13 12:54:21.611124+05:30"
        }
      ]
    },
    "message": "Order created successfully"
  }
  ```

---

### Step 2: Collect Specimen Sample
Assigns the container barcode and transitions the lab item status from `PENDING` to `SAMPLE_COLLECTED` (`IN_TRANSIT` in domain).

* **Method:** `PATCH`
* **URL:** `/diagnostic-orders/lab/{orderItemId}/collect-sample`
* **Headers:**
  * `Authorization`: `Bearer <token>`
  * `Content-Type`: `application/json`
* **Request Body:**
  | Field | Type | Mandatory? | Description |
  | :--- | :--- | :--- | :--- |
  | `barcode` | String | **Yes** | Unique specimen identifier/container label |
  | `notes` | String | No | Collection remarks |

* **Request Example:**
  ```json
  {
    "barcode": "BC-10017",
    "notes": "Specimen collected from left forearm"
  }
  ```
* **Success Response (`200 OK`):**
  ```json
  {
    "success": true,
    "code": 200,
    "data": {
      "id": "799ec1df-b8a5-4e01-a271-86d58a66fa8d",
      "diagnostic_order_id": "ddde2263-6386-4358-8271-a85c4a6dc3f3",
      "test_id": "e5e82d23-96f8-4981-b068-52a12dc10de4",
      "status": "SAMPLE_COLLECTED",
      "barcode": "BC-10017",
      "notes": "Specimen collected from left forearm",
      "updated_at": "2026-07-13 12:55:04.148102+05:30"
    },
    "message": "Sample collected successfully"
  }
  ```

---

### Step 3: Confirm Sample Arrival (Technician)
Confirms that the specimen container has arrived at the laboratory processing bench. Updates status to `SCHEDULED` (`AWAITING_PROCESSING` in domain).

* **Method:** `PATCH`
* **URL:** `/diagnostic-orders/lab/samples/{orderItemId}/confirm-arrival`
* **Headers:**
  * `Authorization`: `Bearer <token>`
* **Success Response (`200 OK`):**
  ```json
  {
    "success": true,
    "code": 200,
    "data": {
      "id": "799ec1df-b8a5-4e01-a271-86d58a66fa8d",
      "status": "SCHEDULED",
      "received_at": "2026-07-13 12:55:12.441990+05:30",
      "received_by": "5221a442-e75f-49d5-aa42-860dc76a76de"
    },
    "message": "Sample arrival confirmed successfully"
  }
  ```

---

### Step 4: Save Result Draft (Optional)
Saves intermediate draft result values to the database. These drafts are mutable and can be saved multiple times.

* **Method:** `POST`
* **URL:** `/diagnostic-orders/lab/results/{orderItemId}/draft`
* **Headers:**
  * `Authorization`: `Bearer <token>`
  * `Content-Type`: `application/json`
* **Request Body:**
  | Field | Type | Mandatory? | Description |
  | :--- | :--- | :--- | :--- |
  | `draft_values` | JSON Object | **Yes** | Key-value pairs of `{parameter_code: value}` |
  | `remarks` | String | No | Technician remarks |

* **Request Example:**
  ```json
  {
    "draft_values": {
      "param-uuid-1111": "11.2"
    },
    "remarks": "Slight turbidity noted"
  }
  ```
* **Success Response (`200 OK`):**
  ```json
  {
    "success": true,
    "code": 200,
    "data": {
      "id": "5fa2bb9d-ea43-4412-a088-294b6de81ee0",
      "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
      "lab_order_item_id": "799ec1df-b8a5-4e01-a271-86d58a66fa8d",
      "draft_values": {
        "param-uuid-1111": "11.2"
      },
      "remarks": "Slight turbidity noted",
      "updated_at": "2026-07-13 12:56:04.148102+05:30"
    },
    "message": "Draft saved successfully"
  }
  ```

---

### Step 5: Flag Critical Values (Optional)
Flags specific parameter values as critical. This logs the alerts to trigger flags on the Pathologist's verification dashboard.

* **Method:** `PATCH`
* **URL:** `/diagnostic-orders/lab/results/{orderItemId}/critical`
* **Headers:**
  * `Authorization`: `Bearer <token>`
  * `Content-Type`: `application/json`
* **Request Body:**
  | Field | Type | Mandatory? | Description |
  | :--- | :--- | :--- | :--- |
  | `parameter_ids` | Array of UUIDs | **Yes** | List of parameter IDs flagged as critical |
  | `reason` | String | **Yes** | Reasons/remarks for critical flagging |
  | `pathologist_id`| UUID string | **Yes** | Pathologist user ID to notify |

* **Request Example:**
  ```json
  {
    "parameter_ids": ["11111111-1111-1111-1111-111111111111"],
    "reason": "Hemoglobin critically low",
    "pathologist_id": "143b85f9-0073-4b51-8357-bf5ccfcffcea"
  }
  ```
* **Success Response (`200 OK`):**
  ```json
  {
    "success": true,
    "code": 200,
    "data": {
      "id": "e3e8cc14-f421-4991-b924-f725357acffb",
      "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
      "lab_order_item_id": "799ec1df-b8a5-4e01-a271-86d58a66fa8d",
      "parameter_ids": ["11111111-1111-1111-1111-111111111111"],
      "reason": "Hemoglobin critically low",
      "marked_by": "5221a442-e75f-49d5-aa42-860dc76a76de",
      "notified_pathologist_id": "143b85f9-0073-4b51-8357-bf5ccfcffcea",
      "notified_at": "2026-07-13 12:56:44.112990+05:30"
    },
    "message": "Results marked as critical and pathologist notified"
  }
  ```

---

### Step 6: Send Result to Pathologist Verification Queue
Submits current draft values as final and promotes the lab item status to `COMPLETED` (`READY_FOR_REVIEW` in domain).

* **Method:** `POST`
* **URL:** `/diagnostic-orders/lab/result-submission/results/{orderItemId}/send`
* **Headers:**
  * `Authorization`: `Bearer <token>`
* **Request Body:** `{}` (Empty Object)
* **Success Response (`200 OK`):**
  ```json
  {
    "success": true,
    "code": 200,
    "data": {
      "id": "717701a9-d1e2-41c6-917b-7d7e2e123be9",
      "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
      "lab_order_item_id": "799ec1df-b8a5-4e01-a271-86d58a66fa8d",
      "result_value": "Completed",
      "result_json": null,
      "report_url": null,
      "abnormal_flag": false,
      "critical_flag": false,
      "entered_by": "5221a442-e75f-49d5-aa42-860dc76a76de",
      "status": "READY_FOR_REVIEW",
      "digital_signature": false,
      "comments": "Slight turbidity noted",
      "created_at": "2026-07-13T09:30:35.208475Z",
      "updated_at": "2026-07-13T09:30:35.208475Z"
    },
    "message": "Result sent to pathologist successfully"
  }
  ```

---

### Step 7: Get Report Details for Pathologist Review
Allows the Pathologist to fetch complete details of the result including test parameters, patient metadata, and status audit logs.

* **Method:** `GET`
* **URL:** `/diagnostic-orders/lab/results/{resultId}`
* **Headers:**
  * `Authorization`: `Bearer <token>`
* **Success Response (`200 OK`):**
  ```json
  {
    "success": true,
    "code": 200,
    "data": {
      "patient": {
        "name": "E2E Simulation Patient",
        "uhid": "PAT-2026-0001",
        "age": null,
        "gender": "MALE",
        "phone": "+919987721170"
      },
      "doctor": "Kiran Patil",
      "department": "Surgery",
      "barcode": "BC-10017",
      "order_id": "ddde2263-6386-4358-8271-a85c4a6dc3f3",
      "priority": "ROUTINE",
      "ordered_at": "2026-07-13T07:24:21Z",
      "result_id": "717701a9-d1e2-41c6-917b-7d7e2e123be9",
      "item_id": "799ec1df-b8a5-4e01-a271-86d58a66fa8d",
      "status": "READY_FOR_REVIEW",
      "digital_signature": false,
      "technician_remarks": "Specimen collected from left forearm",
      "comments": "Slight turbidity noted",
      "created_at": "2026-07-13T09:30:35Z",
      "parameters": [
        {
          "value_id": "3bf4b72b-1634-46da-ae96-5232b9c7d203",
          "actual_value": "11.2",
          "is_critical": false,
          "parameter_name": "Blood Culture Level",
          "parameter_code": "param-uuid-1111",
          "reference_range": "10.0-15.0",
          "unit": "g/dL",
          "critical_range_min": "8.00",
          "critical_range_max": "20.00"
        }
      ],
      "previous_reports": [],
      "status_history": [
        {
          "user_name": "Shivam Gupta",
          "action": "RESULT_SUBMITTED",
          "created_at": "2026-07-13T09:30:35Z",
          "ip_address": "127.0.0.1",
          "metadata": {}
        }
      ],
      "attachments": []
    }
  }
  ```

---

### Step 8: Pathologist Verdict (Approve or Reject)
Validates the report. Can transition status to `APPROVED` (with digital signature) or reject it back to `REPROCESSING` status.

* **Method:** `PATCH`
* **URL:** `/diagnostic-orders/lab/results/{resultId}/status`
* **Headers:**
  * `Authorization`: `Bearer <token>`
  * `Content-Type`: `application/json`
* **Request Body:**
  | Field | Type | Mandatory? | Description |
  | :--- | :--- | :--- | :--- |
  | `status` | String | **Yes** | `'APPROVED'` or `'RETURNED_FOR_CORRECTION'` |
  | `remarks` | String | Conditional | **Mandatory** if status is `'RETURNED_FOR_CORRECTION'` |
  | `digitalSignature`| Boolean | No | Set to `true` when signing off as approved |

* **Request Example (Approve):**
  ```json
  {
    "status": "APPROVED",
    "remarks": "Verified report parameters. Signed by pathologist.",
    "digitalSignature": true
  }
  ```
* **Success Response (`200 OK`):**
  ```json
  {
    "success": true,
    "code": 200,
    "data": {
      "id": "717701a9-d1e2-41c6-917b-7d7e2e123be9",
      "status": "APPROVED",
      "digital_signature": true,
      "validated_by": "143b85f9-0073-4b51-8357-bf5ccfcffcea",
      "validated_at": "2026-07-13T09:31:02.114102Z",
      "comments": "Slight turbidity noted\nPathologist Remarks (APPROVED): Verified report parameters. Signed by pathologist."
    },
    "message": "Result status updated to APPROVED successfully"
  }
  ```

---

### Step 9: Generate PDF Report
Renders the finalized, validated data to a PDF report document in AWS S3.

* **Method:** `PATCH`
* **URL:** `/diagnostic-orders/lab/results/{resultId}/status`
* **Headers:**
  * `Authorization`: `Bearer <token>`
  * `Content-Type`: `application/json`
* **Request Body:**
  ```json
  {
    "status": "REPORT_GENERATED"
  }
  ```
* **Success Response (`200 OK`):**
  ```json
  {
    "success": true,
    "code": 200,
    "data": {
      "id": "717701a9-d1e2-41c6-917b-7d7e2e123be9",
      "status": "REPORT_GENERATED",
      "report_url": "s3://hms-reports/rpt-717701a9.pdf"
    },
    "message": "Result status updated to REPORT_GENERATED successfully"
  }
  ```

---

### Step 10: Deliver Final Report
Transitions report to `REPORT_DELIVERED` status once released to patient/clinical portal.

* **Method:** `PATCH`
* **URL:** `/diagnostic-orders/lab/results/{resultId}/status`
* **Headers:**
  * `Authorization`: `Bearer <token>`
  * `Content-Type`: `application/json`
* **Request Body:**
  ```json
  {
    "status": "REPORT_DELIVERED"
  }
  ```
* **Success Response (`200 OK`):**
  ```json
  {
    "success": true,
    "code": 200,
    "data": {
      "id": "717701a9-d1e2-41c6-917b-7d7e2e123be9",
      "status": "REPORT_DELIVERED"
    },
    "message": "Result status updated to REPORT_DELIVERED successfully"
  }
  ```
