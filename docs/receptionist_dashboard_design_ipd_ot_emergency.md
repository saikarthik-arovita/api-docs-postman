# HMS Receptionist Dashboard Design Plan — IPD, OT, and Emergency

This document details the layout structure, interactive widgets, UX flows, and API integrations for the Inpatient Department (IPD), Operating Theatre (OT), and Emergency Room (ER) views of the Receptionist Dashboard.

---

## 1. Domain-Specific Receptionist Profiles & Goals

### 🛏️ Inpatient (IPD) Desk
*   **Primary Objective**: Manage real-time bed allocations, track active ward occupancies, onboard attendants and insurance, and verify gate clearances for patient discharge.
*   **Key Goals**: Minimum clicks to allocate/transfer beds, instant visibility of bed status transitions, and a single pane to verify multi-department clearances.

### 🔪 Operating Theatre (OT) Desk
*   **Primary Objective**: Map and schedule OT rooms, track surgeon/anaesthetist availability, monitor PAC (Pre-Anaesthetic Checkup) clearances, and record transfers to recovery.
*   **Key Goals**: Drag-and-drop calendar resource scheduling, instant visual warnings for doctor/room conflicts, and clear checkbox tracking of pre-operative guidelines.

### 🚨 Emergency (ER) Desk
*   **Primary Objective**: Rapid walk-in registration, triage code assignment, trauma bed dispatching, and immediate escalation alerts.
*   **Key Goals**: Ultra-fast (sub-10-second) case creation, color-coded urgency sorting (Red/Orange/Yellow/Green), and zero obstruction to clinical treatment.

---

## 2. Page Layouts & UI Architecture

### 2.1 IPD Ward Map & Admissions Dashboard
A 3-column layout focused on spatial bed management and discharge clearance.

```
+-----------------------------------------------------------------------------------+
|  [Header: Branch Selector | Ward Statistics Overview: Total Beds | Occupied | Vacant] |
+-----------------------------------------------------------------------------------+
|  (Left Column - 25%)  |  (Center Column - 50%)        |  (Right Column - 25%)     |
|                       |                               |                           |
|  [Admission Intake]   |  [Interactive Ward Map Grid]  |  [Discharge Check Center] |
|  - Dr. recommended    |  - Tabbed: General/ICU/HDU    |  - Pending clearances     |
|    admissions inbox   |  - Visual Bed Cards           |  - Billing / Rx checklist |
|  - Walk-in ER admissions|  - Color-coded by state       |  - Print Gate Pass action |
|                       |                               |                           |
|  [Quick Actions]      |  [Bed Transfer Panel]         |  [Attendant & Insurance]  |
|  - Direct Admit Form  |  - Drag bed card to transfer  |  - Policy pre-auth logs   |
|  - Find Patient Bed   |  - History logs               |  - Attendant contact card |
+-----------------------------------------------------------------------------------+
```

### 2.2 OT Scheduling & Resource Board
A calendar-centric layout displaying OT room schedules alongside surgical booking queues.

```
+-----------------------------------------------------------------------------------+
|  [Header: Date Navigation | OT Rooms Summary (OT-01, OT-02, OT-03) | Active Cases] |
+-----------------------------------------------------------------------------------+
|  (Left Column - 25%)  |  (Center Column - 50%)        |  (Right Column - 25%)     |
|                       |                               |                           |
|  [Surgery Orders List]|  [OT Room Scheduling Grid]     |  [Pre-Op Clearance Desk]  |
|  - Pending surgeries  |  - Hourly calendar view for   |  - PAC clearance badges   |
|    ordered by doctors |    each active OT room        |  - Informed consent check  |
|  - Estimated duration |  - Click-and-drag blocks      |  - Pre-op checklist JSONB  |
|                       |  - Visual conflict indicators |                           |
|  [Quick Search]       |                               |  [Recovery & Ward Transfer]|
|  - Filter by Surgeon  |                               |  - Post-op release state  |
|  - Search by Procedure|                               |  - Target Bed Selector    |
+-----------------------------------------------------------------------------------+
```

### 2.3 Emergency Room (ER) Triage Board
A high-contrast, alert-focused board with large tactile touch points for extreme urgency.

```
+-----------------------------------------------------------------------------------+
|  [Header: Flashing Alert Bar (Active Reds) | Count: Resuscitation | Emergent | Urgent]|
+-----------------------------------------------------------------------------------+
|  (Left Column - 30%)              |  (Right Column - 70%)                         |
|                                   |                                               |
|  [TACTILE EMERGENCY INTAKE]       |  [Active Emergency Triage Board]              |
|  - Giant Red "New Case" Button    |  - List sorted by Triage Level (Red at top)   |
|  - Quick Fields: Est. Age, Gender |  - Columns: Triage Code | UHID | Bay Bed      |
|    and Chief Complaint            |  - Elapsed time tracking since arrival        |
|                                   |  - NEWS Score Indicators                      |
|  [Trauma Bay Bed Map]             |                                               |
|  - Stretcher/Bay layout           |  [Action Drawer]                              |
|  - Click to instantly assign      |  - Escalate to doctor / Dispatch vitals       |
+-----------------------------------------------------------------------------------+
```

