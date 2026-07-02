# HMS Discharge Module — Complete Design Plan

---

## 1. Patient Lifecycle — Unified Status System

Two fields, one source of truth. Everyone reads from the same DB record.

### `admission_status` — Is the patient in the hospital?

| Value | Meaning |
|---|---|
| `ADMITTED` | Patient is currently admitted |
| `DISCHARGED` | Patient has left the hospital |

Visible to: Patient module, Doctor, Nurse.

---

### `patient_status` — Where is the patient in their clinical journey?

**Path A — Medicational (no surgery needed)**

```
MONITORING → STABLE → DISCHARGE_READY
```

**Path B — Critical (surgery needed)**

```
PRE_SURGERY → IN_SURGERY → MONITORING → STABLE → DISCHARGE_READY
```

Visible to: Doctor and Nurse only.

---

### `condition` — Set at OPD assessment, drives the alerts page

| Value | Meaning |
|---|---|
| `CRITICAL` | Patient needs immediate admission |
| `MEDICATIONAL` | Patient needs monitoring and medication |
| `NONE` | Regular OPD, no admission needed |

Visible to: Doctor and Nurse only.

---

### Who sees what

| Field | Patient module | Doctor | Nurse |
|---|---|---|---|
| `admission_status` | ✓ | ✓ | ✓ |
| `patient_status` | ✓ | ✓ | ✓ |
| `condition` | — | ✓ | ✓ |

---

## 2. Four Types of Discharge

| Type | Who initiates | Trigger |
|---|---|---|
| `NORMAL` | Doctor | Patient is stable, clinically ready to go home |
| `DAMA` | Receptionist → Doctor | Family insists on leaving against medical advice |
| `TRANSFER` | Doctor / Admin | Hospital moving patient to their own branch |
| `DECEASED` | Doctor | Patient has passed away, body needs to be formally discharged |

---

## 3. Discharge Gates — Which Modules Must Clear

| Gate | NORMAL | DAMA | TRANSFER | DECEASED |
|---|---|---|---|---|
| BILLING | Mandatory | Mandatory | Mandatory | Mandatory |
| NURSING checklist | Mandatory | Skipped | Skipped | Skipped |
| Doctor summary | Mandatory | Skipped | Skipped | Skipped |
| Consent document | — | Mandatory | — | — |
| Pharmacy (future) | Mandatory | TBD | TBD | TBD |
| Insurance (future) | Mandatory | TBD | TBD | TBD |

Billing is the only gate mandatory for all four discharge types. No patient leaves without dues cleared.

---

## 4. Discharge Summary Checklist — Nurse Fills and Verifies (Normal Discharge)

All 9 must be `true` before summary can be submitted.

| # | Checklist Item | Field |
|---|---|---|
| 1 | Vitals are stable | `vitals_stable` |
| 2 | Medications explained to patient | `meds_explained` |
| 3 | Home care instructions given | `homecare_instructions_given` |
| 4 | Post-discharge medications clearly documented | `post_discharge_meds_documented` |
| 5 | All bills paid — pharmacy, lab, clinic, everything | `balance_zero` |
| 6 | Billing marked ready | `billing_ready` |
| 7 | Belongings handed over to patient/family | `belongings_handed` |
| 8 | Attending doctor signature obtained | `doctor_signed` |
| 9 | Attending nurse signature obtained | `nurse_signed` |

---

## 5. Database Design — Two Core Tables

### `discharge_cases` — One row per discharge, the master record

| Column | Type | Purpose |
|---|---|---|
| `id` | UUID PK | Primary key |
| `admission_id` | UUID FK | Links to patient admission |
| `tenant_id` | UUID FK | Which hospital |
| `discharge_type` | ENUM | NORMAL / DAMA / TRANSFER / DECEASED |
| `initiated_by` | UUID FK | Doctor who started discharge |
| `initiated_at` | TIMESTAMP | When discharge was initiated |
| `current_stage` | VARCHAR | Current workflow stage |
| `all_gates_cleared` | BOOLEAN | True when all clearances are CLEARED |
| `summary_submitted` | BOOLEAN | True when discharge summary is submitted |
| `finalized_at` | TIMESTAMP | When discharge was completed |
| `discharge_type_metadata` | JSONB | Type-specific data (see below) |

