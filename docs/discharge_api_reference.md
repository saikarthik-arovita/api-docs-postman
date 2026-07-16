# Discharge Workflow API Reference

> **Service**: IPD Service  
> **Base URL**: `{{ipd_base_url}}`  
> **Auth**: `Authorization: Bearer <access_token>`  
> **Tenant**: `x-tenant-id: <branch_id>` (required header)

---

## Discharge Workflow Stages

```
DISCHARGE INITIATED → DOCTOR APPROVAL → BILLING CLEARANCE → FINAL DISCHARGE
       Step 1              Step 2              Step 3              Step 4
```

---

## Step 0 — Get Discharge Summary (Read-only)

Fetch the full discharge summary at any stage. Powers the "Discharge Summary" panel in the UI.

### `GET /ipd/admissions/{id}/discharge-summary`

#### Path Params
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | UUID | Yes | Admission ID |

#### Response `200 OK`
```json
{
  "success": true,
  "data": {
    "id": "admission-uuid",
    "admission_number": "ADM-2026-0042",
    "patient_id": "patient-uuid",
    "patient_name": "Ravi Kumar",
    "patient_uhid": "UHID-001234",
    "patient_phone": "9876543210",
    "attending_doctor_id": "doctor-uuid",
    "attending_doctor_name": "Dr. Priya Sharma",
    "ward_id": "ward-uuid",
    "ward_name": "General Ward A",
    "bed_id": "bed-uuid",
    "bed_number": "G-101",
    "admission_type": "REGULAR",
    "admission_reason": "Hypertension with chest pain",
    "status": "ADMITTED",
    "ipd_status": "READY_FOR_DISCHARGE",
    "discharge_type": "NORMAL",
    "admitted_at": "2026-07-10T08:00:00Z",
    "actual_discharge_at": null,
    "case_closed_at": null,
    "discharge_summary": "Patient stable. BP controlled. Ready for discharge.",
    "follow_up_date": "2026-07-30",
    "follow_up_instructions": "Revisit cardiology OPD in 2 weeks.",
    "medications_on_discharge": "Tab Amlodipine 5mg OD, Tab Aspirin 75mg OD",
    "diet_instructions": "Low sodium diet. Avoid processed foods.",
    "discharge_case": {
      "discharge_case_id": "case-uuid",
      "case_discharge_type": "NORMAL",
      "current_stage": "BILLING",
      "all_gates_cleared": false,
      "initiated_at": "2026-07-16T06:00:00Z",
      "finalized_at": null
    },
    "clearances": [
      { "gate_name": "BILLING",    "gate_status": "PENDING",  "cleared_by": null, "cleared_at": null, "blocking_reason": null },
      { "gate_name": "DOCTOR",     "gate_status": "CLEARED",  "cleared_by": "doctor-uuid", "cleared_at": "2026-07-16T07:00:00Z", "blocking_reason": null },
      { "gate_name": "NURSING",    "gate_status": "CLEARED",  "cleared_by": "nurse-uuid", "cleared_at": "2026-07-16T07:30:00Z", "blocking_reason": null },
      { "gate_name": "PHARMACY",   "gate_status": "PENDING",  "cleared_by": null, "cleared_at": null, "blocking_reason": null }
    ],
    "created_at": "2026-07-10T08:00:00Z",
    "updated_at": "2026-07-16T07:30:00Z"
  }
}
```

#### Error Responses
| Status | Reason |
|--------|--------|
| `404` | Admission not found |

---

## Step 0b — Save Discharge Summary Draft (Edit Draft)

Update narrative fields at any time without triggering status transitions. Powers the **"Edit Draft"** button.

### `PATCH /ipd/admissions/{id}/discharge-summary`

#### Path Params
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | UUID | Yes | Admission ID |

