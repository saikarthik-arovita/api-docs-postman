# Pharmacy Staff Dashboard — Designer Requirements

This document outlines the design, layout, and UX specifications for the **Pharmacy Staff Dashboard**. It provides the product design team with a clear blueprint to build a premium, fast, structured, and low-friction experience for pharmacists, pharmacy assistants, and managers.

The dashboard is composed of 5 main modules:
1. **Inventory Access**
2. **Return & Exchange**
3. **Patient Assistance**
4. **Daily Operations**
5. **Reports**

---

## 1. Inventory Access — Designer Requirements

Need a dedicated tab/screen for auditing and checking medicine stock availability in real time. This screen supports fast search and verification before dispensing.

### Supported Flows
#### 1. Quick Stock Search & Audit
* Search medicines by name, batch number, or barcode.
* View instant stock availability, batch expiry status, and pricing details.
* Filter results by category or stock status to find alternatives.

#### 2. Barcode Scanning Integration (Simulated)
* Tap the "Scan Barcode" action button.
* Renders a camera scanning viewport overlay with a red laser guide.
* Simulates auto-identifying a barcode and opening the corresponding medicine's stock sheet.

### Required Inputs
* **Patient/Staff Search**: Alphanumeric input matching Name, Batch, or Barcode.
* **Category Filter**: Dropdown selector (Tablets, Syrups, Injections, Ointments, Inhalers).
* **Stock Status Filter**: Radio pills (All, In Stock, Low Stock, Out of Stock).

### Main Goal of Screen
This screen must feel **instant**, **visually distinct**, and **error-free**. A pharmacist should be able to verify stock levels in less than 3 seconds to answer patient queries.

### Important Logic
* **Stock Level Flagging**: Automatically calculate stock-level urgency:
  * `Out of Stock`: count = 0 (Coral red)
  * `Low Stock`: count < `min_stock_level` (Amber/Orange)
  * `In Stock`: count >= `min_stock_level` (Emerald green)
* **Real-time Synchronization**: Update stock levels immediately when other workstations dispense or return items.

### Validation Rules
* Block dispensing actions for items flagged as `Out of Stock` or `Expired`.
* Prevent manual entry of negative stock counts.

### UX Behavior (Critical)
* Auto-focus cursor on the Search bar upon entering the tab.
* Display clear warning badges next to batches expiring in less than 30 days.
* Highlight search terms matching the user's query in bold yellow text.

---

## 2. Return & Exchange — Designer Requirements

Need a secure, auditable screen for processing patient returns and exchanges of dispensed medicines.

### Supported Flows
#### 1. Invoice Validation & Return
* Search for the original invoice number.
* Display all items from the transaction (name, quantity, purchase price, total).
* Select items for return, enter return quantities, and specify reasons.
* Calculate and process refund.

#### 2. Item Exchange
* Select the return item and toggle action to "Exchange".
* Search for replacement medicine from current inventory.
* Dynamically compute price differences (refund vs new item cost) and output cash balance due/to-be-returned.

### Required Inputs
* **Invoice Reference**: Input text field (e.g. `INV-2026-9981`).
* **Item Selection**: Checkboxes next to each invoice item row.
* **Return Quantity**: Numerical stepper input (defaults to 1).
* **Action Toggle**: Radio group (`Return / Refund` vs `Exchange / Replace`).
* **Return Reason**: Dropdown (`Damaged Stock`, `Wrong Dosage Prescribed`, `Patient Discharged`, `Expired`).
* **Replacement Selector**: Search field for replacement items (only visible when "Exchange" is active).

### Main Goal of Screen
This screen must feel **precise**, **reassuring**, and **highly controlled**. It prevents inventory errors by locking return parameters to the original invoice.

### Important Logic
* **Quantity Cap**: Return quantity cannot exceed original purchase quantity minus any previously returned quantities.
* **Expiry Gate**: Returns are blocked if the purchase date is older than 14 days (except for damaged stock overrides).
* **Manager PIN Override**: High-value returns (> ₹5,000) or expired invoice exceptions block progress and display a passcode modal requiring a Manager's PIN.

### Validation Rules
* Invoice ID must be verified and exist in the system.
* Replacement item batch must be active, in stock, and non-expired.

### UX Behavior (Critical)
* Highlight return totals and balances in large, clear figures.
* Show visual warnings if returned medicines are marked as "Non-Returnable" (e.g., cold chain storage items like insulin).
* Render a floating confirmation card before locking in the database state change.

---

## 3. Patient Assistance — Designer Requirements

Need an interactive screen that helps pharmacy staff provide clinical guidance, build dosage schedule cards, and schedule refill reminders for patients.

### Supported Flows
#### 1. Dosage Schedule Card Generator
* Select a patient and select their active prescription.
* Map each medicine to a clean visual timeline grid (Morning, Noon, Evening, Night).
* Attach food association symbols (Before/After Food).
* Generate a printable patient-friendly dosage card.

#### 2. Chronic Refill Reminders
* Identify patients on recurring medications.
* Set next refill reminder schedule.
* Send automated SMS template.