`discharge_type_metadata` stores:
- DAMA: consent details, family relation, document key
- TRANSFER: destination branch
- DECEASED: time of death, certifying doctor

---

### `discharge_clearances` — One row per gate per discharge

| Column | Type | Purpose |
|---|---|---|
| `id` | UUID PK | Primary key |
| `discharge_case_id` | UUID FK | Links to discharge_cases |
| `gate_name` | ENUM | BILLING / NURSING / PHARMACY / INSURANCE |
| `gate_status` | ENUM | PENDING / CLEARED / BLOCKED |
| `cleared_by` | UUID FK | User who cleared this gate |
| `cleared_at` | TIMESTAMP | When it was cleared |
| `blocking_reason` | VARCHAR | If BLOCKED, why |

`admission_id` is intentionally NOT repeated here. It lives only in `discharge_cases`. `discharge_case_id` is the only link needed. No redundancy.

---

### How the two tables relate

```
discharge_cases (one row)
    id = dc-001
    admission_id = adm-ramesh-001
         |
         | (one-to-many)
         |
discharge_clearances (one row per gate)
    discharge_case_id = dc-001 | gate = BILLING  | status = CLEARED
    discharge_case_id = dc-001 | gate = NURSING  | status = PENDING
    discharge_case_id = dc-001 | gate = PHARMACY | status = PENDING  ← added in future, zero code change
```

---

### Plug-in gate architecture — how future modules integrate

The discharge service logic never changes:

```python
def can_discharge(discharge_case_id):
    gates = get_all_clearances(discharge_case_id)
    return all(gate.status == 'CLEARED' for gate in gates)
```

Adding pharmacy in future:
- Pharmacy module exposes one endpoint: `POST /pharmacy/discharge-clearance/{admission_id}`
- One new row inserted into `discharge_clearances` when discharge initiates
- Discharge service picks it up automatically
- Nothing else touched

---

## 6. DAMA Flow — Discharge Against Medical Advice

### What it is

Patient's family wants to leave against the doctor's advice. They sign a legal consent document. Patient is discharged despite medical recommendation to stay.

### DAMA lifecycle

```
INITIATED → PENDING_CONSENT → PENDING_APPROVAL → APPROVED → COMPLETED
                                      ↓
                                  CANCELLED (allowed from any stage)
```

### Stage by stage

| Stage | Who acts | What happens |
|---|---|---|
| `INITIATED` | Receptionist → Doctor | Family approaches receptionist. Reason logged. Doctor notified. Doctor formally initiates DAMA. |
| `PENDING_CONSENT` | Family | Family signs consent document. Relation noted. Document uploaded. |
| `PENDING_APPROVAL` | Senior doctor / Admin | If `requires_approval = true`, senior must approve. |
| `APPROVED` | Approver | Approval recorded with notes. |
| `COMPLETED` | Doctor / Nurse | Patient leaves. Billing must be cleared first. |
| `CANCELLED` | Anyone | Family changed their mind. Case closed. |

### DAMA business rules

- Only one active DAMA per admission. New one can only start if previous is `CANCELLED`
- Cannot start DAMA if patient is already `DISCHARGED`
- Cannot complete without consent being obtained
- Billing gate is mandatory — patient cannot leave without clearing dues
- On completion, same cascade fires as normal discharge

### `dama_requests` table — all columns

| Column | Purpose |
|---|---|
| `id` | Primary key |
| `tenant_id` | Which hospital |
| `admission_id` | Which admission |
| `patient_id` | Which patient |
| `initiated_by` | Doctor who formally initiated |
| `reason` | Why family wants to leave |
| `clinical_summary` | Doctor's note on current condition and risk |
| `requires_approval` | Does a senior need to approve |
| `status` | Current stage |
| `consent_obtained` | Boolean |
| `consent_by` | Family member name |
| `consent_relation` | Their relation to patient |
| `consent_obtained_at` | Timestamp |
| `consent_document_key` | S3 key of signed document |
| `approved_by` | Who approved |
| `approved_at` | When |
| `approval_notes` | Notes from approver |
| `completed_by` | Who completed |
| `completed_at` | When |
| `discharge_notes` | Final notes |
| `cancellation_reason` | If cancelled, why |

