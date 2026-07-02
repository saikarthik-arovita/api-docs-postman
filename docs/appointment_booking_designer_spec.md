# HMS Appointment Booking Screen — Designer Requirements & UX Specification
**Version:** 1.0 (Production Designer Spec)  
**Target User Persona:** `RECEPTIONIST` / `FRONT_DESK_EXECUTIVE` / `APPOINTMENT_COORDINATOR`  
**Core Objective:** Enable staff to complete a verified patient consultation booking in **under 10 seconds** with minimal typing and zero friction.

---

## 🎯 Design Objectives & Principles

* **Dual-Track Patient Resolution**: Seamlessly switch between fast multi-attribute search for existing patients and friction-free 2-field quick registration for new patients.
* **Instant Keyboard Navigation**: Autofocus on primary search field; logical `Tab` indexing allowing complete booking without touch/mouse dependency.
* **Contextual Slot Grid**: Dynamic, real-time availability matrix for doctor time slots with clear visual encoding (Available, Selected, Booked, Reserved).
* **Zero-Cognitive-Load Cascades**: Selecting a Department immediately filters doctors; selecting a Doctor immediately loads their active schedule and available slots.
* **Ultra-Low Typing**: Radio buttons, pre-selected defaults, and smart auto-fills minimize manual alphanumeric input.

---

## 🗺️ User Flow & State Machine

```mermaid
graph TD
    A[Screen Load: Focus on Patient Search] --> B{Patient Known?}
    B -->|Yes| C[Search Name / Phone / MRN]
    B -->|No| D[Toggle Quick Add Patient]
    C --> E[Select Patient from Instant Dropdown]
    D --> F[Enter Full Name + Phone Number]
    E --> G[Auto-fill Patient Summary Banner]
    F --> G
    G --> H[Select Department]
    H --> I[Select Doctor (Filtered by Dept)]
    I --> J[Select Date & Available Time Slot Grid]
    J --> K[Select Optional Tags: Type, Priority, Notes]
    K --> L(Click Confirm Appointment / Proceed to Billing)
```

---

## 🎴 UI Wireframes & Layout Transitions

### Screen Layout Architecture (Single Page Split-View)
The screen uses a 60/40 asynchronous split-pane layout to ensure patient context and schedule availability are always visible simultaneously.

```
+-------------------------------------------------------------------------------------------------------+
| 📅 APPOINTMENT BOOKING CONSOLE                                              [⚡ Reset Form (Esc)]      |
+-------------------------------------------------------------------------+-----------------------------+
| LEFT PANE: PATIENT RESOLUTION & APPOINTMENT DETAILS (60%)               | RIGHT PANE: SLOT GRID (40%) |
+-------------------------------------------------------------------------+-----------------------------+
| 👥 STEP 1: PATIENT RESOLUTION                                           | 👨‍⚕️ DOCTOR & SLOT AVAILABILITY|
|                                                                         |                             |
|  (•) Search Existing Patient      ( ) Quick Add New Patient             | Selected: Dr. Sarah Jenkins |
|  +------------------------------------------------------------------+  | Spec: Cardiology (Room 204) |
|  | 🔍 Type Name, Phone (+91), or MRN...                             |  | Date: Today, 29 Jun 2026    |
|  +------------------------------------------------------------------+  |                             |
|  | 💡 John Doe | MRN-9021 | +91 98765 43210 | DOB: 14/05/1988      |  | MORNING SLOTS (09:00 - 12:00)|
|  +------------------------------------------------------------------+  | [ 09:00 AM ]  [ 09:15 AM ]  |
|                                                                         | [ 09:30 AM ]* [ 09:45 AM ]# |
| ✅ Selected: John Doe (MRN-9021) | +91 98765 43210                       | [ 10:00 AM ]  [ 10:15 AM ]  |
|                                                                         |                             |
| 🩺 STEP 2: APPOINTMENT DETAILS                                          | AFTERNOON SLOTS (02:00-05:00|
|  [ Department ] *                     [ Doctor ] *                      | [ 02:00 PM ]  [ 02:15 PM ]  |
|  Cardiology (v)                       Dr. Sarah Jenkins (v)             | [ 02:30 PM ]  [ 02:45 PM ]  |
|                                                                         |                             |
|  [ Date ] *                           [ Consult Fee ]                   | Legend:                     |
|  2026-06-29 (Today) (v)               ₹ 800.00 (Auto-filled)            | [ Green ] Available         |
|                                                                         | [*Blue  ] Selected          |
| 🏷️ STEP 3: OPTIONS & TAGS                                                | [#Gray  ] Booked/Unavailable|
|  Appointment Type:   (•) OPD   ( ) Follow-up   ( ) Emergency            |                             |
|  Priority Level:     (•) Normal   ( ) High / VIP                        |                             |
|  Notes:              [ e.g., Post-op review / Chest discomfort        ] |                             |
|                                                                         |                             |
| +--------------------------------------------------------------------+  |                             |
| | 💳 [ CONFIRM & BOOK APPOINTMENT (Enter) ]                          |  |                             |
| +--------------------------------------------------------------------+  |                             |
+-------------------------------------------------------------------------+-----------------------------+
```

