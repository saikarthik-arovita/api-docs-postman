# HMS — Screen Documentation

**Product:** Arovita Hospital Management System (HMS)  
**Version:** 1.0 (LJB Deployment)  
**Last Updated:** 2026-07-02  
**Prepared By:** Arovita Engineering Team  

---

## Table of Contents

1. [System Overview](#1-system-overview)
2. [Admin Module Screens](#2-admin-module-screens)
   - 2.1 [Admin Dashboard (Summary View)](#21-admin-dashboard-summary-view)
   - 2.2 [Staff Management Screen](#22-staff-management-screen)
   - 2.3 [Patient & TPA Management Screen](#23-patient--tpa-management-screen)
   - 2.4 [Compliance & Audit Log Screen](#24-compliance--audit-log-screen)
   - 2.5 [Settings & Configuration Screen](#25-settings--configuration-screen)
3. [OPD Module Screens](#3-opd-module-screens)
   - 3.1 [OPD Dashboard](#31-opd-dashboard)
   - 3.2 [OPD Patient Consultation View](#32-opd-patient-consultation-view)
4. [IPD Module Screens](#4-ipd-module-screens)
   - 4.1 [IPD Dashboard](#41-ipd-dashboard)
   - 4.2 [IPD Bed Management View](#42-ipd-bed-management-view)
5. [Pharmacy Module Screens](#5-pharmacy-module-screens)
   - 5.1 [Pharmacy Dashboard](#51-pharmacy-dashboard)
   - 5.2 [Pharmacy Staff Dashboard](#52-pharmacy-staff-dashboard)
6. [Lab & Diagnostics Module Screens](#6-lab--diagnostics-module-screens)
   - 6.1 [Lab Dashboard](#61-lab-dashboard)
   - 6.2 [Diagnostics Dashboard](#62-diagnostics-dashboard)
7. [Nursing Module Screens](#7-nursing-module-screens)
   - 7.1 [Nursing Dashboard](#71-nursing-dashboard)
   - 7.2 [Smart Alerts Panel](#72-smart-alerts-panel)
8. [Receptionist Module Screens](#8-receptionist-module-screens)
   - 8.1 [Receptionist Dashboard](#81-receptionist-dashboard)
9. [Authentication Screens](#9-authentication-screens)
   - 9.1 [Login Screen](#91-login-screen)
   - 9.2 [MFA OTP Verification Screen](#92-mfa-otp-verification-screen)
10. [Screen-to-API Mapping](#10-screen-to-api-mapping)

---

## 1. System Overview

The Arovita HMS is a multi-tenant, role-based hospital management system deployed on AWS. The frontend is organized into role-specific dashboards, each surfacing relevant patient, operational, and administrative data.

### Role-to-Screen Matrix

| Role | Primary Screen | Access Level |
|---|---|---|
| `SYSTEM_ADMINISTRATOR` | Admin Dashboard | Full system access across all branches |
| `HOSPITAL_ADMIN` | Admin Dashboard (branch-scoped) | Branch-level admin access |
| `RECEPTIONIST` | Receptionist Dashboard | OPD/IPD registration, appointments |
| `DOCTOR` | OPD Consultation / IPD Ward View | Patient clinical data |
| `NURSE` / `HEAD_NURSE` | Nursing Dashboard | Vitals, NEWS scores, ward tasks |
| `PHARMACIST` / `PHARMACY_MANAGER` | Pharmacy Dashboard | Prescriptions, inventory |
| `LAB_TECHNICIAN` / `LAB_ADMIN` | Lab Dashboard | Orders, samples, reports |
| `RADIOLOGIST` | Diagnostics Dashboard | Radiology orders and reports |
| `BILLING_EXECUTIVE` | Billing Dashboard | Invoice generation, payments |

---

## 2. Admin Module Screens

### 2.1 Admin Dashboard (Summary View)

**Route/URL:** `/admin/dashboard`  
**Required Role:** `SYSTEM_ADMINISTRATOR` or `HOSPITAL_ADMIN`

**Purpose:**  
Central command-center view for hospital administrators. Provides a real-time, high-level overview of hospital operations across staff, patients, departments, and capacity.

**Screen Sections:**

| Section | Description | API Endpoint |
|---|---|---|
| **KPI Cards Row** | Total staff (active/inactive), total patients, total departments, total roles | `GET /admin/dashboard/summary` |
| **Staff Breakdown** | Donut chart showing staff distribution by department | `GET /admin/dashboard/staff-breakdown` |
| **Leave Summary** | Pending / approved / rejected leave requests count | `GET /admin/dashboard/leave-summary` |
| **Bed Capacity Overview** | Occupied vs. available beds per ward/unit | `GET /admin/dashboard/bed-capacity` |
| **Department Activity Table** | Per-department: staff count, patient load, utilization % | `GET /admin/dashboard/departments` |
| **Alerts Banner** | Active clinical alerts flagged by smart alert engine | `GET /admin/alerts/scan` |

**Key UI Elements:**
- Top navigation bar with branch selector (multi-branch sysadmin can switch context)
- Left sidebar with module navigation links
- KPI metric cards with trend arrows and color coding (green/amber/red)
- Donut chart for staff role distribution
- Progress bars for bed occupancy per ward
- Recent activity feed at the bottom

**User Actions Available:**
- Switch branch context from branch dropdown
- Click department row to drill down to department detail
- Click "View Alerts" to navigate to smart alerts panel
- Click KPI card to view full list (e.g., all inactive staff)
- Export summary as PDF/CSV

---

### 2.2 Staff Management Screen

**Route/URL:** `/admin/staff`  
**Required Role:** `SYSTEM_ADMINISTRATOR` or `HOSPITAL_ADMIN`

**Purpose:**  
Full lifecycle management of hospital staff — onboarding, role assignment, activation, deactivation, and leave management.

**Screen Sections:**

| Section | Description | API Endpoint |
|---|---|---|
| **Staff Table** | Paginated list of all staff with name, role, department, status, branch | `GET /admin/staff` |
| **Filter Bar** | Filter by role, department, branch, activation state | Query params on `GET /admin/staff` |
| **Staff Detail Panel** | Right-side drawer showing full staff profile when row is selected | `GET /admin/staff/{staff_id}` |
| **Leave Requests Tab** | Sub-table of leave requests for the selected staff member | `GET /admin/leaves` |
| **Onboarding Queue** | List of staff pending activation approval | `GET /admin/staff?onboarding_status=PENDING` |

**Key UI Elements:**
- Searchable, sortable data table: Name, Employee ID, Role, Department, Branch, Status
- Status badge chips (ACTIVE / PENDING / INACTIVE / REJECTED) with color coding
- Inline action buttons: Approve, Reject, Deactivate, View Profile
- Leave calendar mini-view within staff detail drawer
- Bulk action selector for mass-approve or export

**User Actions Available:**
- Search staff by name, email, or employee ID
- Approve or reject pending onboarding requests
- Activate / deactivate a staff account
- View and manage leave requests

**API Interactions:**

| Action | Method | Endpoint |
|---|---|---|
| List staff | `GET` | `/admin/staff` |
| Approve leave | `PUT` | `/admin/leaves/{leave_id}/approve` |
| Reject leave | `PUT` | `/admin/leaves/{leave_id}/reject` |
| Deactivate staff | `PUT` | `/admin/staff/{staff_id}/deactivate` |

---

### 2.3 Patient & TPA Management Screen

**Route/URL:** `/admin/tpa`  
**Required Role:** `SYSTEM_ADMINISTRATOR` or `HOSPITAL_ADMIN`

**Purpose:**  
Manage Third-Party Administrator (TPA) insurance providers and track patient TPA claim statuses.

**Screen Sections:**

| Section | Description | API Endpoint |
|---|---|---|
| **TPA Provider List** | All registered TPA/insurance companies with contact and status | `GET /admin/tpa/providers` |
| **Claim Pipeline View** | Tabular view of all patient TPA claims by status | `GET /admin/tpa/reviews` |
| **Claim Detail Drawer** | Per-claim breakdown: patient UHID, admission date, claim amount | `GET /admin/tpa/reviews/{review_id}` |
| **TPA Summary Cards** | Total claims, pending amount, approved amount, rejection rate | Aggregated |

**Key UI Elements:**
- TPA provider cards with insurer name, contact info, and active/inactive badge
- Status pipeline: Submitted > Under Review > Approved / Rejected
- Amount fields with INR currency formatting
- Action buttons: Approve Claim, Reject Claim, Request More Info

**API Interactions:**

| Action | Method | Endpoint |
|---|---|---|
| List providers | `GET` | `/admin/tpa/providers` |
| Create provider | `POST` | `/admin/tpa/providers` |
| List reviews | `GET` | `/admin/tpa/reviews` |
| Approve review | `PUT` | `/admin/tpa/reviews/{review_id}/approve` |
| Reject review | `PUT` | `/admin/tpa/reviews/{review_id}/reject` |

---

### 2.4 Compliance & Audit Log Screen

**Route/URL:** `/admin/compliance`  
**Required Role:** `SYSTEM_ADMINISTRATOR`

**Purpose:**  
View immutable audit trail of all system events — logins, data edits, approvals, and configuration changes. Used for regulatory compliance and security review.

**Screen Sections:**

| Section | Description | API Endpoint |
|---|---|---|
| **Audit Event Table** | Chronological list of all system events with actor, action, timestamp, IP | `GET /admin/compliance/audit-logs` |
| **Filter Panel** | Filter by actor, event type, date range, branch | Query params |
| **Event Detail Modal** | Full event payload: before/after state, metadata, device/IP info | Part of audit log response |
| **Export Controls** | Download filtered log as CSV or PDF | Client-side export |

**Key UI Elements:**
- Sticky-column table: Timestamp | Actor | Action | Resource | Branch | IP Address
- Color-coded event type badges (LOGIN, UPDATE, DELETE, APPROVE, REJECT)
- Date range picker
- "Sensitive" flag indicator for high-risk events (deletion, role changes)

**API Interactions:**

| Action | Method | Endpoint |
|---|---|---|
| List audit logs | `GET` | `/admin/compliance/audit-logs` |
| List regulatory reports | `GET` | `/admin/compliance/reports` |
| List incident logs | `GET` | `/admin/compliance/incidents` |

---

### 2.5 Settings & Configuration Screen

**Route/URL:** `/admin/settings`  
**Required Role:** `SYSTEM_ADMINISTRATOR`

**Purpose:**  
System-wide and branch-level configuration including alert thresholds, notification preferences, inventory rules, and feature flags.

**Screen Sections:**

| Section | Description | API Endpoint |
|---|---|---|
| **Alert Thresholds** | Configure NEWS score, vitals, and lab value thresholds | `GET/PUT /admin/settings` |
| **Notification Settings** | Enable/disable WhatsApp, SMS, email per event type | `GET/PUT /admin/settings` |
| **Inventory Rules** | Set minimum stock levels and auto-reorder quantities | `GET/PUT /admin/settings` |
| **Bulk Settings Import** | Upload a JSON config file to apply multiple settings at once | `POST /admin/settings/bulk` |

**API Interactions:**

| Action | Method | Endpoint |
|---|---|---|
| Get settings | `GET` | `/admin/settings` |
| Update single setting | `PUT` | `/admin/settings` |
| Bulk update settings | `POST` | `/admin/settings/bulk` |

---

## 3. OPD Module Screens

### 3.1 OPD Dashboard

**Route/URL:** `/opd/dashboard`  
**Required Role:** `RECEPTIONIST`, `HOSPITAL_ADMIN`, `DOCTOR`

**Purpose:**  
Overview of outpatient department activity — appointments, walk-ins, doctor availability, and queue status.

**Screen Sections:**

| Section | Description | API Endpoint |
|---|---|---|
| **Today's Appointments** | Scheduled appointments for current day | `GET /opd/appointments?date=today` |
| **Doctor Availability Grid** | Available / in consultation / on break per doctor | `GET /opd/doctors/availability` |
| **Queue Status** | Current waiting queue per doctor with estimated wait time | `GET /opd/queue` |
| **Walk-in Registration** | Quick-registration form for walk-in patients | `POST /opd/visits` |

**Key UI Elements:**
- Color-coded doctor availability badges (Available / Busy / On Break / Off)
- Queue number display with estimated wait in minutes
- Appointment status pills (Scheduled / Checked-In / Completed / Cancelled)

---

### 3.2 OPD Patient Consultation View

**Route/URL:** `/opd/consultation/{visit_id}`  
**Required Role:** `DOCTOR`

**Purpose:**  
The primary screen used by doctors during an OPD consultation to record clinical findings, prescriptions, and investigation orders.

**Screen Sections:**

| Section | Description | API Endpoint |
|---|---|---|
| **Patient Header** | UHID, name, age, gender, allergies, visit reason | `GET /opd/visits/{visit_id}` |
| **Vitals** | Temperature, BP, pulse, SpO2, weight, height | `POST /nursing/vitals` |
| **Diagnosis** | ICD-10 code search and selection | `PUT /opd/consultation/{visit_id}` |
| **Prescription** | Drug search, dosage, frequency, duration, route | `POST /opd/prescription` |
| **Lab Orders** | Order lab investigations | `POST /diagnostic/lab-orders` |
| **Radiology Orders** | Order imaging investigations | `POST /diagnostic/radiology-orders` |
| **Referral / Follow-up** | Refer or schedule follow-up appointment | `POST /opd/referral`, `POST /opd/appointments` |

---

## 4. IPD Module Screens

### 4.1 IPD Dashboard

**Route/URL:** `/ipd/dashboard`  
**Required Role:** `HOSPITAL_ADMIN`, `RECEPTIONIST`, `DOCTOR`, `NURSE`

**Purpose:**  
Real-time overview of all inpatient admissions, bed occupancy, and ward-level activity.

**Screen Sections:**

| Section | Description | API Endpoint |
|---|---|---|
| **Bed Occupancy Summary** | Total / occupied / available / maintenance per ward | `GET /ipd/dashboard/bed-summary` |
| **Active Admissions Table** | All admitted patients with UHID, ward, bed, doctor | `GET /ipd/admissions?status=ACTIVE` |
| **Discharge Pipeline** | Patients flagged for discharge today | `GET /ipd/dashboard/discharge-pipeline` |
| **Critical Patients** | Patients with elevated NEWS scores or ICU flags | `GET /admin/alerts/scan` |

**Key UI Elements:**
- Visual bed map per ward (color-coded: Occupied/Available/Blocked)
- Ward filter tabs (General / ICU / HDU / Maternity / Paediatric)
- Quick actions: Admit Patient, Transfer Bed, Initiate Discharge

---

### 4.2 IPD Bed Management View

**Route/URL:** `/ipd/beds`  
**Required Role:** `HOSPITAL_ADMIN`, `RECEPTIONIST`

**Purpose:**  
Manage individual bed assignments, transfers, and maintenance flags across all wards.

**Screen Sections:**

| Section | Description | API Endpoint |
|---|---|---|
| **Bed Grid** | Visual grid of all beds in the selected ward | `GET /ipd/beds?ward_id=...` |
| **Bed Detail Panel** | Current patient details, admission duration, doctor | `GET /ipd/beds/{bed_id}` |
| **Transfer Modal** | Transfer patient to another bed/ward | `POST /ipd/transfers` |
| **Maintenance Flag** | Mark bed as under maintenance/cleaning | `PUT /ipd/beds/{bed_id}/status` |

---

## 5. Pharmacy Module Screens

### 5.1 Pharmacy Dashboard

**Route/URL:** `/pharmacy/dashboard`  
**Required Role:** `PHARMACY_MANAGER`, `PHARMACIST`

**Purpose:**  
Overview of pharmacy operations — pending prescriptions, inventory levels, dispensing activity, and stock alerts.

**Screen Sections:**

| Section | Description | API Endpoint |
|---|---|---|
| **Pending Prescriptions** | Queue of prescriptions awaiting dispensing | `GET /pharmacy/prescriptions?status=PENDING` |
| **Low Stock Alerts** | Drugs below minimum stock level | `GET /pharmacy/inventory?low_stock=true` |
| **Expiry Alerts** | Drugs expiring within 30/60/90 days | `GET /pharmacy/inventory?expiry_warning=true` |
| **Procurement Queue** | Pending purchase orders from suppliers | `GET /pharmacy/procurement/orders?status=PENDING` |
| **Sales/Revenue Chart** | Daily/weekly/monthly pharmacy sales trend | `GET /pharmacy/dashboard/sales` |

**Key UI Elements:**
- Drug inventory table with stock level progress bars (Red/Amber/Green)
- Expiry date column with countdown labels
- One-click Dispense action per prescription item

---

### 5.2 Pharmacy Staff Dashboard

**Route/URL:** `/pharmacy/staff-dashboard`  
**Required Role:** `PHARMACIST`

**Purpose:**  
Individual pharmacist's work queue — shows only the prescriptions assigned to them for dispensing. Scoped to the logged-in pharmacist rather than all pharmacy operations.

---

## 6. Lab & Diagnostics Module Screens

### 6.1 Lab Dashboard

**Route/URL:** `/lab/dashboard`  
**Required Role:** `LAB_ADMIN`, `LAB_TECHNICIAN`, `SAMPLE_COLLECTOR`

**Purpose:**  
End-to-end lab workflow: order receipt, sample collection, processing, result entry, and report release.

**Screen Sections:**

| Section | Description | API Endpoint |
|---|---|---|
| **Pending Orders** | Lab orders awaiting sample collection | `GET /diagnostic/lab-orders?status=PENDING` |
| **Sample Collection Queue** | Orders where sample is ready to be collected | `GET /diagnostic/lab-orders?status=ORDERED` |
| **Processing Queue** | Samples under analysis | `GET /diagnostic/lab-orders?status=SAMPLE_COLLECTED` |
| **Results Pending Validation** | Results awaiting validator sign-off | `GET /diagnostic/lab-orders?status=RESULT_ENTERED` |
| **Completed Reports** | Validated and released lab reports | `GET /diagnostic/lab-orders?status=REPORT_RELEASED` |

**Key UI Elements:**
- Kanban-style pipeline columns per status stage
- Urgency badge per order (ROUTINE / URGENT / STAT)
- CRITICAL value alert when result exceeds critical range
- Validator sign-off button (LAB_ADMIN / REPORT_VALIDATOR only)

---

### 6.2 Diagnostics Dashboard

**Route/URL:** `/diagnostics/dashboard`  
**Required Role:** `RADIOLOGIST`, `LAB_ADMIN`

**Purpose:**  
Radiology-specific dashboard for managing imaging orders (X-Ray, MRI, CT, Ultrasound).

**Screen Sections:**

| Section | Description | API Endpoint |
|---|---|---|
| **Imaging Orders Queue** | Pending radiology orders by modality | `GET /diagnostic/radiology-orders?status=PENDING` |
| **Scheduled Scans** | Confirmed imaging appointments | `GET /diagnostic/radiology-orders?status=SCHEDULED` |
| **Report Entry** | Upload/enter radiology report and images | `PUT /diagnostic/radiology-orders/{order_id}/report` |
| **Completed Reports** | Released radiology reports | `GET /diagnostic/radiology-orders?status=COMPLETED` |

---

## 7. Nursing Module Screens

### 7.1 Nursing Dashboard

**Route/URL:** `/nursing/dashboard`  
**Required Role:** `NURSE`, `HEAD_NURSE`

**Purpose:**  
Ward-level nursing operations — patient monitoring, vitals entry, medication administration, and task management.

**Screen Sections:**

| Section | Description | API Endpoint |
|---|---|---|
| **Patient Ward List** | All patients in the nurse's assigned ward | `GET /nursing/patients?ward_id=...` |
| **Vitals Entry** | Record temperature, BP, pulse, SpO2, RR, GCS | `POST /nursing/vitals` |
| **NEWS Score Panel** | Auto-calculated NEWS2 score per patient with trend | `GET /nursing/news-scores` |
| **Medication Administration** | MAR — mark drugs as given/held/refused | `PUT /nursing/mar/{mar_id}` |
| **Nursing Notes** | Free-text shift notes for a patient | `POST /nursing/notes` |
| **Task Checklist** | Per-patient nursing tasks | `GET/POST /nursing/tasks` |

**Key UI Elements:**
- Patient card grid sorted by NEWS score (highest risk first)
- NEWS score badge: Green (0-2) / Yellow (3-4) / Orange (5-6) / Red (7+)
- MAR table: drug name | dose | time | status (Given / Held / Refused)
- Shift handover summary generator

---

### 7.2 Smart Alerts Panel

**Route/URL:** `/nursing/alerts` or accessible from Admin dashboard  
**Required Role:** `NURSE`, `HEAD_NURSE`, `HOSPITAL_ADMIN`, `SYSTEM_ADMINISTRATOR`

**Purpose:**  
Real-time display of automatically generated clinical alerts based on patient vitals, lab results, and configurable thresholds.

**Alert Types:**

| Alert Category | Trigger Condition | Severity |
|---|---|---|
| High NEWS Score | NEWS >= 5 | Critical |
| Critical Lab Value | Lab result outside critical range | Critical |
| Vital Deterioration | Rapid change in BP/SpO2 over last 2 readings | High |
| Medication Due | MAR item overdue by > 30 min | Medium |
| Low Stock Alert | Drug stock below minimum threshold | Medium |

**Key UI Elements:**
- Alert feed sorted by severity (Critical first)
- Per-alert: patient name, UHID, ward, alert type, timestamp, current value vs. threshold
- Acknowledge and Escalate buttons per alert
- Auto-refresh every 60 seconds

---

## 8. Receptionist Module Screens

### 8.1 Receptionist Dashboard

**Route/URL:** `/reception/dashboard`  
**Required Role:** `RECEPTIONIST`

**Purpose:**  
Primary screen for front-desk operations — patient registration, appointment management, OPD token management, and IPD admission intake.

**Screen Sections:**

| Section | Description | API Endpoint |
|---|---|---|
| **Patient Registration** | Register a new patient (OPD walk-in or first visit) | `POST /patient/register` |
| **Appointment Booking** | Book/reschedule an appointment for a patient | `POST /opd/appointments` |
| **Today's Appointment List** | All appointments for today, filterable by doctor | `GET /opd/appointments?date=today` |
| **IPD Admission Intake** | Process a new IPD admission from referral/OPD | `POST /ipd/admissions` |
| **Patient Search** | Search existing patients by UHID, name, or phone | `GET /patient/search` |
| **Queue Display** | Current OPD queue token numbers per doctor | `GET /opd/queue` |

**Key UI Elements:**
- Quick-search patient lookup bar (top of screen)
- New Patient Registration button (prominent CTA)
- Appointment calendar/list toggle view
- IPD admission wizard (step-by-step form)
- Token slip print button

**User Actions Available:**
- Register a new patient (create UHID)
- Book, reschedule, or cancel an appointment
- Check-in a patient for their appointment
- Initiate IPD admission and assign bed
- Print token slip / appointment receipt

---

## 9. Authentication Screens

### 9.1 Login Screen

**Route/URL:** `/login`  
**Required Role:** Public (unauthenticated)

**Purpose:**  
Entry point for all users. Supports login via email, phone number (any Indian format), or employee ID.

**Screen Elements:**
- HMS logo and hospital branding header
- Username input field (accepts email / phone / employee ID)
- Password input field (with show/hide toggle)
- "Sign In" primary button
- "Forgot Password?" link

**Login Flow:**
1. User enters username + password
2. System authenticates against AWS Cognito
3. On success, system dispatches OTP via WhatsApp to registered phone
4. User is redirected to MFA OTP screen

**API Interaction:**
```
POST /auth/login
{
  "username": "manojkumarpatil2103@gmail.com",
  "password": "Arovita@2026",
  "device_info": "Postman Runner"
}
```

**Response — MFA Required:**
```json
{
  "success": true,
  "data": {
    "challenge": "MFA_OTP_REQUIRED",
    "phone": "****5186",
    "username": "manojkumarpatil2103@gmail.com"
  }
}
```

---

### 9.2 MFA OTP Verification Screen

**Route/URL:** `/login/mfa`  
**Required Role:** Public (in-progress authentication)

**Purpose:**  
Second factor verification. After successful password authentication, the user must enter the 6-digit OTP sent to their registered phone via WhatsApp.

**Screen Elements:**
- Masked phone number display (e.g., OTP sent to ****5186)
- 6-digit OTP input (individual digit boxes)
- "Verify OTP" button
- "Resend OTP" link (rate-limited, available after 60 seconds)
- OTP expiry countdown timer (5 minutes)

**OTP Verification Flow:**
1. User enters 6-digit OTP from WhatsApp
2. System validates OTP against DynamoDB (user_otp table)
3. On success, system issues access_token, refresh_token, and id_token
4. User is redirected to their role-specific dashboard

**API Interaction:**
```
POST /auth/login/mfa/verify
{
  "username": "manojkumarpatil2103@gmail.com",
  "otp": "123456"
}
```

---

## 10. Screen-to-API Mapping

| Screen | Primary APIs |
|---|---|
| Admin Dashboard | `GET /admin/dashboard/summary`, `GET /admin/dashboard/bed-capacity` |
| Staff Management | `GET /admin/staff`, `PUT /admin/leaves/{id}/approve` |
| TPA Management | `GET /admin/tpa/providers`, `GET /admin/tpa/reviews` |
| Compliance & Audit | `GET /admin/compliance/audit-logs` |
| Settings | `GET /admin/settings`, `POST /admin/settings/bulk` |
| OPD Dashboard | `GET /opd/appointments`, `GET /opd/queue` |
| OPD Consultation | `GET /opd/visits/{id}`, `POST /opd/prescription` |
| IPD Dashboard | `GET /ipd/admissions`, `GET /ipd/dashboard/bed-summary` |
| Bed Management | `GET /ipd/beds`, `POST /ipd/transfers` |
| Pharmacy Dashboard | `GET /pharmacy/prescriptions`, `GET /pharmacy/inventory` |
| Lab Dashboard | `GET /diagnostic/lab-orders`, results entry endpoints |
| Diagnostics | `GET /diagnostic/radiology-orders` |
| Nursing Dashboard | `GET /nursing/patients`, `POST /nursing/vitals`, `GET /nursing/news-scores` |
| Smart Alerts | `GET /admin/alerts/scan` |
| Receptionist Dashboard | `GET /opd/appointments`, `POST /patient/register`, `POST /ipd/admissions` |
| Login Screen | `POST /auth/login` |
| MFA Verification | `POST /auth/login/mfa/verify` |

---

*Document maintained by Arovita Engineering. For API payload specifications, refer to [admin_api_docs.md](./admin_api_docs.md) and [auth_api_docs.md](./auth_api_docs.md).*
