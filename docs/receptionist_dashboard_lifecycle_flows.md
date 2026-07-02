# Receptionist Dashboard Lifecycle Flows

This document describes the operational lifecycle flows managed by the `RECEPTIONIST` user within the HMS dashboard. It details the step-by-step receptionist actions, UI state changes, and downstream clinical impacts across four primary domains: Outpatient (OPD), Inpatient (IPD), Emergency (ER), and Surgery / Operating Theatre (OT) scheduling.

---

## 1. Outpatient Department (OPD) Lifecycle Flow

```
 [Arrives] ──► [Search/Register] ──► [Book Ticket & Collect Fee] ──► [Check In to Queue] ──► [Consultation Outcome]
```

### Step 1: Reception Desk Verification
- **Receptionist Dashboard View**: Active lobby view with a centralized search bar.
- **Action**: Receptionist types the patient's name, phone, or UHID.
  - **Match Found**: System loads the patient's demographic card (address, age, blood group).
  - **No Match**: Clicking "Register Patient" opens a slide-over form. Receptionist logs `full_name`, `phone`, and `age` (dob is optional). Saving generates the unique sequential number `PAT-YYYY-XXXX`.

### Step 2: OPD Ticket Booking & Fee Collection
- **Receptionist Dashboard View**: "Book Appointment" workspace modal.
- **Action**:
  1. Selects clinical department (e.g., Gastroenterology, Colorectal).
  2. Selects available doctor (the UI lists real-time lobby queue lengths for each doctor).
  3. Collects payment (displays billing fee). Receptionist logs payment mode (`CASH`/`UPI`/`CARD`).
  4. Saving creates an appointment record in state `'SCHEDULED'` and prints a queue token ticket (e.g., `OPD-042`).

### Step 3: Queue Check-In
- **Receptionist Dashboard View**: Live Lobby Roster (`Scheduled` Tab).
- **Action**: Receptionist clicks "Check-In" when the patient is physically seated in the lobby.
  - **UI State Transition**: The row moves from the `Scheduled` tab to the `Arrived / Waiting` tab.
  - **Downstream Impact**: The patient immediately appears at the top of the assigned doctor's portal queue list as `Waiting`.

### Step 4: Outcome & Clearing
- **Receptionist Dashboard View**: Live Lobby Roster (`Completed` Tab).
- **Action**: Once the doctor signs the consultation, the visit transitions to `'COMPLETED'`.
- **Downstream Impact**: The receptionist guides the patient to the Pharmacy or Diagnostics center, as billing has cleared the invoice.

---

## 2. Inpatient Department (IPD) Lifecycle Flow

```
 [Admission Order] ──► [Bed Allocation Map] ──► [Attendant & Insurance] ──► [Stay Updates] ──► [Discharge & Release]
```

### Step 1: Admission Request Intake
- **Receptionist Dashboard View**: "IPD Admission Requests" inbox widget.
- **Action**: The receptionist clicks on an admission order recommended by an OPD or ER doctor.

### Step 2: Bed Allocation
- **Receptionist Dashboard View**: Interactive ward map.
- **Action**: Receptionist filters by ward type (General, Semi-Private, Private, ICU) and logs:
  - Selects a bed flagged as `AVAILABLE` (green).
  - Bed transitions to `RESERVED` (orange).
  - Saves the record, generating a unique `admission_number`. Bed transitions to `OCCUPIED` (red).

### Step 3: Attendant & Insurance Setup
- **Receptionist Dashboard View**: "Inpatient Onboarding Details" tab.
- **Action**:
  - **Insurance**: Captures provider, policy number, and scheme type (`ROHINI`/`TPA`/etc.) to run pre-authorization.
  - **Attendant**: Captures name, relationship, and contact numbers.

### Step 4: Ongoing Monitoring (Stay Period)
- **Receptionist Dashboard View**: Patient roster (`Admitted` Tab).
- **Action**: Receptionist tracks length of stay, monitors room transfers (e.g., ward bed to ICU bed), and logs bed transfers using "Transfer Bed" modal.

### Step 5: Discharge Clearance & Bed Release
- **Receptionist Dashboard View**: "Pending Discharges" panel.
- **Action**:
  1. Once the doctor signs the discharge summary, the receptionist checks the clearance status checklist (Billing, Pharmacy, Nursing, Insurance).
  2. Family settles the invoice.
  3. Receptionist prints the Gate Pass and clicks "Discharge Patient".
  4. **UI State Transition**: `ipd.ipd_admissions` moves to `'DISCHARGED'`. The bed returns to state `'AVAILABLE'` (green) on the interactive ward map.

---

## 3. Emergency (ER) Walk-In Lifecycle Flow

```
 [Critical Arrival] ──► [Emergency Ticket] ──► [Triage Level] ──► [NEWS Escalation] ──► [Admit or Discharge]
```

### Step 1: Emergency Registration
- **Receptionist Dashboard View**: Floating red button "New Emergency Case" accessible from any tab.
- **Action**: Receptionist enters name/gender and creates the case immediately. Bypasses standard profile requirements to prevent treatment delay.

### Step 2: Triage Level Assignment
- **Receptionist Dashboard View**: Triage allocation modal.
- **Action**: Receptionist assigns a triage level:
  - `RED` (Resuscitation)
  - `ORANGE` (Emergent)
  - `YELLOW` (Urgent)
  - `GREEN` (Non-urgent)
- **Downstream Impact**: The ER queue dashboard is re-sorted, placing the `RED`/`ORANGE` patient at the top of the queue.

### Step 3: Bed/Stretcher Allocation & Escalation
- **Receptionist Dashboard View**: ER Room Bed Map.
- **Action**: Receptionist assigns the patient to an available trauma bay bed.

---

## 4. Surgery & Operation Theatre (OT) Scheduling Flow

```
 [Surgery Order] ──► [OT Scheduler] ──► [PAC Checklist] ──► [Check In to OT] ──► [Post-Op Transfer]
```

### Step 1: Surgery Request Intake
- **Receptionist Dashboard View**: "Surgical Orders Intake" list.
- **Action**: Receptionist opens a request submitted by a consultant colorectal surgeon from the surgery catalogue.

### Step 2: Scheduling Resources
- **Receptionist Dashboard View**: Operation Theatre calendar block scheduler.
- **Action**:
  1. Drags the surgery request onto a calendar slot mapping a specific OT Room.
  2. Assigns Surgeon, Anaesthetist, and Scrub Nurse.
  3. Confirms scheduled start/end times. Status is set to `'SCHEDULED'`.

### Step 3: Pre-Op & PAC Tracking
- **Receptionist Dashboard View**: "Pre-Op Clearance Board".
- **Action**: Receptionist monitors completion checkmarks:
  - `PAC Cleared` (flagged by anaesthetist).
  - `Consent Signed` (flagged by nurse).

### Step 4: OT Check-In
- **Receptionist Dashboard View**: OT Live Monitor.
- **Action**: Receptionist clicks "Check-In to OT" when the patient is moved to the pre-op holding area.
  - **UI State Transition**: Session moves to `'PRE_OP'`.
  - **Downstream Impact**: Doctor's monitor shows status as ready for incision.