### Required Inputs
* **Patient Lookup**: Search bar (MRN, Name, Phone).
* **Dosage Schedule Grid**:
  * Morning (Sunrise icon) checkbox.
  * Noon (Sun icon) checkbox.
  * Evening (Sunset icon) checkbox.
  * Night (Moon icon) checkbox.
* **Food Association**: Toggle switch (`Before Food` vs `After Food`).
* **Language Selection**: Dropdown list (English, Hindi, Telugu, Tamil, etc.).
* **Refill Reminder Date**: Calendar picker (defaults to 3 days before current supply runs out).

### Main Goal of Screen
This screen must feel **empathetic**, **highly visual**, and **patient-centric**. It bridge the gap between complex prescription codes (OD, BD, TID) and easily readable patient routines.

### Important Logic
* **Auto-Translation**: The system translates directions (e.g., "Take 1 tablet after breakfast") based on the selected patient language.
* **Refill Calculation**: Auto-calculate next refill date: `Current Date + (Prescribed Quantity / Daily Dosage) - 3 days`.

### Validation Rules
* Reminders cannot be scheduled for phone numbers that do not have active SMS status.
* Schedule dates cannot be set in the past.

### UX Behavior (Critical)
* Use intuitive color-coded icons for shift slots (Yellow for Sunrise, Orange for Sun, Purple for Sunset, Dark Blue for Moon).
* Show a real-time preview of the printed label or SMS message on the right sidebar.
* Support single-click SMS sending with a toast confirmation "Reminder Sent".

---

## 4. Daily Operations — Designer Requirements

Need a structured workspace screen for pharmacists to log handovers, count cash drawers, and manage operations during shift changes.

### Supported Flows
#### 1. Shift Handover & Reconciliation
* Start shift-end flow.
* Count and enter the physical cash remaining in the drawer.
* View system-expected cash based on invoices generated.
* Write notes for the incoming staff and submit handover.

#### 2. Operations History Review
* Browse a log of previous shift handovers to track performance history and past discrepancies.

### Required Inputs
* **Outgoing Pharmacist**: Read-only text (Auto-filled by active session).
* **Incoming Pharmacist**: Dropdown list of active pharmacy staff.
* **Physical Cash Counted**: Currency input field.
* **Handover Notes**: Large multiline text area.

### Main Goal of Screen
This screen must feel **uncompromisingly accurate**, **clear**, and **auditable**. It eliminates financial discrepancies and communication gaps between shift handovers.

### Important Logic
* **Reconciliation Check**: System checks if `counted_cash` matches `expected_cash`.
  * Status set to `MATCHED` if identical.
  * Status set to `MISMATCHED` if there is any difference.
* **System Lock**: Once a handover is finalized, the outgoing pharmacist's session is locked out of editing transaction records for that shift.

### Validation Rules
* Physical cash counted must be a valid, positive number.
* Incoming pharmacist selection is mandatory.
* Handover notes must contain a minimum of 10 characters.

### UX Behavior (Critical)
* If a cash mismatch occurs, show a high-priority amber warning box detailing the mismatch amount (surplus/deficit).
* Prevent accidental submissions with a two-step confirm slider button ("Slide to confirm handover").
* Display timestamped receipts of past handovers for transparency.

---

## 5. Reports — Designer Requirements

Need an analytical, visually engaging reports screen for managers and pharmacists to monitor daily sales metrics, top-moving items, and stock health.

### Supported Flows
#### 1. Daily Sales Summary
* Load today's sales velocity graph.
* Track total invoices, average invoice value, and split details (Cash, Card, UPI).

#### 2. Stock Health & Expiry Analytics
* Audit stock health percentages.
* View upcoming expiries and fast-moving medicine catalogues.

### Required Inputs
* **Date Range Selector**: Dropdown (Today, Yesterday, Last 7 Days, Month, Custom).
* **Report Category**: Sidebar selector (Sales Trends, Stock Audits, Returns, Shift Handover History).
* **Export Action**: Button options (`Export to PDF` / `Export to CSV`).

### Main Goal of Screen
This screen must feel **clean**, **insightful**, and **professional**. Charts should simplify pharmacy performance metrics into easily digestible visual components.

### Visual Elements (Widgets)
* **Sales Sparkline**: Line chart showing sales velocity throughout the hours of the day.
* **Inventory Doughnut**: Ring chart representing:
  * In Stock (Emerald)
  * Low Stock (Amber)
  * Out of Stock (Coral)
* **Top Dispensed Items**: Ranked horizontal bar list mapping drug names to quantity sold.
* **Returns Volume**: Small line graph comparing daily returns vs sales.

### Important Logic
* **Access Control**: Advanced reports (such as profit margins and month-over-month sales growth) are hidden from junior pharmacists and require manager authentication.
* **Dynamic Calculations**: Totals are aggregated on the fly using time-range query filters.

### Validation Rules
* Custom date ranges cannot exceed a maximum duration of 1 year.
* Start date cannot be after the current date.

### UX Behavior (Critical)
* Provide rich hover tooltips on all graph points showing the exact currency/count values.
* Render empty states with beautiful illustrations if no data is found for a custom filter.
* Ensure print-friendly stylesheets are applied so that PDF exports format correctly onto standard A4 layouts.
