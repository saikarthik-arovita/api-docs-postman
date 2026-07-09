# OT / Surgery — Simple Unified Flow

## The one rule

Before any surgery, the system needs exactly **2 things**:

1. **A surgery advice** — doctor said "this patient needs surgery" → ONE api: `POST /opd/surgery` (works for OPD and IPD patients — just pass `opd_visit_id` OR `admission_id`).
2. **A bed** — patient must be admitted. If already admitted → skip. If not → ONE action: "Admit Patient".

After those 2 → one OT Case runs everything: `Schedule → Consents → PAC → Pre-Op → Start → Complete → Transfer-Out`.

**All scenarios differ ONLY in step 2 (the bed).** Everything else is identical for everyone.

---

## The 3 real-hospital scenarios

### SCENARIO A — OPD patient (elective) · most common

> Patient visits clinic → doctor decides surgery → patient gets admitted → surgery.

| Step | Who | Screen | Action / API |
|---|---|---|---|
| 1 | Doctor | OPD patient detail | **"Advise Surgery"** → `POST /opd/surgery` with `opd_visit_id` |
| 2 | Doctor | same dialog chains → | **"Admit Patient"** (pick ward + bed) → admission created, linked to the advice |
| 3 | Doctor | patient page / Surgery&OT page | **"Schedule OT Case"** (room, date/time, team) → `POST /ipd/ot-sessions` |
| 4 | Doctor + patient | OT Case page | **Consents** — create + sign → `POST /ipd/ot-cases/{id}/consents`, `POST /ipd/consents/{id}/sign` |
| 5 | Doctor/Anaesthetist | OT Case page | **Clear PAC** → `POST /ipd/ot-sessions/{id}/pac` |
| 6 | Nurse | OT Case page (or OT Suite) | **Pre-Op checklist** → `POST /ipd/ot-sessions/{id}/pre-op` |
| 7 | Doctor | OT Case page | **Start** → `/start` … **Complete** (notes + anaesthesia) → `/complete` |
| 8 | Nurse | OT Case page | **Transfer Out** (ward + bed + recovery notes) → `/transfer-out` — OT freed, patient back in ward |
| 9 | — | — | normal post-op care → existing discharge flow |

### SCENARIO B — IPD patient (already admitted)

> Patient already in a bed (admitted for illness / after workup) → ward round decides surgery.

Same as Scenario A but **step 2 is skipped** — the patient already has `admission_id`.

| Step | Action / API |
|---|---|
| 1 | **"Advise Surgery"** from IPD patient detail → `POST /opd/surgery` with `admission_id` |
| 2 | ~~admit~~ — already admitted, dialog jumps straight to → |
| 3–9 | Schedule OT Case → Consents → PAC → Pre-Op → Start → Complete → Transfer-Out (identical to A) |

### SCENARIO C — Emergency

> ER patient (appendicitis, trauma) needs surgery NOW.

Same pipeline, compressed:

| Step | Difference from A |
|---|---|
| 1 | Advise Surgery with `priority: EMERGENCY` |
| 2 | Admit immediately (ER bed flow already exists) — or already admitted |
| 3 | Schedule into next available / emergency OT slot |
| 4 | Consents — guardian signs if patient can't (sign dialog already supports guardian) |
| 5 | **PAC emergency override** — 2 doctors + reason (already built): `{"emergency_override": true, "override_reason": "…", "primary_doctor_id": "…", "second_doctor_id": "…"}` |
| 6–8 | Pre-op (short) → Start → Complete → Transfer-Out — same screens, just faster |

---

## What's needed vs NOT needed

| Thing | Verdict | Why |
|---|---|---|
| `POST /opd/surgery` | ✅ THE only create | already accepts both `opd_visit_id` and `admission_id` |
| `POST /ipd/surgeries` (+ its list/schedule/start/complete) | ❌ REMOVE | duplicate of create + ot-session — this is the #1 source of confusion |
| Admission request → approve → admit (3 user steps) | ⚠️ COLLAPSE to 1 | the surgeon IS the approver for their own surgery patient. UI shows one **"Admit Patient"** dialog (ward+bed). Backend: either allow direct `POST /ipd/admissions` with `service_order_id`, or the frontend silently fires request→READY→admit in one go. The request entity stays only as an internal record — the user never sees a 3-step approval |
| `POST /ipd/ot-sessions` + pac/pre-op/start/complete/transfer-out + consents | ✅ KEEP as-is | this is the working case engine (the stepper already drives it) |
| `/ops/ot/*` (nurse legacy suite, 9 endpoints, fake PAC stub) | ❌ REMOVE | parallel world, disconnected from real cases |
| `/ops/surgery*` | ❌ REMOVE | old duplicate of create |
| Consumables + Cancel on ot-session | ➕ ADD | `POST /ipd/ot-sessions/{id}/consumables` `{item_name, quantity, batch_no?, implant_serial?}` · `POST /ipd/ot-sessions/{id}/cancel` `{reason}` (both exist only on the legacy flow today) |