---

### Alternative View: Quick Add New Patient Mode
When the staff toggles to **Quick Add New Patient**, the search input dynamically shrinks into a compact 2-field form without page reload.

```
+-------------------------------------------------------------------------------------------------------+
| 👥 STEP 1: PATIENT RESOLUTION                                                                          |
|                                                                                                       |
|  ( ) Search Existing Patient      (•) Quick Add New Patient ⚡                                        |
|  +-------------------------------------+ +---------------------------------------------------+ |
|  | Full Name *                         | | Phone Number (+91) *                              | |
|  | Anita Sharma                        | | 98123 45678                                       | |
|  +-------------------------------------+ +---------------------------------------------------+ |
|  ℹ️ Minimal details mode enabled. Full registration can be completed during consultation.               |
+-------------------------------------------------------------------------------------------------------+
```

---

## 🧩 Component Specifications & Behavior

### 1. Patient Resolution Module (Dual Flow)
* **Flow 1: Existing Patient Search**:
  * **Input Type**: Predictive Auto-suggest Combobox.
  * **Search Trigger**: Triggers API lookup after typing **2 characters** or **3 digits**.
  * **Search Scope**: Matches against `MRN`, `Patient Full Name`, and `Phone Number`.
  * **Results Display**: High-density dropdown showing Avatar/Gender, Full Name, MRN badge, Mobile number, and DOB/Age.
  * **Selection State**: Upon selection, collapse input into a verified green banner with a `[Change Patient]` button.
* **Flow 2: Quick Add New Patient**:
  * **Fields Required**: Only `Full Name` (Text) and `Phone Number` (10-digit Tel).
  * **Validation**: Real-time duplicate phone number check. If phone exists, display subtle inline prompt: *"1 existing patient found with this number. [View Matches]"*.
  * **Behavior**: Temporary guest/quick record created seamlessly in background upon appointment confirmation.

### 2. Department & Doctor Cascade
* **Department Selector**: Dropdown pre-populated with active clinical departments (e.g., Cardiology, Orthopedics, General Medicine, Pediatrics).
* **Doctor Selector**:
  * Dynamically filtered based on selected Department.
  * Displays Doctor Name, Specialty Badge, Room/Cabin Number, and Consultation Fee.
  * If only 1 doctor exists in the department, auto-select that doctor immediately.

### 3. Interactive Slot Grid Matrix (Right Pane)
* **Visual Grid**: Categorized by time segments (`Morning`, `Afternoon`, `Evening`).
* **Slot Chips**: Interactive touch-friendly chips (`80px x 40px`).
* **Color States**:
  * **Emerald Green (`#10B981`)**: Available slot.
  * **Indigo Blue (`#6366F1`)**: Currently selected slot (with bold border and checkmark).
  * **Neutral Gray (`#9CA3AF`)**: Already booked / Unavailable (Disabled state with strikethrough).
  * **Amber Orange (`#F59E0B`)**: Reserved for Emergency / VIP.
* **Double-Booking Protection**: Slots are soft-locked for **120 seconds** once clicked by a staff member to prevent concurrent booking conflicts across multiple front desk terminals.