---

## 7. Transfer Discharge

Hospital moves patient to their own branch.

- Destination branch recorded in `discharge_type_metadata`
- Referral summary generated
- Patient records shared with receiving branch
- Bed released at source branch
- New admission created at destination branch

---

## 8. Deceased Discharge

Patient has passed away. Body needs to be formally discharged.

- Time of death recorded
- Doctor certifies death
- Death summary generated
- Body handover to family recorded

---

## 9. Enterprise Operational Discharge Workflow (Validated)

This flow aligns with existing HMS architecture and clarifies what is already reusable vs what must be added.

### Validation Scope

- What is already correct
- What endpoints already exist
- What endpoints are missing
- What should be modified
- Where workflow responsibilities should live
- Where future issues may occur

---

### Step 1 - Doctor Changes Patient Status

Doctor dashboard has a Patient Status dropdown:

- `MONITORING`
- `STABLE`
- `DISCHARGE_READY`

When doctor selects `DISCHARGE_READY`, discharge flow begins.

Existing endpoint:

- `POST /ops/discharge/initiate` (already supported)

Important recommendation:

Do not trigger discharge directly from dropdown selection. Use:

`DISCHARGE_READY` -> Confirmation Modal -> Initiate Discharge -> `POST /ops/discharge/initiate`

Reason: discharge initiation is a serious operational action.

---

### Step 2 - Create Discharge Case

When `POST /ops/discharge/initiate` is called, also create a `discharge_cases` record.

| Field | Value |
|---|---|
| `admission_id` | Patient admission |
| `discharge_type` | `NORMAL` |
| `current_stage` | `BILLING_PENDING` |
| `initiated_by` | Doctor |
| `initiated_at` | Timestamp |

This is a critical part of workflow tracking.

---

### Step 3 - Head Nurse Discharge Management Page

Required queue access:

- Preferred: `GET /discharge/cases?stage=BILLING_PENDING`
- Alternate: `GET /discharge/head-nurse/queue`

Preferred approach is the generic cases filter because future stages become filterable (`PHARMACY_PENDING`, `INSURANCE_PENDING`, `SUMMARY_PENDING`).

Queue should show:

- Patient
- MRN
- Ward
- Room
- Doctor
- Discharge stage
- Billing status
- Delay time

---

### Step 4 - Billing Clearance

Billing workflow should be query-driven, not notification-coupled.

Reception dashboard flow:

1. `GET /discharge/cases?stage=BILLING_PENDING`
2. Open patient
3. `GET /billing/clearance/{admission_id}` (already exists)

Missing write endpoint:

- Recommended: `POST /billing/clearance/{admission_id}/clear`
- Alternative: `PATCH /billing/clearance/{admission_id}`

Clear action should:

- Mark billing cleared
- Set `cleared_by`
- Set `cleared_at`
- Update `discharge_clearances`
- Move discharge stage to `SUMMARY_PENDING`

Backend must own stage transitions. Frontend must not directly enable summary logic.

---

### Step 5 - Summary Enabled

Once billing clears:

- Doctor dashboard and nurse dashboard receive `summary_enabled = true`

Ownership model:

- Summary is collaborative
- Both doctor and assigned nurse can edit

---

### Step 6 - Discharge Summary API

Required endpoints:

- `GET /discharge/summary/{admission_id}`
- `PATCH /discharge/summary/{admission_id}`

Summary payload design:

- Auto-generated sections from HMS data
- Editable sections limited to:
- Post-discharge medications
- Home care instructions
- Follow-up advice

---

### Step 7 - Medications Endpoints

Required endpoints:

- `POST /discharge/{admission_id}/medications`
- `GET /discharge/{admission_id}/medications`

Store in dedicated table: `discharge_medications`.

