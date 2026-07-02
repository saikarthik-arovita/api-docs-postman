# UI/UX Design Specification: Pharmacy Staff Dashboard

This document details the visual, structural, and interaction design requirements for the **Pharmacy Staff Dashboard**. Its goal is to provide the product design team with a clear blueprint to build a high-fidelity design system and screen mockups that guarantee a fast, structured, low-friction, and visually premium experience for pharmacists and pharmacy assistants.

---

## 1. Design System & Aesthetics

The dashboard must feel modern, clean, and highly professional. To achieve a premium aesthetic, designers should follow these guidelines:

### Typography
* **Primary Font**: `Outfit` (for headers, metrics, and navigation).
* **Secondary Font**: `Inter` (for body text, tables, forms, and alerts).
* **Font Weights**: Light (`300`), Regular (`400`), Medium (`500`), Semi-Bold (`600`), Bold (`700`).

### Color Palette (HSL Tailored)
Avoid generic primary colors. Use a curated, dark-themed palette to reduce eye strain during long shifts:
* **Dark Background (Main)**: `hsl(224, 71%, 4%)` (Deep slate navy)
* **Panel/Card Background**: `hsla(224, 71%, 8%, 0.7)` (With glassmorphism border and blur)
* **Text (Primary)**: `hsl(210, 40%, 98%)` (Crisp off-white)
* **Text (Muted)**: `hsl(215, 20%, 65%)` (Cool slate gray)
* **Brand Accent (Active)**: `hsl(180, 100%, 42%)` (Vibrant Cyan/Teal)
* **Success/In-Stock**: `hsl(145, 80%, 40%)` (Deep emerald green)
* **Warning/Expiry**: `hsl(35, 100%, 55%)` (Warm amber orange)
* **Danger/Out-of-Stock**: `hsl(0, 95%, 60%)` (Soft coral red)

### Glassmorphism & UI Style
* **Borders**: `1px solid hsla(210, 40%, 98%, 0.1)` (Thin, semi-transparent borders for cards).
* **Shadows**: Soft, diffused shadows using active accents (`box-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.37)`).
* **Blur**: `backdrop-filter: blur(12px)` for all active modal and card overlays.
* **Border Radius**: standard `12px` for cards/inputs, `8px` for buttons, and `20px` for pills/tags.

---

## 2. Layout Structure (The Shell)

The application shell is divided into a three-column sidebar-focused dashboard:

```
+-----------------------------------------------------------------------------------+
|  [Logo] PHARMACY   |  [Search Patient/Prescription...]    (Notifications) [Profile] |
|--------------------+--------------------------------------------------------------|
|  [ ] Dashboard     |                                                              |
|  [ ] Dispensing    |                                                              |
|  [ ] Billing       |                                                              |
|  [ ] Inventory     |                      MAIN CONTENT AREA                       |
|  [ ] Returns       |                        (DYNAMIC TABS)                        |
|  [ ] Patient Help  |                                                              |
|  [ ] Shift Logs    |                                                              |
|  [ ] Reports       |                                                              |
+--------------------+--------------------------------------------------------------+
```

### Global Header
* **Global Search**: Floating center bar supporting autocompleting queries (searches Patient name, Prescription ID, or Medicine name).
* **Quick Stats/Shift**: Active shift badge (e.g., `Shift: Morning (08:00 - 16:00)`) with a quick-switch dropdown.
* **Notification Bell**: Interactive icon. Clicking opens a dropdown showing real-time operational alerts (Critical: Out of stock; Warning: Expiry within 15 days).
* **Pharmacist Profile**: Shows photo, name (`Priya Sharma`), and role (`Pharmacist`). Clicking triggers the logout and settings options.

---

## 3. Tab Requirements & Screen Specifications

### Tab 1: Dashboard Overview (Main Home Screen)
Provides a high-level summary of daily operations.
1. **Summary Metrics (5 Grid Cards)**:
   * **Pending Prescriptions**: count, active queue link.
   * **Pending Dispensing**: count of partially dispensed items.
   * **Bills Generated**: total invoices printed today.
   * **Today's Sales**: currency figure (`₹48,650`) with progress indicator against targets.
   * **Cash Collected**: split details (Cash vs Card vs UPI).
2. **Prescription Queue (Table)**:
   * Displays the 5 most recent pending prescriptions.
   * Columns: Patient Name, Doctor, Date/Time, Status (Pending, Partial, Completed), Action (Quick Dispense button).
3. **Quick Actions Panel (3x3 Grid)**:
   * Circular cards with distinct iconography:
     * *Search Prescription*, *Dispense Medicine*, *Generate Bill*, *Search Medicine*, *Print Label*, *Stock Availability*, *Return Medicine*, *Patient Guidance*, *Daily Sales Entry*.