### 4. Appointment Tags & Badges
* **Appointment Type Tags**: Styled as visual radio buttons (`OPD`, `FOLLOW_UP`, `EMERGENCY`).
  * `EMERGENCY` tag automatically sets Priority to `HIGH` and highlights border in Red (`#EF4444`).
* **Priority Tags**: Toggle between `NORMAL` and `HIGH`.

---

## 📋 Field Specifications & Validation Catalog

| Field Name | Component Type | Mandatory | Default Value | Validation Rules | UX / Auto-population Behavior |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `patient_search` | Predictive Search | Conditional | Blank | Min 2 chars / 3 digits | Auto-focuses on screen load. Press `Down Arrow` to navigate results. |
| `quick_full_name` | Text Input | Conditional | Blank | Min 2 chars, Alpha + spaces | Mandatory if Quick Add mode selected. Capitalizes first letters automatically. |
| `quick_phone` | Tel Input | Conditional | Blank | Exactly 10 numeric digits | Validates Indian mobile number format (`^[6-9]\d{9}$`). Auto-formats spacing. |
| `department_id` | Dropdown | Yes | Pre-selected (General Med) | Valid Department UUID | Triggers instant fetch of linked doctors and availability calendars. |
| `doctor_id` | Dropdown | Yes | First Available Doctor | Valid Doctor UUID | Auto-selects if single doctor available. Updates consultation fee instantly. |
| `appointment_date`| Date Picker | Yes | Current Date (`Today`) | Cannot be in past | Defaults to Today. Highlights days with active doctor schedules. |
| `slot_time` | Grid Chip | Yes | None (Selectable) | Must be an AVAILABLE slot | Mandatory selection before Submit button enables. |
| `appointment_type`| Radio Pills | No | `OPD` | Allowed enum list | Modifies consultation fee dynamically if Follow-up rules apply. |
| `priority` | Toggle Pill | No | `NORMAL` | `NORMAL`, `HIGH` | Sets visual flag on doctor's daily OPD consultation queue. |
| `notes` | Text Area | No | Blank | Max 250 characters | Optional chief complaint or booking remarks. |

---

## ⚙️ Business Rules & UX Logic

### 1. Patient Validation Rule (`BR-VAL-01`)
The System shall strictly prevent appointment submission unless **EITHER** a valid existing patient is selected **OR** both Quick Add fields (`Full Name` + `Phone Number`) are validly populated.

### 2. Slot Availability & Concurrency Rule (`BR-VAL-02`)
* Slot selection executes a real-time availability check.
* If another staff member books the slot milliseconds prior to submission, the system displays a non-intrusive toast: *"Slot 09:30 AM was just booked by Terminal 2. Please select an alternate slot."* The grid refreshes automatically.

### 3. Payment Workflow Integration Options (`BR-PAY-01`)
The UI supports 2 configurable organizational payment modes (controlled via system setting `APPOINTMENT_PAYMENT_MODE`):
* **Mode A (Pay at Billing Counter / Post-Consultation)**: Clicking `Confirm Appointment` instantly issues the appointment ticket with status `CONFIRMED (UNPAID)`.
* **Mode B (Immediate Pay & Book)**: Clicking `Proceed to Pay` opens a high-speed modal overlay for cash/UPI collection. Upon payment entry, ticket is issued as `CONFIRMED (PAID)`.

---

## 🎨 Design System & Keyboard Shortcuts

### Color Palette (Tailwind Aligned)
* **Primary Actions**: Indigo 600 (`#4F46E5`)
* **Success / Available**: Emerald 500 (`#10B981`)
* **Warning / Locked**: Amber 500 (`#F59E0B`)
* **Danger / Emergency**: Red 600 (`#DC2626`)
* **Background Surface**: Slate 50 (`#F8FAFC`)

### Ergonomic Keyboard Shortcuts
* `Alt + S`: Jump focus to Patient Search / Quick Add.
* `Alt + D`: Open Department Selector.
* `Alt + N`: Toggle between Search and Quick Add mode.
* `Enter` (when slot is selected): Submit & Confirm Appointment.
* `Esc`: Clear form and reset to default state.