Do not embed medication list inside summary JSON blob.

---

### Step 8 - Home Care Instructions Endpoints

Required endpoints:

- `POST /discharge/{admission_id}/instructions`
- `GET /discharge/{admission_id}/instructions`

Store in dedicated table: `homecare_instructions`.

Do not store as free-form summary-only text.

---

### Step 9 - Discharge Checklist Reuse

Existing endpoint to reuse:

- `PATCH /nurses/admissions/{admission_id}/discharge-checklist`

Reuse strategy is correct; do not rebuild checklist module.

Checklist schema should be extended with:

- `doctor_signed`
- `nurse_signed`
- `post_discharge_meds_documented`
- `homecare_instructions_given`
- `balance_zero`

---

### Step 10 - Submit Summary

Required endpoint:

- `POST /discharge/{admission_id}/submit-summary`

Validations before submit:

- Billing cleared
- Checklist complete
- Medications exist
- Instructions exist
- Doctor signed
- Nurse signed

On success:

- `summary_submitted = true`
- `current_stage = READY_FOR_FINAL_DISCHARGE`

---

### Step 11 - Final Discharge

Existing endpoint:

- `POST /ops/discharge/complete` (reuse; do not rewrite)

This should continue handling:

- Discharge lifecycle completion
- Bed release
- Audit hooks

---

### Critical Missing Capability - Workflow Logs

Add `discharge_workflow_logs` for full auditability.

Track actions:

- Initiated
- Billing cleared
- Checklist completed
- Medications added
- Summary submitted
- Discharge completed

Benefits:

- Audit trail
- Medico-legal history
- Operational analytics

---

### Final Endpoint List (Validated)

#### Already Exists

| Endpoint | Status |
|---|---|
| `POST /ops/discharge/initiate` | Yes |
| `POST /ops/discharge/complete` | Yes |
| `GET /billing/clearance/{admission_id}` | Yes |
| `PATCH /nurses/admissions/{admission_id}/discharge-checklist` | Yes |
| `GET discharge checklist` | Yes |

#### New Endpoints Needed

Discharge Cases:

- `GET ops/discharge/cases`
- `GET ops/discharge/cases/{admission_id}`

Billing Clearance Action:

- `POST /billing/clearance/{admission_id}/clear`
(Find if we have any similar apis?)
Summary:

- `GET ops/discharge/summary/{admission_id}`
- `PATCH ops/discharge/summary/{admission_id}`
- `POST ops/discharge/{admission_id}/submit-summary`

Medications:

- `POST ops/discharge/{admission_id}/medications`
- `GET ops/discharge/{admission_id}/medications`

Home Care:

- `POST ops/discharge/{admission_id}/instructions`
- `GET ops/discharge/{admission_id}/instructions`

One additional high-value endpoint:

- `GET /discharge/{admission_id}/timeline`

This timeline should show:

- Initiated
- Billing cleared
- Checklist done
- Summary submitted
- Discharge completed
- All strings detached
- Records permanently archived

---

## 9. On Final Discharge — Cascade of Events

Fires for all four discharge types on completion.

| Event | What happens |
|---|---|
| `admission_status` | `ADMITTED → DISCHARGED` |
| `patient_status` | `DISCHARGE_READY → null` |
| Bed | `bed_status → VACANT` |
| IPD list | Patient removed from active IPD |
| Billing | Frozen — no new charges possible |
| Assigned nurse | Detached from patient |
| Assigned doctor | Detached from patient |
| Active medications | Closed |
| Discharge summary | Locked — no further edits |

---

## 10. What Stays in the Database Forever

All data is retained. Only active hospital strings are cut.

- Full patient record
- All OPD and IPD visits
- All diagnoses and conditions
- All lab, pharmacy, and billing history
- The discharge summary (locked after submission)
- DAMA consent documents and audit trail
- Workflow logs of every action taken

---

## 11. Modules Involved Now vs Future