4. **Alerts & Notifications Card**:
   * Lists critical warnings.
   * Example: `Out of Stock: Amoxicillin 500mg (Strip) - 10:20 AM`.
5. **Analytics Column**:
   * **Inventory Snapshot**: Doughnut chart splitting inventory state: In Stock (Green), Low Stock (Orange), Out of Stock (Red).
   * **Today's Task Summary**: Horizontal stack progress bars (e.g., "30 of 42 Prescriptions Dispensed").
   * **Daily Sales Summary**: Smooth sparkline chart showing sales fluctuations over the 24-hour cycle.

### Tab 2: Inventory Access
Allows staff to quickly audit available medicines.
* **Filters**: Fast-acting text filter matching:
  * Medicine name (e.g., "Paracetamol")
  * Batch Number (e.g., "BAT-2026-99")
  * Barcode SKU (e.g., "890104300122")
* **Stock Grid**: Interactive table displaying:
  * Medicine Details (Image, Name, Category).
  * Batch Details (Batch #, Expiry Date, Supplier).
  * Quantities (In Stock count, Reorder Threshold).
  * Status Badge: `In Stock` (Emerald green), `Low Stock` (Amber, count < 100), `Out of Stock` (Coral red, count = 0).
* **Simulated Barcode Scan Mode**:
  * Action button "Simulate Scanner" -> launches a camera-shaped box with a red glowing scanning line moving vertically. Scanning simulates matching a barcode and automatically opening the stock detail card.

### Tab 3: Return & Exchange
A dedicated interface for returns processing.
* **Flow 1: Invoice Validation**:
  * Input for Invoice/Bill ID.
  * Searching returns the original invoice list containing medicines, purchase dates, and prices.
* **Flow 2: Return Items Selection**:
  * Checkboxes next to purchased medicines.
  * Input fields for return quantity (cannot exceed purchase quantity).
  * Radio options: `Return (Refund)` vs `Exchange (Replace)`.
* **Flow 3: Exchange Matching**:
  * If "Exchange" is chosen, reveals a search panel to find the replacement medicine, checking real-time stock availability of the replacement batch.
* **Return Reasons (Dropdown)**:
  * `Damaged Stock`, `Wrong Dosage Prescribed`, `Patient Discharged (Excess)`, `Expired`.
* **Approval Override Screen**:
  * For high-value returns (e.g., value > ₹5,000), show an overlay requesting a Manager override PIN (with visual password dots).

### Tab 4: Patient Assistance
Supports clear communication and adherence tracking.
* **Search Patient**: Resolves active profile.
* **Dosage Card Builder**:
  * Select a prescribed medicine -> generates a visual schedule grid:
    * Morning (Sunrise icon) | Noon (Sun icon) | Evening (Sunset icon) | Night (Moon icon)
  * Frequency codes: `OD` (Once daily), `BD` (Twice daily), `TID` (Three times), `QID` (Four times).
  * Food association tags: `Before Food` (Empty plate), `After Food` (Plate & fork).
* **Refill Reminders Interface**:
  * Dynamic calendar selection for scheduling next reminder.
  * Template messages dropdown (e.g., "Refill Reminder: Your prescription for Metformin expires in 3 days. Press reply to confirm refill.").
  * Trigger button "Send Refill SMS" which updates state to `Refill Scheduled`.

### Tab 5: Daily Operations
Maintains shift-level financial audits and handovers.
* **Shift Handover Form**:
  * Input field: Handover Notes (Text area).
  * Financial audit field: Cash Drawer Counted (Input currency box).
  * Recipient field: "Select Incoming Pharmacist" (Dropdown of active users).
  * Trigger button: "Finalize Shift Handover" -> locks current metrics, generates a timestamped receipt log.
* **History Logs Table**:
  * List of previous handovers.
  * Columns: Date, Shift Type, Outgoing Pharmacist, Incoming Pharmacist, Cash Match (Match/Mismatch badge), Handover Notes.

### Tab 6: Reports
Provides analytics widgets and sales audit performance reports.
* **Reports Categories**:
  * **Sales Trends**: Graphical representation of daily billing, transaction counts, and collection splits (Cash vs Card vs UPI).
  * **Stock Health**: Insights into low-stock indicators and near-expiry batches.
  * **Returns & Exchanges**: Summary of returned inventory volumes and reasons.
* **Action Buttons**:
  * **Export PDF**: Generates A4 print-optimized report summaries.
  * **Export CSV**: Extracts raw transaction records for spreadsheet analysis.

---

## 4. Reference Specifications

For the highly detailed functional specifications matching the screen requirements, inputs, validation rules, and UX behaviors, see:
* [pharmacy_dashboard_designer_requirements.md](file:///c:/Users/saika/OneDrive/Desktop/Arovita/ops-hemorrhoids/docs/pharmacy_dashboard_designer_requirements.md)
