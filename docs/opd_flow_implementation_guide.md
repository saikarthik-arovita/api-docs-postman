# Outpatient Department (OPD) Flow Implementation Guide

This guide details the frontend and backend design architectures for implementing the outpatient journey in the Healthcare Management System (HMS), drawing workflows from [presentation_flow.md](file:///c:/Users/saika/OneDrive/Desktop/Arovita/ops-hms-ljb/docs/presentation_flow.md) (Scenarios 2 & 3).

---

## ── SECTION 1: FRONTEND DESIGNER GUIDE ──
*Focuses on UI components, design aesthetics, user pathways, and real-time state feedback. No backend API signatures are listed here.*

### 1.1 User Pathways & Lobby Workflows

```
 [Patient Arrives] ──► [Search / Register] ──► [Select Doctor / Dept] ──► [Queue Check-in] ──► [Clinical Assessment]
```

1. **Reception Search & Profile Loading**:
   - The lobby receptionist needs a single search bar that acts as a search shortcut (matching name, phone digits, or UHID).
   - If a record is found, populate a slide-out profile card displaying basic demographics.
   - If no record is found, display a prominent "Quick Register" action, prompting for Name, Phone, and Age.

2. **Booking & OPD Ticket Wizard**:
   - Step-by-step modal displaying available doctors categorized by department.
   - Displays real-time wait counts next to each doctor's name (e.g., *Dr. Ramesh (Colorectal) — 3 waiting*).
   - Upon confirming the ticket, show a visual preview of the OPD Token (e.g., `OPD-042`) with print options.

3. **Live Queue Board (Receptionist View)**:
   - Tabbed view showing:
     - `Scheduled / Booked`: Patient appointments scheduled for the day.
     - `Arrived / Waiting`: Checked-in lobby patients waiting to see the doctor.
     - `In Consultation`: Patients currently in the consultation rooms.
     - `Completed`: Outpatients cleared for billing.

4. **Doctor Consultation Workspace**:
   - Split-screen workspace designed to minimize clinical cognitive load.
   - **Left Panel (Read-only)**: Patient profile summary, active allergies (highlighted in red alert badges), chronic conditions, and nurse vital charts.
   - **Right Panel (Actionable)**: Textarea inputs for subjective chief complaints, diagnosis, and prescription builder.
   - **Prescription Builder**: Typeahead medicine search with instant row additions. Each row includes input slots for dosage, frequency (OD, BD, etc.), and duration.

### 1.2 Design Aesthetics & State Behaviors

- **Harmonious Palette**: Light Mode lobby theme. Clear white backgrounds (`#FFFFFF`), neutral slate table rows (`#F8FAFC`), and deep indigo primary actions (`#4F46E5`).
- **Interactive State Indicators**:
  - `Scheduled` status: Muted amber label.
  - `Arrived / Waiting` status: Vibrant blue label with pulse animation indicating lobby presence.
  - `In Consultation` status: Emerald label indicating the patient is currently in the room.
  - `Completed` status: Soft gray label indicating consultation is done.
- **Micro-interactions**:
  - Dragging appointments to change priorities should animate with elastic transitions.
  - Success validation of critical vitals shows a subtle check icon next to fields.

---

## ── SECTION 2: BACKEND DEVELOPER GUIDE ──
*Focuses on database tables, RLS context settings, API routing definitions, business rules validation, and transaction boundaries.*

### 2.1 Database Entities & Relationships

All tables are branch-aware (isolated using Row Level Security) and reside within their respective schemas:

```sql
-- patient.patients: Demographics storage
-- scheduling.appointments: Scheduling mapping
-- clinical.opd_visits: Receptionist ticket metadata
-- clinical.consultations: Clinical notes
-- pharmacy.prescriptions: E-prescription tracking
```

#### Row Level Security (RLS) Rule
Each table must enforce the tenant isolation policy checking `branch_id`:
```sql
ALTER TABLE patient.patients ENABLE ROW LEVEL SECURITY;
CREATE POLICY tenant_isolation ON patient.patients
    USING (branch_id = current_setting('app.branch_id', true)::uuid);
```

### 2.2 API Endpoint & Routing Schema

#### A. Patient Registration & Search
- `POST /patients/register`
  - **Permission**: `patients:create`
  - **Inputs**: `RegisterPatientRequest` Pydantic model (requires `full_name`, `phone`, `age`, strips whitespaces).
  - **Logic**:
    1. Check `_repo.phone_exists(conn, phone)`. If exists, raise `ValidationError("phone already exists", 400)`.
    2. Read current year and execute:
       ```sql
       SELECT uhid FROM patient.patients WHERE uhid LIKE 'PAT-YYYY-%%' ORDER BY uhid DESC LIMIT 1
       ```
    3. Generate next sequential ID `PAT-YYYY-XXXX`.
    4. Call `public.fn_encrypt_field` to populate `email_encrypted` / `phone_encrypted`.

- `GET /patients/search?q={query_str}&page=1&page_size=20`
  - **Permission**: `patients:search`
  - **Logic**: Executes pattern match query on name, phone, or UHID.

#### B. Appointment Check-In & Queue Status
- `POST /ops/appointments`
  - **Permission**: `appointments:book`
  - **Logic**: Inserts into `scheduling.appointments` returning token.

- `POST /ops/appointments/{id}/check-in`
  - **Permission**: `appointments:edit`
  - **Logic**:
    1. Transaction start.
    2. Update `scheduling.appointments.status = 'ARRIVED'`.
    3. Insert into `clinical.opd_visits` with state `'SCHEDULED'` and generate `visit_number`.

#### C. Clinical Consultation & Outcome
- `POST /opd/visits/{id}/start`
  - **Permission**: `appointments:edit`
  - **Inputs**: `{ "chief_complaint": "str" }`
  - **Logic**: Transitions `clinical.opd_visits.status = 'IN_PROGRESS'` and initializes the consultation status to `'DRAFT'`.

- `POST /opd/visits/{id}/complete`
  - **Permission**: `appointments:edit`
  - **Inputs**: `{ "decision": "str", "diagnosis": "str", "follow_up_notes": "str", "follow_up_date": "date" }`
  - **Logic**:
    1. Open transaction block.
    2. Transition patient/visit status to `COMPLETED` based on final clinical decision.
    3. Update `clinical.opd_visits.status = 'COMPLETED'` and `scheduling.appointments.status = 'COMPLETED'`.
    4. Write auditable row into `audit.events` tracking doctor actor ID.
    5. Commit transaction.

### 2.3 Business Logic & Data Constraints

1. **State Transition Guards**:
   - An appointment cannot transition to `ARRIVED` unless its previous state was `SCHEDULED`.
   - A visit cannot transition to `COMPLETED` unless it was `IN_PROGRESS` with an active consultation record populated.

2. **Decoupled Billing Lock**:
   - Invoices are locked until the appointment/visit status matches `'COMPLETED'` in the `clinical` schema to prevent receipt mismatches on diagnostic/consultation adjustments.