| Module | Role | Status |
|---|---|---|
| OPS | Discharge orchestration, DAMA flow, bed release | Now |
| Patient | Admission records, status tracking | Now |
| Doctor | Initiates discharge, signs summary | Now |
| Nurse | Checklist, summary, signs | Now |
| Pharmacy | Pharmacy gate clearance | Future |
| Insurance | Insurance gate clearance | Future |

---

*Designed for HMS. All future modules plug into `discharge_clearances` with zero changes to core discharge logic.*

Enterprise Operational Discharge Workflow

And the best part is:
it aligns properly with the existing HMS architecture instead of conflicting with it.

Now let me validate your full flow carefully and identify:

✅ what is already correct
✅ what endpoints already exist
✅ what endpoints are missing
✅ what should be modified
✅ where workflow responsibilities should live
✅ where you may face future issues
FINAL NORMAL DISCHARGE FLOW (VALIDATED)
STEP 1 — Doctor Changes Patient Status

Doctor dashboard:

Patient Status Dropdown

Possible statuses:

MONITORING
STABLE
DISCHARGE_READY

Once doctor selects:

DISCHARGE_READY

Flow begins.

EXISTING THING YOU ALREADY HAVE

OPS already supports:

POST /ops/discharge/initiate

✅ Correct.

So doctor UI should call this endpoint.

IMPORTANT RECOMMENDATION

Do NOT directly trigger discharge from dropdown alone.

Instead:

Dropdown -> DISCHARGE_READY
        ↓
Confirmation Modal
        ↓
Initiate Discharge
        ↓
POST /ops/discharge/initiate

Because discharge initiation is a serious operational action.

STEP 2 — CREATE DISCHARGE CASE

VERY IMPORTANT.

When:

/ops/discharge/initiate

is called:

You should ALSO create:

discharge_cases

record.

This is critical.

WHAT SHOULD HAPPEN HERE

Create:

Field	Value
admission_id	patient admission
discharge_type	NORMAL
current_stage	BILLING_PENDING
initiated_by	doctor
initiated_at	timestamp
STEP 3 — HEAD NURSE DISCHARGE MANAGEMENT PAGE

YES.
This is correct.

You need:

NEW ENDPOINT
GET /discharge/head-nurse/queue

OR

GET /discharge/cases?stage=BILLING_PENDING

Better approach:
second one.

Because later:

pharmacy pending
insurance pending
summary pending

all become filterable.

WHAT THIS PAGE SHOULD SHOW
Field
patient
MRN
ward
room
doctor
discharge stage
billing status
delay time

This becomes operational dashboard.

Excellent feature.

STEP 4 — BILLING NOTIFICATION

You said:

notify receptionist dashboard

YES.
Correct.

But architecturally:

billing should poll/query discharge queue

Instead of hard notification dependency.

BETTER ARCHITECTURE

Reception dashboard calls:

GET /discharge/cases?stage=BILLING_PENDING

Then receptionist opens patient.

Then:

GET /billing/clearance/{admission_id}

which already exists.

Excellent reuse.

IMPORTANT MISSING ENDPOINT

You correctly identified this.

Current billing only has:

GET /billing/clearance/{admission_id}

But receptionist must CLEAR billing.

So YES:
you need:

NEW ENDPOINT
PATCH /billing/clearance/{admission_id}

OR better:

POST /billing/clearance/{admission_id}/clear

This is cleaner enterprise style.

WHAT THIS SHOULD DO
Action
mark billing cleared
set cleared_by
set cleared_at
update discharge_clearances
move discharge stage
VERY IMPORTANT

DO NOT directly enable summary from frontend logic.

Instead backend should control:

current_stage = SUMMARY_PENDING

ONLY after billing cleared.

STEP 5 — SUMMARY ENABLED

YES.
Correct.

Once billing clears:

Doctor dashboard + nurse dashboard get:

summary_enabled = true
VERY IMPORTANT DESIGN

Summary should NOT be role-owned.

Correct decision.

Both:

doctor
assigned nurse

can edit collaboratively.

Excellent.

STEP 6 — DISCHARGE SUMMARY API

YES.
You need this.

NEW ENDPOINT REQUIRED
GET SUMMARY
GET ops/discharge/summary/{admission_id}