---

## API quick reference (final set — nothing else)

```json
// 1. CREATE (both scenarios — send whichever id you have)
POST /opd/surgery
{ "patient_id": "…", "opd_visit_id": "…" OR "admission_id": "…",
  "service_name": "Lap Cholecystectomy", "service_code": "SURG-104",
  "order_type": "SURGERY", "priority": "ROUTINE|URGENT|EMERGENCY",
  "assigned_doctor_id": "…", "notes": "…", "pre_op_instructions": "…" }
→ data: { "id": "so-771", "status": "ADVISED", "admission_id": null|"…", "ot_session_id": null }
   // admission_id null = needs step 2 · filled = skip to step 3

// 2. ADMIT (only if admission_id was null) — ONE user action
POST /ipd/admissions
{ "patient_id": "…", "ward_id": "…", "bed_id": "…", "attending_doctor_id": "…",
  "admission_reason": "Surgery: Lap Chole", "admission_type": "ELECTIVE|EMERGENCY",
  "service_order_id": "so-771" }        // ← backend links it back to the advice

// 3. SCHEDULE = create the OT case
POST /ipd/ot-sessions
{ "service_order_id": "so-771", "admission_id": "…", "patient_id": "…",
  "surgeon_id": "…", "anaesthetist_id": "…", "scrub_nurse_id": "…",
  "ot_room_id": "OT-2", "scheduled_start": "2026-07-16T08:00:00Z" }
→ data: { "id": "ots-31", "status": "SCHEDULED", ... }

// 4. CONSENTS
POST /ipd/ot-cases/ots-31/consents   { "consent_codes": ["SURGICAL","ANAESTHESIA","PROCEDURE","BLOOD_TRANSFUSION"] }
POST /ipd/consents/{id}/sign         { "signature_type": "PATIENT", "signed_by": "…", "witness_id": "…" }

// 5. PAC (all mandatory consents signed unlocks this)
POST /ipd/ot-sessions/ots-31/pac     { "notes": "ASA 2, NPO confirmed" }        → status PRE_OP

// 6. PRE-OP (nurse)
POST /ipd/ot-sessions/ots-31/pre-op  { "pre_op_checklist": { "npo_confirmed": true, "site_marked": true, "consent_verified": true, "pre_med_given": true } }

// 7. SURGERY
POST /ipd/ot-sessions/ots-31/start     {}                                        → IN_PROGRESS
POST /ipd/ot-sessions/ots-31/complete  { "intra_op_notes": "…", "anaesthesia_type": "GA" }  → POST_OP

// 8. TRANSFER OUT (nurse) — OT freed, patient back to ward
POST /ipd/ot-sessions/ots-31/transfer-out
{ "transferred_to_ward_id": "…", "transferred_to_bed_id": "…", "recovery_notes": "…" }       → COMPLETED

// READS
GET /opd/surgery?patient_id=&status=          // advices
GET /ipd/ot-sessions?status=&admission_id=    // cases
GET /ipd/ot-sessions/{id}                     // one case
GET /ipd/ot-cases/{id}/consents               // consents of a case
```

---

## Frontend (4 changes only)

**1. Patient detail page (doctor, OPD & IPD — same behavior)**
"Advise Surgery" button → dialog (catalogue + priority + notes) → save → dialog chains automatically:
- no bed → *"Admit patient now?"* → ward+bed picker → done
- has bed → *"Schedule OT case now?"* → room/slot/team → done
Never a dead end. Patient page then shows a **Surgery Journey card**: `Advised ✓ → Admitted ✓ → Scheduled ⏳ → Consents → PAC → Pre-Op → Surgery → Recovery` — tap opens the case.

**2. Doctor "Surgery & OT" page** — replace the 3 stacked tables with stage tabs, one row per case, ONE action button per row:
`[Advised] [Needs Bed] [To Schedule] [Pre-Surgery] [In OT] [Recovery] [Done]`

**3. Shared OT Case page** — the existing stepper, plus: 3 read-only leading chips (Advised/Admitted/Scheduled), role-gated buttons that show whose move it is (*"Waiting for nurse: pre-op checklist"*), Cancel button (till Start), consumables inside the Surgery stage.

**4. Nurse OT Suite** — delete the legacy `/ops/ot` screens; replace with **"OT Today"** board (cases by room & slot, colored by stage, "Next: ... (you)" labels) → rows open the same shared case page. Remove the nurse-mode surgery-orders sidebar page.

---

## One-line summary per role

- **Doctor:** advise → (admit) → schedule → consent+PAC → operate. All from the patient page; the app tells you the next step every time.
- **Nurse:** open OT Today → the board says which case needs your pre-op checklist / consumables / transfer-out.
- **Backend:** one create API, one case engine, one admit call — delete the rest.