#### Request Body (all optional, at least one required)
```json
{
  "discharge_summary": "Patient stable, blood pressure controlled under medication.",
  "follow_up_date": "2026-07-30",
  "follow_up_instructions": "Review cardiology OPD in 2 weeks. Repeat ECG on next visit.",
  "medications_on_discharge": "Tab Amlodipine 5mg OD\nTab Aspirin 75mg OD\nTab Atorvastatin 10mg OD",
  "diet_instructions": "Low sodium diet. Avoid processed food. No smoking."
}
```

#### Request Body Fields
| Field | Type | Description |
|-------|------|-------------|
| `discharge_summary` | string | Doctor's clinical narrative |
| `follow_up_date` | date (`YYYY-MM-DD`) | Date for follow-up OPD visit |
| `follow_up_instructions` | string | Textual follow-up instructions |
| `medications_on_discharge` | string | List of discharge medicines |
| `diet_instructions` | string | Dietary advice |

#### Response `200 OK`
Returns the same shape as `GET /discharge-summary`.

#### Error Responses
| Status | Reason |
|--------|--------|
| `400` | No fields provided / already closed case |
| `404` | Admission not found |
| `409` | Case is already permanently closed |

---

## Step 1 — Initiate Discharge

Doctor marks patient as ready for discharge. Transitions: `UNDER_TREATMENT → READY_FOR_DISCHARGE`. Creates the discharge case and all clearance gates.

### `POST /ipd/admissions/{id}/discharge/initiate`

#### Path Params
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | UUID | Yes | Admission ID |

#### Request Body
```json
{
  "discharge_summary": "Patient is stable. Blood pressure well controlled. Cleared for discharge.",
  "follow_up_date": "2026-07-30",
  "follow_up_instructions": "Review cardiology OPD in 2 weeks.",
  "notes": "Patient and family counselled."
}
```

#### Request Body Fields
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `discharge_summary` | string | Recommended | Clinical narrative |
| `follow_up_date` | date (`YYYY-MM-DD`) | Optional | Follow-up OPD date |
| `follow_up_instructions` | string | Optional | Follow-up text |
| `notes` | string | Optional | Internal notes |

#### Response `200 OK`
```json
{
  "success": true,
  "data": {
    "admission_id": "admission-uuid",
    "ipd_status": "READY_FOR_DISCHARGE"
  }
}
```

#### Error Responses
| Status | Reason |
|--------|--------|
| `400` | Invalid state transition |
| `404` | Admission not found |
| `409` | Case already closed |

---

## Step 2 & 3 — Update Clearance Gates

Update approval status for each gate. Powers **"Approve" / "Request Changes"** (Doctor Gate) and **"Verify Billing Clearance"** buttons.

### `POST /ipd/admissions/{id}/discharge/clearance`

#### Path Params
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | UUID | Yes | Admission ID |

#### Request Body
```json
{
  "gate_name": "DOCTOR",
  "gate_status": "CLEARED",
  "blocking_reason": null
}
```

#### Request Body Fields
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `gate_name` | string | Yes | Gate to update. See values below |
| `gate_status` | string | Yes | New gate status. See values below |
| `blocking_reason` | string | Optional | Reason if blocking/pending |

#### `gate_name` Values
| Gate | Who Updates | UI Section |
|------|-------------|------------|
| `DOCTOR` | Attending doctor | Doctor Approval panel |
| `BILLING` | Billing staff | Billing Clearance panel |
| `PHARMACY` | Pharmacist | Pharmacy Gate (checklist) |
| `NURSING` | Nursing staff | Nursing Gate (checklist) |

#### `gate_status` Values
| Status | Meaning |
|--------|---------|
| `CLEARED` | Gate approved / cleared |
| `PENDING` | Not yet cleared (default) |
| `BLOCKED` | Gate is blocking discharge (add `blocking_reason`) |

#### Doctor Approval — "Approve" button
```json
{ "gate_name": "DOCTOR", "gate_status": "CLEARED" }
```

#### Doctor Approval — "Request Changes" button
```json
{
  "gate_name": "DOCTOR",
  "gate_status": "BLOCKED",
  "blocking_reason": "Discharge summary needs revision. Labs not reviewed."
}
```