Returns:

auto-fetched data
editable sections
medications
instructions
checklist state
SAVE SUMMARY
PATCH ops/discharge/summary/{admission_id}

OR

POST ops/discharge/summary/{admission_id}

PATCH better.

IMPORTANT ARCHITECTURAL DECISION

Most fields:

AUTO GENERATED

Only editable:

Editable
post discharge meds
home care instructions
followup advice

Everything else should come from HMS automatically.

Correct design.

STEP 7 — MEDICATIONS ENDPOINTS

YES.
You correctly identified this.

Need:

ADD MEDICATION
POST ops/discharge/{admission_id}/medications
GET MEDICATIONS
GET ops/discharge/{admission_id}/medications
IMPORTANT

Do NOT store medications inside summary JSON blob.

Make separate table:

discharge_medications

Correct enterprise approach.

STEP 8 — HOME CARE INSTRUCTIONS

YES.
Also correct.

Need:

ADD INSTRUCTION
POST ops/discharge/{admission_id}/instructions
GET INSTRUCTIONS
GET ops/discharge/{admission_id}/instructions
AGAIN IMPORTANT

Separate table:

homecare_instructions

NOT inside summary text.

STEP 9 — DISCHARGE CHECKLIST

YES.
Already exists.

And your reuse strategy is CORRECT.

Existing endpoint:

PATCH /nurses/admissions/{admission_id}/discharge-checklist

should absolutely be reused.

Do NOT rebuild checklist system.

Good decision.

BUT IMPORTANT CHANGE NEEDED

Current checklist fields are OLD.

You proposed improved checklist:

New Field
doctor_signed
nurse_signed
post_discharge_meds_documented
homecare_instructions_given
balance_zero

These are VERY important.

So:

checklist schema needs extension.
STEP 10 — SUMMARY SUBMIT

YES.
Correct.

Need endpoint:

POST ops/discharge/{admission_id}/submit-summary
WHAT THIS SHOULD VALIDATE

Before allowing submission:

Validation
billing cleared
checklist complete
medications exist
instructions exist
doctor signed
nurse signed

ONLY THEN:

summary_submitted = true
current_stage = READY_FOR_FINAL_DISCHARGE
STEP 11 — FINAL DISCHARGE

You correctly identified:

Already exists:

POST ops/discharge/complete

Excellent.

DO NOT rewrite this.

Reuse it.

WHAT SHOULD HAPPEN HERE

OPS already handles:

discharge lifecycle
bed release
audit hooks

Perfect.

IMPORTANT MISSING THING

You are currently missing:

WORKFLOW LOGS

VERY IMPORTANT.

Need:

discharge_workflow_logs

Track:

Action
initiated
billing cleared
checklist completed
meds added
summary submitted
discharge completed

This becomes:

audit trail
medico-legal history
operational analytics
FINAL ENDPOINT LIST (VALIDATED)
ALREADY EXISTS
Endpoint	Status
POST /ops/discharge/initiate	✅
POST /ops/discharge/complete	✅
GET /billing/clearance/{admission_id}	✅
PATCH /nurses/admissions/{admission_id}/discharge-checklist	✅
GET discharge checklist	✅
NEW ENDPOINTS NEEDED
DISCHARGE CASES
GET ops/discharge/cases
GET ops/discharge/cases/{admission_id}
BILLING CLEARANCE ACTION
POST /billing/clearance/{admission_id}/clear
SUMMARY
GET ops/discharge/summary/{admission_id}
PATCH ops/discharge/summary/{admission_id}
POST ops/discharge/{admission_id}/submit-summary
MEDICATIONS
POST ops/discharge/{admission_id}/medications
GET ops/discharge/{admission_id}/medications
HOME CARE
POST ops/discharge/{admission_id}/instructions
GET ops/discharge/{admission_id}/instructions
ONE MORE IMPORTANT ENDPOINT YOU MISSED

Need:

GET ops/discharge/{admission_id}/timeline

This will show:

initiated
billing cleared
checklist done
summary submitted
discharge completed

Very useful operationally.
