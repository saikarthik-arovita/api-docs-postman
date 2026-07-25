# HMS Admin Service: Comprehensive API Catalog & Reference

This document lists and defines **every** single REST API endpoint exposed by the `hms-admin` microservice. It aggregates all base routes from [router.py](file:///c:/Users/saika/OneDrive/Desktop/Arovita/ops-hms-ljb/services/admin/app/routes/router.py) and extended routes from [admin_router_ext.py](file:///c:/Users/saika/OneDrive/Desktop/Arovita/ops-hms-ljb/services/admin/app/routes/admin_router_ext.py).

---

## Global Headers & Environment
* **Base URL**: `https://vo585bjv4a.execute-api.ap-south-1.amazonaws.com/dev`
* **Authorization**: `Bearer <Cognito_Access_Token>`
* **Branch Isolation**: Uses active user session or override via `X-Branch-ID: <UUID>` header.

---

## 1. Hospital Profile & System Settings

| Method | Endpoint | Purpose / Description | Required Permission |
| :--- | :--- | :--- | :--- |
| `GET` | `/admin/hospital-profile` | Retrieve the active branch and organization profile details. | `SYSTEM_SETTING_VIEW` |
| `POST` | `/admin/hospital-profile` | Initialize organization profile details. | `SYSTEM_SETTING_EDIT` |
| `PATCH` | `/admin/hospital-profile/{profile_id}` | Modify profile parameters (e.g. contact email, GSTIN, NABH number). | `SYSTEM_SETTING_EDIT` |
| `DELETE` | `/admin/hospital-profile/{profile_id}` | Hard delete organization/facility profile record. | `SYSTEM_SETTING_EDIT` |
| `GET` | `/admin/settings` | Retrieve system settings (key-value configs) for the branch. | `SYSTEM_SETTING_VIEW` |
| `POST` | `/admin/settings` | Set or update a single configuration setting. | `SYSTEM_SETTING_EDIT` |
| `DELETE` | `/admin/settings` | Remove a setting configuration key. | `SYSTEM_SETTING_EDIT` |
| `POST` | `/admin/settings/bulk` | Bulk upsert configuration settings array. | `SYSTEM_SETTING_EDIT` |
| `GET` | `/admin/settings/security` | Retrieve session security, password complexity, and MFA rules. | `SECURITY_SETTING_VIEW` |
| `PATCH` | `/admin/settings/security` | Update password policy or MFA requirements. | `SECURITY_SETTING_EDIT` |

---

## 2. Dashboard & Performance Analytics

### 2.1 Summary KPIs & Statistics
| Method | Endpoint | Purpose / Description | Required Permission |
| :--- | :--- | :--- | :--- |
| `GET` | `/admin/dashboard/summary` | Retrieve high-level counts (total staff, patients, departments). | `DASHBOARD_VIEW` |
| `GET` | `/admin/dashboard/overview` | Retrieve chronological aggregations of inpatient vs outpatient counts. | `DASHBOARD_VIEW` |
| `GET` | `/admin/dashboard/hospital-performance` | Fetch revenue growth, OPD consultation conversion, and IPD discharges. | `DASHBOARD_VIEW` |
| `GET` | `/admin/dashboard/resource-utilization` | Retrieve operating room, diagnostics (MRI/CT), and ICU utilization. | `DASHBOARD_VIEW` |
| `GET` | `/admin/dashboard/reports` | Retrieve list of recently generated clinical/financial reports. | `DASHBOARD_VIEW` |
| `GET` | `/admin/dashboard/statistics` | Retrieve daily, weekly, or custom statistics overview. | `DASHBOARD_VIEW` |
| `GET` | `/admin/dashboard/department-performance` | Retrieve performance metrics (average length of stay, revenue) per dept. | `DASHBOARD_VIEW` |
| `GET` | `/admin/dashboard/report-library` | List pre-configured report templates available to trigger. | `DASHBOARD_VIEW` |
| `GET` | `/admin/dashboard` | General administrative aggregate landing view data. | `DASHBOARD_VIEW` |

### 2.2 Deep Analytics Tabs
| Method | Endpoint | Purpose / Description | Required Permission |
| :--- | :--- | :--- | :--- |
| `GET` | `/admin/dashboard/analytics/patient` | Demographic analytics (gender split, age groups, patient retention). | `ANALYTICS_VIEW` |
| `GET` | `/admin/dashboard/analytics/opd` | OPD metrics (average wait time, consultation volumes by doctor). | `ANALYTICS_VIEW` |
| `GET` | `/admin/dashboard/analytics/ipd` | Inpatient trends (admissions, discharge rates, average bed occupancy). | `ANALYTICS_VIEW` |
| `GET` | `/admin/dashboard/analytics/operations` | Surgical volume analysis, surgery types, and theatre utilization. | `ANALYTICS_VIEW` |
| `GET` | `/admin/dashboard/analytics/financial` | Billing KPIs, outstanding balances, claim settlements, and cash flows. | `ANALYTICS_VIEW` |
| `GET` | `/admin/dashboard/analytics/department` | Comparative analysis of departments by volume, margin, and staff counts. | `ANALYTICS_VIEW` |
| `GET` | `/admin/dashboard/analytics/doctor` | Consultation load, clinical conversion, and patient feedback index. | `ANALYTICS_VIEW` |
| `GET` | `/admin/dashboard/analytics/master-summary` | Aggregated executive KPIs for all branches under the organization. | `ANALYTICS_VIEW` |

### 2.3 Standalone Analytics Metrics
| Method | Endpoint | Purpose / Description | Required Permission |
| :--- | :--- | :--- | :--- |
| `GET` | `/admin/analytics/kpi` | High-level analytical metrics (Average Revenue per Occupied Bed, etc.). | `ANALYTICS_VIEW` |
| `GET` | `/admin/analytics/patient-inflow` | Timeline graph data for patient visit volumes. | `ANALYTICS_VIEW` |
| `GET` | `/admin/analytics/revenue` | Revenue trend graphs grouped by department or payment mode. | `ANALYTICS_VIEW` |
| `GET` | `/admin/analytics/opd-ipd-conversion` | Ratio analysis of outpatient visits converted to admissions. | `ANALYTICS_VIEW` |
| `GET` | `/admin/analytics/doctor-performance` | Timeline comparison profiles for doctor consultation outcomes. | `ANALYTICS_VIEW` |
| `GET` | `/admin/analytics/emergency` | Emergency room traffic, triage distribution, and ambulance log counts. | `ANALYTICS_VIEW` |
| `GET` | `/admin/analytics/bed-occupancy` | Real-time and historic bed occupancy percentages by ward type. | `ANALYTICS_VIEW` |
| `GET` | `/admin/analytics/ot` | Operation theatre utilization metrics and surgical delays analysis. | `ANALYTICS_VIEW` |
| `GET` | `/admin/analytics/lab-tat` | Laboratory Turnaround Time (TAT) compliance rates. | `ANALYTICS_VIEW` |
| `GET` | `/admin/analytics/billing` | Claims rejection rate and collection period analysis. | `ANALYTICS_VIEW` |

---

## 3. Staff Profile & Permissions Management

| Method | Endpoint | Purpose / Description | Required Permission |
| :--- | :--- | :--- | :--- |
| `GET` | `/admin/staff/me` | Retrieve the active caller's staff profile and role setup. | *None* (Self) |
| `GET` | `/admin/staff` | List or search all staff profiles (doctors, nurses, admin, support). | `STAFF_VIEW` |
| `PATCH` | `/admin/staff/{user_id}/fee` | Update consultation fee details for a doctor. | `STAFF_EDIT` |
| `PATCH` | `/admin/staff/{user_id}/role` | Assign a new system role to a staff member. | `STAFF_EDIT` |
| `GET` | `/admin/staff/{user_id}/permissions` | List custom permissions explicitly assigned to this staff user. | `STAFF_VIEW` |
| `POST` | `/admin/staff/{user_id}/permissions` | Assign custom permission overrides to this user. | `STAFF_EDIT` |
| `DELETE` | `/admin/staff/{user_id}/permissions` | Revoke custom permission overrides. | `STAFF_EDIT` |
| `GET` | `/admin/roles` | List all available Roles (`SYSTEM_ADMINISTRATOR`, `DOCTOR`, `NURSE`, etc.). | `ROLE_VIEW` |
| `GET` | `/admin/permissions` | Retrieve the master list of all individual security permissions. | `ROLE_VIEW` |
| `GET` | `/admin/roles/{role_id}/permissions` | List permissions associated with a specific role. | `ROLE_VIEW` |
| `POST` | `/admin/roles/{role_id}/permissions` | Assign new permissions to a role. | `ROLE_EDIT` |
| `DELETE` | `/admin/roles/{role_id}/permissions` | Revoke permissions from a role. | `ROLE_EDIT` |

---

## 4. Attendance & Leaves Management

| Method | Endpoint | Purpose / Description | Required Permission |
| :--- | :--- | :--- | :--- |
| `GET` | `/admin/staff/me/attendance` | Retrieve the active caller's attendance logs. | *None* (Self) |
| `GET` | `/admin/staff/me/leaves` | List active caller's applied leave history. | *None* (Self) |
| `GET` | `/admin/staff/me/leaves/balance` | Fetch active caller's remaining leave counts (Casual, Sick, etc.). | *None* (Self) |
| `POST` | `/admin/staff/me/leaves/apply` | File a new leave application request. | *None* (Self) |
| `DELETE` | `/admin/staff/me/leaves/{leave_id}` | Cancel/withdraw a pending leave request. | *None* (Self) |
| `GET` | `/admin/staff/leaves/pending` | List all pending leave requests requiring admin approval. | `LEAVE_APPROVE` |
| `PATCH` | `/admin/staff/leaves/{leave_id}/approve` | Approve a staff member's leave request. | `LEAVE_APPROVE` |
| `PATCH` | `/admin/staff/leaves/{leave_id}/reject` | Reject a staff member's leave request. | `LEAVE_APPROVE` |
| `GET` | `/admin/staff/leaves/summary` | Retrieve leave summary counts for all active staff. | `LEAVE_APPROVE` |
| `GET` | `/admin/leaves/pending` | Alternate path to list all pending leave requests. | `LEAVE_APPROVE` |
| `POST` | `/admin/leaves/{leave_id}/approve` | Alternate path to approve a leave request. | `LEAVE_APPROVE` |
| `POST` | `/admin/leaves/{leave_id}/reject` | Alternate path to reject a leave request. | `LEAVE_APPROVE` |

---

## 5. Facility Spatial Hierarchy (Floors, Blocks, Wards, Beds)

### 5.1 Floors & Blocks
| Method | Endpoint | Purpose / Description | Required Permission |
| :--- | :--- | :--- | :--- |
| `GET` | `/admin/floors` | List all floors. | `BED_VIEW` |
| `POST` | `/admin/floors` | Create a new floor. | `BED_CREATE` |
| `GET` | `/admin/floors/{floor_id}` | Retrieve details of a specific floor. | `BED_VIEW` |
| `PATCH` | `/admin/floors/{floor_id}` | Update details of a floor. | `BED_EDIT` |
| `GET` | `/admin/blocks` | List all blocks. | `BED_VIEW` |
| `POST` | `/admin/blocks` | Create a new building block. | `BED_CREATE` |
| `GET` | `/admin/blocks/{block_id}` | Retrieve details of a specific block. | `BED_VIEW` |
| `PATCH` | `/admin/blocks/{block_id}` | Update details of a block. | `BED_EDIT` |

### 5.2 Wards, Units, & Beds
| Method | Endpoint | Purpose / Description | Required Permission |
| :--- | :--- | :--- | :--- |
| `GET` | `/admin/wards` | List all wards (with optional department, block, or floor filtering). | `BED_VIEW` |
| `POST` | `/admin/wards` | Create a new ward. | `BED_CREATE` |
| `GET` | `/admin/wards/{ward_id}` | Retrieve details of a specific ward. | `BED_VIEW` |
| `PATCH` | `/admin/wards/{ward_id}` | Update details of a ward. | `BED_EDIT` |
| `GET` | `/admin/wards/{ward_id}/units` | List units mapped to a specific ward. | `BED_VIEW` |
| `GET` | `/admin/units` | List all units. | `BED_VIEW` |
| `POST` | `/admin/units` | Create a new unit. | `BED_CREATE` |
| `GET` | `/admin/beds/summary` | Retrieve high-level bed metrics (capacity, occupied, available, etc.). | `BED_VIEW` |
| `GET` | `/admin/beds` | List all beds. | `BED_VIEW` |
| `POST` | `/admin/beds` | Create a new bed record. | `BED_CREATE` |
| `GET` | `/admin/beds/{bed_id}` | Retrieve details of a specific bed. | `BED_VIEW` |
| `PATCH` | `/admin/beds/{bed_id}` | Update details of a bed. | `BED_EDIT` |
| `PATCH` | `/admin/beds/{bed_id}/status` | Update operational status (e.g. `'AVAILABLE'`, `'OCCUPIED'`). | `BED_EDIT` |

---

## 6. Setup & Master Data (Departments & Services)

### 6.1 Departments & Specializations
| Method | Endpoint | Purpose / Description | Required Permission |
| :--- | :--- | :--- | :--- |
| `GET` | `/admin/departments` | List departments with staff counts, locations, and operating hours. | `STAFF_VIEW` |
| `POST` | `/admin/departments` | Create a new department. | `STAFF_EDIT` |
| `PATCH` | `/admin/departments/{department_id}` | Update department configuration. | `STAFF_EDIT` |
| `GET` | `/admin/specializations` | List all specializations. | `STAFF_VIEW` |
| `POST` | `/admin/specializations` | Create a new specialization. | `STAFF_EDIT` |
| `PATCH` | `/admin/specializations/{specialization_id}` | Update specialization configuration. | `STAFF_EDIT` |
| `GET` | `/admin/specialities/summary` | Retrieve aggregated staff and patient counts mapped by speciality. | `STAFF_VIEW` |
| `GET` | `/admin/specialities` | List all specialities. | `STAFF_VIEW` |
| `POST` | `/admin/specialities` | Create a new speciality. | `STAFF_EDIT` |
| `GET` | `/admin/specialities/{speciality_id}` | Retrieve specific speciality details. | `STAFF_VIEW` |
| `PATCH` | `/admin/specialities/{speciality_id}` | Modify speciality attributes. | `STAFF_EDIT` |
| `DELETE` | `/admin/specialities/{speciality_id}` | Delete speciality master configuration. | `STAFF_EDIT` |

### 6.2 Services & Service Catalog
| Method | Endpoint | Purpose / Description | Required Permission |
| :--- | :--- | :--- | :--- |
| `GET` | `/admin/services/summary` | Retrieve count summaries of active clinical services. | `SERVICE_VIEW` |
| `GET` | `/admin/services/staff-directory` | List all staff members mapped to active services. | `SERVICE_VIEW` |
| `GET` | `/admin/services` | List all clinical services. | `SERVICE_VIEW` |
| `POST` | `/admin/services` | Register a new clinical service. | `SERVICE_EDIT` |
| `GET` | `/admin/services/{service_id}` | Retrieve details of a specific service. | `SERVICE_VIEW` |
| `PATCH` | `/admin/services/{service_id}` | Update service configuration. | `SERVICE_EDIT` |
| `DELETE` | `/admin/services/{service_id}` | Delete a clinical service. | `SERVICE_EDIT` |
| `GET` | `/admin/service-catalogue/summary` | Retrieve summary of the organizational service catalogue. | `SERVICE_VIEW` |
| `GET` | `/admin/service-catalogue/records` | List all catalogue records (rate cards, consultations). | `SERVICE_VIEW` |
| `POST` | `/admin/service-catalogue/records` | Add a new record to the service catalogue. | `SERVICE_EDIT` |
| `GET` | `/admin/service-catalogue/records/{record_id}` | Retrieve details of a service catalogue record. | `SERVICE_VIEW` |
| `PATCH` | `/admin/service-catalogue/records/{record_id}` | Modify catalogue record pricing or rules. | `SERVICE_EDIT` |
| `DELETE` | `/admin/service-catalogue/records/{record_id}` | Remove a record from the catalogue. | `SERVICE_EDIT` |

---

## 7. Department Overview Setup (Pharmacy, Radiology, Lab)

### 7.1 Pharmacy Overview
| Method | Endpoint | Purpose / Description | Required Permission |
| :--- | :--- | :--- | :--- |
| `GET` | `/admin/pharmacy-overview/summary` | Retrieve pharmacy sales, dispensing speed, and stock alerts. | `PHARMACY_VIEW` |
| `GET` | `/admin/pharmacy-overview/purchase-orders` | List purchase orders initiated by the pharmacy. | `PROCUREMENT_ORDER_VIEW` |
| `POST` | `/admin/pharmacy-overview/purchase-orders` | Create a new purchase order. | `PROCUREMENT_ORDER_EDIT` |
| `GET` | `/admin/pharmacy-overview/purchase-orders/{po_id}` | Retrieve specific PO details. | `PROCUREMENT_ORDER_VIEW` |
| `PATCH` | `/admin/pharmacy-overview/purchase-orders/{po_id}` | Update purchase order. | `PROCUREMENT_ORDER_EDIT` |
| `DELETE` | `/admin/pharmacy-overview/purchase-orders/{po_id}` | Cancel/delete a purchase order. | `PROCUREMENT_ORDER_EDIT` |

### 7.2 Radiology & Laboratory Overview
| Method | Endpoint | Purpose / Description | Required Permission |
| :--- | :--- | :--- | :--- |
| `GET` | `/admin/radiology-overview/summary` | Retrieve scanning statistics and device load factors. | `RADIOLOGY_VIEW` |
| `GET` | `/admin/radiology-overview/schedule` | Retrieve radiology device scheduling slots. | `RADIOLOGY_VIEW` |
| `POST` | `/admin/radiology-overview/schedule` | Book a radiology schedule slot. | `RADIOLOGY_EDIT` |
| `GET` | `/admin/radiology-overview/schedule/{schedule_id}` | Retrieve specific slot details. | `RADIOLOGY_VIEW` |
| `PATCH` | `/admin/radiology-overview/schedule/{schedule_id}` | Reschedule radiology scan. | `RADIOLOGY_EDIT` |
| `DELETE` | `/admin/radiology-overview/schedule/{schedule_id}` | Cancel radiology scan slot. | `RADIOLOGY_EDIT` |
| `GET` | `/admin/laboratory-overview/summary` | Retrieve test processing rates and critical values. | `LAB_VIEW` |
| `GET` | `/admin/laboratory-overview/tests` | Retrieve list of configured lab tests. | `LAB_VIEW` |
| `POST` | `/admin/laboratory-overview/tests` | Add a new test type to the lab. | `LAB_EDIT` |
| `GET` | `/admin/laboratory-overview/tests/{test_id}` | Retrieve laboratory test configurations. | `LAB_VIEW` |
| `PATCH` | `/admin/laboratory-overview/tests/{test_id}` | Modify lab test specs or normal ranges. | `LAB_EDIT` |
| `DELETE` | `/admin/laboratory-overview/tests/{test_id}` | Remove a test type from the lab. | `LAB_EDIT` |

---

## 8. Inventory & Procurement Management

### 8.1 Inventory Items
| Method | Endpoint | Purpose / Description | Required Permission |
| :--- | :--- | :--- | :--- |
| `GET` | `/admin/inventory/medical` | List all medical inventory items (e.g. syringes, medicine batches). | `INVENTORY_VIEW` |
| `POST` | `/admin/inventory/medical` | Add a new medical inventory item. | `INVENTORY_EDIT` |
| `GET` | `/admin/inventory/medical/{item_id}` | Retrieve details of a specific medical item. | `INVENTORY_VIEW` |
| `PATCH` | `/admin/inventory/medical/{item_id}` | Modify medical inventory parameters. | `INVENTORY_EDIT` |
| `DELETE` | `/admin/inventory/medical/{item_id}` | Delete a medical item. | `INVENTORY_EDIT` |
| `GET` | `/admin/inventory/non-medical` | List all non-medical assets (e.g. computers, surgical gowns). | `INVENTORY_VIEW` |
| `POST` | `/admin/inventory/non-medical` | Register a new non-medical asset. | `INVENTORY_EDIT` |
| `GET` | `/admin/inventory/non-medical/{item_id}` | Retrieve details of a specific non-medical asset. | `INVENTORY_VIEW` |
| `PATCH` | `/admin/inventory/non-medical/{item_id}` | Modify non-medical asset parameters. | `INVENTORY_EDIT` |
| `DELETE` | `/admin/inventory/non-medical/{item_id}` | Delete a non-medical asset. | `INVENTORY_EDIT` |
| `GET` | `/admin/inventory/stock-alerts` | Retrieve items operating below their minimum safety thresholds. | `INVENTORY_VIEW` |
| `GET` | `/admin/inventory/valuation` | Fetch total valuation of active stock using FIFO/Weighted Average. | `INVENTORY_VIEW` |

### 8.2 Stock Transfers
| Method | Endpoint | Purpose / Description | Required Permission |
| :--- | :--- | :--- | :--- |
| `GET` | `/admin/inventory/transfers` | List all inter-departmental or inter-branch stock transfers. | `INVENTORY_VIEW` |
| `POST` | `/admin/inventory/transfers` | Request a new stock transfer. | `INVENTORY_EDIT` |
| `GET` | `/admin/inventory/transfers/{transfer_id}` | Retrieve details of a transfer request. | `INVENTORY_VIEW` |
| `PATCH` | `/admin/inventory/transfers/{transfer_id}` | Update transfer details. | `INVENTORY_EDIT` |
| `DELETE` | `/admin/inventory/transfers/{transfer_id}` | Cancel a transfer request. | `INVENTORY_EDIT` |
| `POST` | `/admin/inventory/transfers/{transfer_id}/approve` | Approve and commit stock deduct/add for a transfer. | `INVENTORY_APPROVE` |
| `POST` | `/admin/inventory/transfers/{transfer_id}/reject` | Reject a stock transfer request. | `INVENTORY_APPROVE` |

### 8.3 Suppliers & Procurement
| Method | Endpoint | Purpose / Description | Required Permission |
| :--- | :--- | :--- | :--- |
| `GET` | `/admin/inventory/suppliers` | List all active vendors/suppliers. | `INVENTORY_VIEW` |
| `POST` | `/admin/inventory/suppliers` | Register a new vendor. | `INVENTORY_EDIT` |
| `GET` | `/admin/inventory/suppliers/{supplier_id}` | Retrieve details of a supplier. | `INVENTORY_VIEW` |
| `PATCH` | `/admin/inventory/suppliers/{supplier_id}` | Update supplier contact info or bank details. | `INVENTORY_EDIT` |
| `DELETE` | `/admin/inventory/suppliers/{supplier_id}` | Deregister a supplier. | `INVENTORY_EDIT` |
| `GET` | `/admin/procurement/requisitions` | List internal department purchase requisitions. | `PROCUREMENT_ORDER_VIEW` |
| `POST` | `/admin/procurement/requisitions` | Create a new purchase requisition. | `PROCUREMENT_ORDER_EDIT` |
| `GET` | `/admin/procurement/requisitions/{req_id}` | Retrieve details of a purchase requisition. | `PROCUREMENT_ORDER_VIEW` |
| `PATCH` | `/admin/procurement/requisitions/{req_id}` | Update a requisition. | `PROCUREMENT_ORDER_EDIT` |
| `DELETE` | `/admin/procurement/requisitions/{req_id}` | Cancel/delete a requisition. | `PROCUREMENT_ORDER_EDIT` |
| `POST` | `/admin/procurement/requisitions/{req_id}/approve` | Approve a requisition (transitions it to PO eligibility). | `PROCUREMENT_ORDER_APPROVE` |
| `POST` | `/admin/procurement/requisitions/{req_id}/reject` | Reject a purchase requisition. | `PROCUREMENT_ORDER_APPROVE` |
| `GET` | `/admin/procurement/orders` | List purchase orders. | `PROCUREMENT_ORDER_VIEW` |
| `POST` | `/admin/procurement/orders` | Generate a new PO. | `PROCUREMENT_ORDER_EDIT` |
| `GET` | `/admin/procurement/orders/{po_id}` | Retrieve specific PO details. | `PROCUREMENT_ORDER_VIEW` |
| `PATCH` | `/admin/procurement/orders/{po_id}` | Update details of a PO. | `PROCUREMENT_ORDER_EDIT` |
| `DELETE` | `/admin/procurement/orders/{po_id}` | Cancel a PO. | `PROCUREMENT_ORDER_EDIT` |
| `POST` | `/admin/procurement/orders/{po_id}/approve` | Approve a purchase order. | `PROCUREMENT_ORDER_APPROVE` |
| `POST` | `/admin/procurement/orders/{po_id}/reject` | Reject a purchase order. | `PROCUREMENT_ORDER_APPROVE` |
| `GET` | `/admin/procurement/orders/{po_id}/grns` | List Goods Received Notes (GRN) linked to a PO. | `PROCUREMENT_ORDER_VIEW` |
| `POST` | `/admin/procurement/orders/{po_id}/grns` | Generate a new GRN (commit item stock to warehouse). | `PROCUREMENT_ORDER_EDIT` |
| `GET` | `/admin/procurement/orders/{po_id}/grns/{grn_id}` | Retrieve details of a specific GRN. | `PROCUREMENT_ORDER_VIEW` |
| `PATCH` | `/admin/procurement/orders/{po_id}/grns/{grn_id}` | Update a GRN. | `PROCUREMENT_ORDER_EDIT` |
| `DELETE` | `/admin/procurement/orders/{po_id}/grns/{grn_id}` | Delete a GRN. | `PROCUREMENT_ORDER_EDIT` |
| `POST` | `/admin/procurement/orders/{po_id}/grns/{grn_id}/verify` | Verify and authorize a GRN (locks quantities). | `PROCUREMENT_ORDER_APPROVE` |
| `GET` | `/admin/procurement/orders/{po_id}/invoices` | List vendor invoices linked to a PO. | `PROCUREMENT_ORDER_VIEW` |
| `POST` | `/admin/procurement/orders/{po_id}/invoices` | Create a vendor invoice. | `PROCUREMENT_ORDER_EDIT` |
| `GET` | `/admin/procurement/orders/{po_id}/invoices/{inv_id}` | Retrieve details of a vendor invoice. | `PROCUREMENT_ORDER_VIEW` |
| `PATCH` | `/admin/procurement/orders/{po_id}/invoices/{inv_id}` | Update a vendor invoice. | `PROCUREMENT_ORDER_EDIT` |
| `DELETE` | `/admin/procurement/orders/{po_id}/invoices/{inv_id}` | Delete a vendor invoice. | `PROCUREMENT_ORDER_EDIT` |
| `POST` | `/admin/procurement/orders/{po_id}/invoices/{inv_id}/reconcile`| Reconcile PO, GRN, and Invoice values (3-Way Match). | `PROCUREMENT_ORDER_APPROVE` |
| `GET` | `/admin/procurement/analytics` | Requisitions vs PO and GRN fulfillment rate analytics. | `PROCUREMENT_ORDER_VIEW` |

---

## 9. TPA & Insurance Claims Management

| Method | Endpoint | Purpose / Description | Required Permission |
| :--- | :--- | :--- | :--- |
| `GET` | `/admin/tpa/providers` | List all active Third Party Administrators (TPA) & Insurance partners. | `TPA_VIEW` |
| `POST` | `/admin/tpa/providers` | Register a new TPA provider. | `TPA_EDIT` |
| `GET` | `/admin/tpa/providers/{provider_id}` | Retrieve specific TPA details. | `TPA_VIEW` |
| `PATCH` | `/admin/tpa/providers/{provider_id}` | Update TPA provider configuration. | `TPA_EDIT` |
| `DELETE` | `/admin/tpa/providers/{provider_id}` | Delete a TPA provider. | `TPA_EDIT` |
| `GET` | `/admin/tpa/providers/{provider_id}/schemes` | List insurance schemes offered by a TPA. | `TPA_VIEW` |
| `POST` | `/admin/tpa/providers/{provider_id}/schemes` | Add a new insurance scheme. | `TPA_EDIT` |
| `GET` | `/admin/tpa/providers/{provider_id}/schemes/{scheme_id}`| Retrieve specific scheme details. | `TPA_VIEW` |
| `PATCH` | `/admin/tpa/providers/{provider_id}/schemes/{scheme_id}`| Modify scheme details (deductibles, co-pays). | `TPA_EDIT` |
| `DELETE` | `/admin/tpa/providers/{provider_id}/schemes/{scheme_id}`| Remove an insurance scheme. | `TPA_EDIT` |
| `GET` | `/admin/tpa/claims` | List all patients insurance claims. | `TPA_VIEW` |
| `POST` | `/admin/tpa/claims` | File a new insurance claim. | `TPA_EDIT` |
| `GET` | `/admin/tpa/claims/{claim_id}` | Retrieve details of a filed claim. | `TPA_VIEW` |
| `PATCH` | `/admin/tpa/claims/{claim_id}` | Modify a claim's parameters. | `TPA_EDIT` |
| `DELETE` | `/admin/tpa/claims/{claim_id}` | Cancel/delete a filed claim. | `TPA_EDIT` |
| `POST` | `/admin/tpa/claims/{claim_id}/preauth` | Submit pre-authorization request to insurance portal. | `TPA_EDIT` |
| `POST` | `/admin/tpa/claims/{claim_id}/verify` | Verify and check validation rules for claims documents. | `TPA_EDIT` |
| `POST` | `/admin/tpa/claims/{claim_id}/settle` | Log an insurance settlement payment amount. | `TPA_EDIT` |
| `POST` | `/admin/tpa/claims/{claim_id}/reject` | Log a claim rejection reason. | `TPA_EDIT` |
| `GET` | `/admin/tpa/claims/analytics` | Rejections, delays, and average claim settlement duration analytics. | `TPA_VIEW` |

---

## 10. Audit Logging, Alerts & Compliance

| Method | Endpoint | Purpose / Description | Required Permission |
| :--- | :--- | :--- | :--- |
| `GET` | `/admin/audit-logs` | Retrieve chronological administrative audit logs. | `AUDIT_LOG_VIEW` |
| `GET` | `/admin/alerts` | List active administrative system alerts (e.g. stock low, critical lab). | `ALERT_VIEW` |
| `POST` | `/admin/alerts` | Trigger a new administrative alert. | `ALERT_EDIT` |
| `GET` | `/admin/alerts/{alert_id}` | Retrieve details of a specific alert. | `ALERT_VIEW` |
| `PATCH` | `/admin/alerts/{alert_id}` | Modify alert properties. | `ALERT_EDIT` |
| `DELETE` | `/admin/alerts/{alert_id}` | Delete an alert. | `ALERT_EDIT` |
| `POST` | `/admin/alerts/{alert_id}/acknowledge` | Acknowledge an alert (assigns acknowledgment user context). | `ALERT_EDIT` |
| `POST` | `/admin/alerts/{alert_id}/resolve` | Resolve an active system alert. | `ALERT_EDIT` |
| `GET` | `/admin/compliance/audit-logs` | Fetch regulatory compliance audit trails (e.g. data downloads). | `COMPLIANCE_VIEW` |
| `GET` | `/admin/compliance/retention-policies` | List active database retention and archival policies. | `COMPLIANCE_VIEW` |
| `POST` | `/admin/compliance/retention-policies` | Create a new data retention policy. | `COMPLIANCE_EDIT` |
| `GET` | `/admin/compliance/retention-policies/{policy_id}` | Retrieve retention policy details. | `COMPLIANCE_VIEW` |
| `PATCH` | `/admin/compliance/retention-policies/{policy_id}` | Update data retention metrics. | `COMPLIANCE_EDIT` |
| `DELETE` | `/admin/compliance/retention-policies/{policy_id}` | Remove a data retention policy. | `COMPLIANCE_EDIT` |
| `GET` | `/admin/compliance/encryption-keys` | List active branch data-encryption keys. | `COMPLIANCE_VIEW` |
| `POST` | `/admin/compliance/encryption-keys` | Register or generate a new encryption key. | `COMPLIANCE_EDIT` |
| `GET` | `/admin/compliance/encryption-keys/{key_id}` | Retrieve specific key details. | `COMPLIANCE_VIEW` |
| `PATCH` | `/admin/compliance/encryption-keys/{key_id}` | Rotate or disable a specific encryption key. | `COMPLIANCE_EDIT` |
| `DELETE` | `/admin/compliance/encryption-keys/{key_id}` | Remove an encryption key catalog reference. | `COMPLIANCE_EDIT` |