#### Verify Billing Clearance
```json
{ "gate_name": "BILLING", "gate_status": "CLEARED" }
```

#### Response `200 OK`
```json
{
  "success": true,
  "data": {
    "gate_name": "DOCTOR",
    "gate_status": "CLEARED",
    "cleared_by": "user-uuid",
    "cleared_at": "2026-07-16T09:00:00Z",
    "blocking_reason": null
  }
}
```

#### Error Responses
| Status | Reason |
|--------|--------|
| `400` | Missing gate_name or gate_status / no active discharge case |
| `404` | Admission not found |

---

## Step 4 — Final Discharge

Completes discharge. All gates must be `CLEARED`. Releases the bed, marks admission `DISCHARGED`.

### `POST /ipd/admissions/{id}/discharge/complete`

#### Path Params
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | UUID | Yes | Admission ID |

#### Request Body (all optional if fields were saved via draft)
```json
{
  "discharge_summary": "Patient stable. Vitals normal. Full recovery expected.",
  "follow_up_date": "2026-07-30",
  "follow_up_instructions": "OPD visit in 2 weeks. Repeat ECG.",
  "medications_on_discharge": "Tab Amlodipine 5mg OD\nTab Aspirin 75mg OD",
  "diet_instructions": "Low sodium diet."
}
```

#### Request Body Fields
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `discharge_summary` | string | Required* | *Required if not saved in draft |
| `follow_up_date` | date | Optional | |
| `follow_up_instructions` | string | Optional | |
| `medications_on_discharge` | string | Optional | |
| `diet_instructions` | string | Optional | |

#### Response `200 OK`
```json
{
  "success": true,
  "data": {
    "id": "admission-uuid",
    "admission_number": "ADM-2026-0042",
    "status": "DISCHARGED",
    "ipd_status": "DISCHARGED",
    "discharge_type": "NORMAL",
    "actual_discharge_at": "2026-07-16T10:30:00Z",
    "discharge_summary": "Patient stable. Vitals normal. Full recovery expected.",
    "follow_up_date": "2026-07-30",
    "follow_up_instructions": "OPD visit in 2 weeks. Repeat ECG.",
    "medications_on_discharge": "Tab Amlodipine 5mg OD\nTab Aspirin 75mg OD",
    "diet_instructions": "Low sodium diet."
  }
}
```

#### Error Responses
| Status | Reason |
|--------|--------|
| `400` | Pending clearance gates / discharge_summary missing |
| `404` | Admission not found |
| `409` | Case already closed / invalid state |

---

## Workflow Summary Table

| UI Step | Button / Action | API | Method |
|---------|-----------------|-----|--------|
| Any time | Load summary page | `/ipd/admissions/{id}/discharge-summary` | `GET` |
| Any time | Edit Draft | `/ipd/admissions/{id}/discharge-summary` | `PATCH` |
| Step 1 | Discharge Initiated | `/ipd/admissions/{id}/discharge/initiate` | `POST` |
| Step 2 | Approve (Doctor Gate) | `/ipd/admissions/{id}/discharge/clearance` | `POST` |
| Step 2 | Request Changes | `/ipd/admissions/{id}/discharge/clearance` | `POST` |
| Step 3 | Verify Billing Clearance | `/ipd/admissions/{id}/discharge/clearance` | `POST` |
| Step 4 | Final Discharge Patient | `/ipd/admissions/{id}/discharge/complete` | `POST` |

---

## Gate Clearance Status — Checklist Mapping

| Checklist Item | Gate Name |
|----------------|-----------|
| Discharge Summary | Checked via `discharge_summary` field |
| Doctor Gate | `gate_name: DOCTOR` |
| Billing Gate | `gate_name: BILLING` |
| Pharmacy Gate | `gate_name: PHARMACY` |
| Nursing Gate | `gate_name: NURSING` |
