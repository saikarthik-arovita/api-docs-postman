# HMS Diagnostics Module — Technical Implementation Documentation
**Version:** 1.0 (Current Codebase State)  
**Service Name:** `diagnostics` microservice (`services/diagnostics/`)  
**Database Schema:** `diagnostic` schema (PostgreSQL)

---

## 🏗️ Architecture & Component Overview

The Diagnostics service is a multi-layered microservice handling both **Pathology Laboratory** and **Radiology Imaging** operations, integrated tightly with a multi-tenant PostgreSQL database.

```
services/diagnostics/app/
├── core/                        # Business Logic & Service Engines
│   ├── service.py               # Main Diagnostic Order & Test Routing Engine
│   ├── lab_service.py           # Laboratory Test Lifecycle & Result Sign-off
│   ├── lab_inventory_service.py # Complete Reagent & Consumable Inventory Logic
│   ├── radiology_service.py     # Machine Slots, PACS Link, Critical Findings
│   ├── radiology_inventory_service.py # Radiology Films & Contrast Inventory
│   └── workflow_engine.py       # State Machine & Event Dispatcher
├── repositories/                # Data Access Layer (raw psycopg2 SQL execution)
│   ├── repo.py                  # Core Order DB queries
│   ├── lab_repo.py              # Specimen & Result DB queries
│   ├── lab_inventory_repo.py    # 17-Table Lab Inventory SQL queries
│   └── radiology_repo.py        # Modalities, Appointments, DICOM Link queries
├── routes/                      # API Endpoints & Request Routing
│   ├── router.py                # Service Entry Router
│   ├── lab_router.py            # Pathology API Endpoints
│   ├── radiology_router.py      # Imaging API Endpoints
│   ├── amendments.py            # Report Addendum & Revision API
│   ├── diagnostic_history.py    # Patient Timeline API
│   └── route_patterns.py        # Endpoint Regex Pattern Matcher
└── schemas/                     # Pydantic Request/Response Validation Schemas
    ├── schema.py                # Base Diagnostic Schemas
    ├── lab_schema.py            # Laboratory Schemas
    ├── lab_inventory_schema.py  # Inventory & Procurement Schemas
    ├── radiology_schema.py      # Radiology & Slot Schemas
    └── radiology_inventory_schema.py # Contrast & Spare Schemas
```

---

## 🛠️ Summary of Implemented Modules & Features

### 1. Core Laboratory Operations (`lab_service.py` & `lab_repo.py`)
* **Diagnostic Test Order Processing**: Supports OPD, IPD, Emergency, and Walk-in orders.
* **Specimen Collection & Accessioning**:
  * Barcode Accession Generation (`ACC-` pattern).
  * Container / Tube type tracking (`EDTA_PURPLE`, `SST_YELLOW`, `CITRATE_BLUE`, etc.).
  * Specimen Status Lifecycle (`PENDING_COLLECTION` ➡️ `SPECIMEN_COLLECTED` ➡️ `ACCESSIONED` ➡️ `IN_PROCESSING` ➡️ `COMPLETED` / `REJECTED`).
  * Specimen Rejection handling with logged reasons (Hemolyzed, Clotted, Insufficient Volume).
* **Laboratory Result Workbench**:
  * Technical result entry (numeric observed values, qualitative text).
  * Automated comparison against age/gender biological reference ranges.
  * Two-tier verification: Technical Preliminary Verification followed by Pathologist Final Clinical Sign-off.
  * Report versioning (`diagnostic_order_versions`).

---

### 2. Radiology & Imaging Operations (`radiology_service.py` & `radiology_repo.py`)
* **Modality Equipment Registry (`diagnostic.radiology_machines`)**: Tracking CT Scanners, MRI Machines, Digital X-Ray units, Ultrasounds, and Mammography units.
* **Appointment Scheduling & Slot Allocation (`diagnostic.radiology_appointments`)**: Time-slot management preventing double-booking of high-cost machines.
* **PACS & DICOM Integration**: Storing DICOM Study Instance UIDs and linking external PACS viewer endpoints.
* **Critical Findings Broadcasting (`diagnostic.radiology_critical_findings`)**: Immediate flagging of life-threatening radiological findings with acknowledgment tracking.
* **Audit Event Logging (`diagnostic.radiology_order_events`)**: Complete activity history for imaging orders.

---

### 3. Laboratory Inventory Management System (`lab_inventory_service.py` & `migrate_lab_inventory.sql`)
A complete 17-table schema and service implementation built to manage lab supplies:

