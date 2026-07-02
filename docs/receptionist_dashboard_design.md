# HMS Receptionist Dashboard Design Plan

This document outlines the visual layout, user interactions, key metrics, and API integrations for the Receptionist Dashboard in the Healthcare Management System (HMS).

---

## 1. Persona Profile & Key Goals

- **User Role**: `RECEPTIONIST`
- **Primary Objective**: Quick check-in of scheduled patients, registering new patients, managing real-time doctor consultation queues, and collecting OPD payments.
- **Critical Success Factors**: Minimum clicks per action, highly visible queue status, robust quick search (by name/phone/UHID), and instant feedback on billing status.

---

## 2. Page Layout & UI Architecture

The dashboard uses a 3-column layout structure tailored to wide-screen display environments (e.g., reception desk monitors).

```
+-----------------------------------------------------------------------------------+
|  [Header: Logo | Branch Selector | Receptionist Profile | Notification Bell]      |
+-----------------------------------------------------------------------------------+
|  [KPI Cards: Today's Registrations | Appointments | Active Queue | Collections]   |
+-----------------------------------------------------------------------------------+
|  (Left Column - 25%)  |  (Center Column - 50%)        |  (Right Column - 25%)     |
|                       |                               |                           |
|  [Quick Actions]      |  [Real-Time Queue Management] |  [OPD Cashier Checklist]  |
|  - Register Patient   |  - Doctor queues list         |  - Pending bills list     |
|  - Book Appointment   |  - Token number tracking      |  - Collected payments     |
|  - Search Patient     |  - Queue status controls      |  - Payment mode logs      |
|                       |                               |                           |
|  [Recent Search]      |  [Appointment Schedule List]  |  [Vitals Dispatch Center] |
|  - Quick search logs  |  - Scheduled patients today   |  - Vitals pending/done    |
+-----------------------------------------------------------------------------------+
```

---

## 3. Component Details & Widgets

### 3.1 Metric Overview Cards (KPIs)
A row of four cards displaying today's real-time branch metrics:
1. **Today's Registrations**: Increments on successful patient registration.
2. **Consultation Progress**: Shows `Completed Consultations / Total Appointments` (e.g., `18 / 45`).
3. **Active Queue**: Total patients currently checked in and waiting.
4. **Collections Log**: Real-time sum of collected cashier payments today (INR).

### 3.2 Quick Action Panel (Left column)
Provides instantaneous access to key pathways:
- **Register Patient Modal Launcher**: Emits `POST /patients/register`.
- **Search Bar**: Executes `GET /patients/search?q={query}` dynamically. Displays suggestions inline.
- **OPD Ticket Booker**: Opens a fast-path appointment booking wizard mapping Patient ID, Doctor ID, Department, and Appointment Type.

### 3.3 Active Consultation Queue Table (Center column)
The core component displaying the consultation queue:
- **Columns**: `Token`, `UHID`, `Patient Name`, `Assigned Doctor`, `Department`, `Queue Status`, `Actions`.
- **Queue Statuses**:
  - `Scheduled` (Orange badge)
  - `Arrived / Waiting` (Blue badge)
  - `In Consultation` (Green badge)
  - `Completed` (Gray badge)
  - `No Show` (Red badge)
- **Inline Actions**:
  - **Check In**: Marks patient as `Arrived`.
  - **Start Consultation**: Transfers patient into doctor's consultation room state.
  - **Mark No Show**: Flags patient as absent.

### 3.4 Cashier & Payment Widget (Right column)
Tracks OPD invoices and collections:
- **Pending Bills Table**: Lists patients with outstanding invoices. Shows Patient Name, Invoiced Amount, and a "Collect Payment" button.
- **Collect Payment Dialog**: Selecting a bill opens a modal to specify payment mode (`CASH`, `UPI`, `CARD`) and reference ID.

---

## 4. API & Integration Map

The dashboard binds to these backend endpoints across services:

| Widget | HTTP Action | Backend Endpoint | Service Module |
|---|---|---|---|
| Quick Search | `GET` | `/patients/search?q=...` | `patient` |
| Registration | `POST` | `/patients/register` | `patient` |
| Vitals Dispatch | `POST` | `/patients/{id}/vitals` | `patient` |
| Patient History | `GET` | `/patients/{id}/summary` | `patient` |
| Book OPD Ticket | `POST` | `/ops/appointments` | `ops` |
| Load Schedule | `GET` | `/ops/appointments?date=YYYY-MM-DD` | `ops` |
| Queue Controls | `POST` | `/ops/appointments/{id}/status` | `ops` |
| Bill Collection | `POST` | `/billing/collect` | `billing` |

---

## 5. Mockup / Design Aesthetics Guide

- **Theme**: Premium light mode by default (high visibility under bright lobby lighting) with a dark slate background header block.
- **Typography**: `Outfit` font for metrics, `Inter` for tables and text fields.
- **Interaction Micro-animations**:
  - Hovering over a queue row highlights it in soft indigo.
  - Changing token status plays a subtle green fade-in effect to draw the eye to screen updates.
  - Quick action buttons have standard border glow transforms on cursor focus.