---

## 3. Component Details & Widgets

### 3.1 Interactive Bed Grid (IPD)
*   **Visual State**: Individual bed cards color-coded based on status:
    *   `AVAILABLE` (Vibrant Emerald: `#10B981` / HSL `160, 84%, 39%`)
    *   `OCCUPIED` (Crimson: `#EF4444` / HSL `0, 84%, 60%`)
    *   `RESERVED` (Amber: `#F59E0B` / HSL `38, 92%, 50%`)
    *   `MAINTENANCE` (Slate Gray: `#64748B` / HSL `215, 16%, 47%`)
*   **Interactions**:
    *   Hovering over an occupied bed card opens a tooltip displaying: *Patient Name, UHID, Admitting Doctor, Admission Date, and Ward Number.*
    *   Clicking an available bed triggers a quick slide-over to bind a pending Admission request.
    *   Dragging an occupied bed card onto an available bed card opens the **Bed Transfer Dialog** confirming the reason for transfer.

### 3.2 Drag-and-Drop OT Resource Scheduler (OT)
*   **Visual State**: A vertical grid displaying OT rooms as columns, and hours of the day (08:00 to 20:00) as rows.
*   **Interactions**:
    *   Surgical order requests in the left column can be dragged directly onto a specific time slot under an OT room.
    *   Hovering over a scheduled slot displays surgeon name, anaesthetist name, and scrub nurse name.
    *   **Conflict Indicator**: If a surgeon is assigned to two overlapping slots in different rooms, both slots display a pulsating red warning border and an alert icon.

### 3.3 ER High-Contrast Triage Cards (Emergency)
*   **Urgency Badges**: Large pill badges with background pulses:
    *   `RED (Resuscitation)`: Pulsating Crimson background, bold white text. Indicates immediate life threat.
    *   `ORANGE (Emergent)`: Solid Dark Orange background. Target assessment time within 10 minutes.
    *   `YELLOW (Urgent)`: Solid Amber background. Target assessment within 30 minutes.
    *   `GREEN (Non-Urgent)`: Solid Blue/Gray background. Target assessment within 120 minutes.
*   **Time-Since-Arrival Counter**: Real-time timer counts minutes elapsed since registration. Triggers a flash animation if elapsed time exceeds the triage level SLA target.

---

## 4. API & Integration Map

These interfaces bind to the newly partitioned domain microservices:

| Component / Action | HTTP Method | Endpoint | Source Service |
|---|---|---|---|
| **IPD: Load Wards & Beds** | `GET` | `/ipd/wards` | `ipd` |
| **IPD: Bed Allocations** | `POST` | `/ipd/admissions` | `ipd` |
| **IPD: Bed Transfer** | `POST` | `/ipd/admissions/{id}/transfer` | `ipd` |
| **IPD: Check Clearances** | `GET` | `/ipd/admissions/{id}/discharge-clearance` | `ipd` |
| **IPD: Execute Discharge** | `POST` | `/ipd/admissions/{id}/discharge` | `ipd` |
| **OT: Load Calendar Grid** | `GET` | `/ot/sessions?date=YYYY-MM-DD` | `ot` |
| **OT: Schedule Surgery** | `POST` | `/ot/sessions` | `ot` |
| **OT: Reschedule Surgery** | `PATCH` | `/ot/sessions/{id}/reschedule` | `ot` |
| **OT: Monitor Checklist** | `GET` | `/ot/sessions/{id}/pre-op` | `ot` |
| **ER: Immediate Intake** | `POST` | `/emergency/visits` | `emergency` |
| **ER: Assign Triage Code** | `PATCH` | `/emergency/visits/{id}/triage` | `emergency` |
| **ER: Bed Dispatch** | `PATCH` | `/emergency/visits/{id}/status` | `emergency` |

---

## 5. Mockup & Design Aesthetics Guide

*   **Core Typography**: `Outfit` font for counts and triage banners; `Inter` for bed labels, scheduling tables, and form text inputs.
*   **Palette Rules**:
    *   Default UI is Light Mode with high contrast (`#FFFFFF` background, `#F1F5F9` slate cards) to reduce glare in brightly lit clinic environments.
    *   Header blocks use dark slate (`#0F172A`) to establish depth.
*   **Interactive Micro-animations**:
    *   **Alert Flashes**: Emergency triage level changes trigger a brief green or orange wash animation on the row.
    *   **Drag States**: While dragging a surgery block, potential valid slots glow in a soft indigo (`#EEF2FF` background, dashed indigo border). Valid bed cards highlight green when dragging an admission request over them.
    *   **NEWS Score Alerts**: Critical vital checks trigger a soft red pulsing circle next to the patient name on the ER board.