* **Item Master Catalogue (`lab_item_master`)**: Centralized registry of reagents, chemicals, consumables, calibrators, controls, spares, hazard classes (GHS), and storage conditions.
* **Batch & Lot-Level Stock Tracking (`lab_inventory_batches`)**: Tracking manufacturing/expiry dates, received vs remaining quantities, and Certificate of Analysis (COA) availability.
* **Procurement Workflow (`lab_purchase_orders` & `lab_po_line_items`)**: End-to-end PO lifecycle (`DRAFT`, `APPROVED`, `ORDERED`, `RECEIVED`, `CANCELLED`).
* **Goods Received Notes (GRN) (`lab_grn_receipts` & `lab_grn_items`)**: Inspection of incoming shipments, accepted vs rejected quantities, and automatic batch generation.
* **3-Way Vendor Invoice Matching (`lab_vendor_invoices`)**: Matching PO ──────── GRN ──────── Invoice.
* **Automatic Consumption Logging (`lab_consumption_log`)**: Automated stock deduction based on test execution, manual technician adjustments, spillage, and QC runs.
* **Stock Transfers & Adjustments (`lab_stock_transfers` & `lab_stock_adjustments`)**: Managing inter-bench transfers and physical stock count reconciliations.
* **Biomedical Waste Disposal Log (`lab_disposal_log`)**: Logging expired or contaminated reagents for NABL / NABH compliance.
* **Quality Control (QC) Tracking (`lab_qc_records`)**: Monitoring daily control runs, expected vs observed values, standard deviations, and Westgard rule violations (e.g., 1-2s, 1-3s, 2-2s).
* **Reorder & Expiry Alert Queue (`lab_reorder_alerts`)**: Automatic alerts for low stock, critical minimums, near-expiry, and expired lots.
* **Internal Indent Requisitions (`lab_indent_requests` & `lab_indent_items`)**: Internal requests from lab benches to the main store.
* **MSDS Registry (`lab_msds_registry`)**: Storing Material Safety Data Sheets for hazardous chemical compliance.

---

### 4. Radiology Inventory Management (`radiology_inventory_service.py`)
* Tracking contrast media vials (Barium, Iodine, Gadolinium), radiology X-Ray films, cassettes, and machine maintenance spare parts.
* Stock deduction linked directly to radiological procedure completion.

---

### 5. Amendments & Patient History (`amendments.py` & `diagnostic_history.py`)
* **Patient Diagnostic Timeline**: Historical aggregation of all past pathology and radiology reports for a patient across admissions.
* **Report Addendums**: Formal process for attaching post-sign-off clinical notes without altering original verified diagnostic records.

---

## 📊 Database Schema Summary (`diagnostic` schema)

| Table Name | Purpose / Functionality Implemented |
| :--- | :--- |
| `diagnostic.lab_item_master` | Central catalogue of reagents, consumables, and spares. |
| `diagnostic.lab_item_test_mapping` | Linkage driving automated consumption per test. |
| `diagnostic.lab_inventory_batches` | Lot-level tracking with expiry and QC status. |
| `diagnostic.lab_purchase_orders` | Procurement PO header. |
| `diagnostic.lab_po_line_items` | PO itemized lines. |
| `diagnostic.lab_grn_receipts` | Goods received note header. |
| `diagnostic.lab_grn_items` | GRN itemized lines with inspection notes. |
| `diagnostic.lab_vendor_invoices` | 3-way matching financial records. |
| `diagnostic.lab_consumption_log` | Real-time stock usage audit trail. |
| `diagnostic.lab_stock_transfers` | Bench-to-bench stock movements. |
| `diagnostic.lab_stock_adjustments` | Physical count reconciliation entries. |
| `diagnostic.lab_disposal_log` | Biomedical waste write-off tracking. |
| `diagnostic.lab_qc_records` | QC run evaluations & Westgard rules. |
| `diagnostic.lab_reorder_alerts` | Automated low stock and expiry alert queue. |
| `diagnostic.lab_indent_requests` | Internal store requisition header. |
| `diagnostic.lab_indent_items` | Internal store requisition lines. |
| `diagnostic.lab_msds_registry` | Material Safety Data Sheets storage. |
| `diagnostic.radiology_machines` | Imaging modality equipment registry. |
| `diagnostic.radiology_appointments` | Machine slot booking calendar entries. |
| `diagnostic.radiology_critical_findings` | Life-threatening radiology alert records. |
| `diagnostic.radiology_order_events` | Imaging workflow audit events log. |
