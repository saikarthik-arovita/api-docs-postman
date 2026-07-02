# Arovita HMS – Overall Enterprise Architecture & CMS Requirements Plan
🌐 One Platform • Multiple Healthcare Business Models • Configurable Healthcare SaaS Ecosystem

This document specifies the business objectives, system capabilities, database-driven configurations, and architectural layout for the Arovita Content Management System (CMS) and Healthcare Management System (HMS) SaaS platform.

---

## 🎯 1. Architecture Vision
Build a single Healthcare SaaS Platform capable of serving diverse business models:
*   **Medical Facility Segments**: Single Doctor Clinics, Family Clinics, Polyclinics, Specialty Clinics, Diagnostic Centers, Small Hospitals, Multi-Specialty Hospitals, Hospital Chains, Corporate Healthcare Networks.
*   **Care Modalities**: Telemedicine Providers, Home Healthcare Services, Wellness & Preventive Care Centers.

### Core Architecture Principles
*   ✅ **Single Codebase**: No tenant-specific code branches.
*   ✅ **Multi-Tenant Isolation**: Row-Level Security (RLS) restricts access based on tenant and branch.
*   ✅ **Shared Infrastructure**: Multi-tenant database schema deployment.
*   ✅ **Configurable Modules & Features**: Dynamic runtime enablement of interfaces.
*   ✅ **Branch-Level Customization**: Fine-grained settings per facility.
*   ✅ **Subscription-Aware Engine**: Access limits dynamically resolved at login.
*   ✅ **API-First & Event-Ready Design**: Decoupled asynchronous services.

---

## 🏗️ 2. High-Level Architecture

```
┌────────────────────────────────────────────────────────┐
│                  Client Applications                   │
├────────────────────────────────────────────────────────┤
│  Flutter Web (Optional)  │  Flutter Desktop  │  Mobile │
└──────────────────────────┬─────────────────────────────┘
                           │
                           ▼
┌────────────────────────────────────────────────────────┐
│                   CDN / Cloudflare                     │
└──────────────────────────┬─────────────────────────────┘
                           │
                           ▼
┌────────────────────────────────────────────────────────┐
│                     API Gateway                        │
└──────────────────────────┬─────────────────────────────┘
                           │
                           ▼
┌────────────────────────────────────────────────────────┐
│                 Authentication & RBAC                  │
│       JWT • OAuth2 • SSO • MFA • Cognito IAM           │
└──────────────────────────┬─────────────────────────────┘
                           │
                           ▼
┌────────────────────────────────────────────────────────┐
│                   Application Layer                    │
├────────────────────────────────────────────────────────┤
│ Tenant · Organization · Branch · Subscription · Catalog│
│ Workflow · Notification · Billing · Reporting · Audit  │
└──────────────────────────┬─────────────────────────────┘
                           │
                           ▼
┌────────────────────────────────────────────────────────┐
│                    Business Modules                    │
├────────────────────────────────────────────────────────┤
│ Clinical · Diagnostics · Patient Care · Admin · Finance│
└──────────────────────────┬─────────────────────────────┘
                           │
                           ▼
┌────────────────────────────────────────────────────────┐
│                       Data Layer                       │
├────────────────────────────────────────────────────────┤
│ PostgreSQL · Redis · S3 Storage · OpenSearch · Redshift│
└────────────────────────────────────────────────────────┘
```

---

## 🏢 3. Business Architecture & Hierarchy

```
Platform
  └── Organizations
       └── Branches
            ├── Active Departments
            ├── Specialties & Diagnostics
            ├── Custom Services & Pricing
            ├── Active Users & Schedules
            ├── Local Workflows & Operating Hours
            └── Tax & Insurance Configurations
```

### 3.1 Tenant Hierarchy
Each branch operates independently under the organization umbrella. At the branch level, the system dynamically configures:
*   **Specialties & Diagnostics**: Active clinics and lab/radiology panels.
*   **Pricing**: Services priced per branch or insurance scheme.
*   **Operating Hours**: Scheduling calendars and leave overrides.
*   **Integrations**: Local insurance (TPA) providers and tax configurations.

---

## 📚 4. Catalog & Configuration Architecture

### 4.1 Master Catalog
The single source of truth at the Organization level:
*   **Metadata**: Clinical Specialties, Diagnostic Services, Roles, Workflows, Document Forms, Report Formats, Templates, and API Integrations.
*   **Principle**: Defined once by the system administrator; read-only at the branch level.

### 4.2 Organization Settings Configuration
Defines what the parent organization pays for. Governs organization-wide limits for enabled Specialties, Diagnostics, Add-On Services, and billing integrations.

### 4.3 Branch Configuration
Defines the local clinic context. Resolves which sub-selection of active organization capabilities are assigned to the branch, along with specific hours, local users, queue strategies, billing configurations, and notifications.

---

## 🩺 5. Functional Business Modules

### 🏥 Clinical Modules
*   **Patient Registration**: Demographics, UHID sequence generation.
*   **Appointment Management**: Doctor scheduling, availability overrides.
*   **Consultation & SOAP Notes**: Digital prescription builder, chief complaints, diagnostic ordering.
*   **Clinical History**: Chronological timeline of patient allergies, chronic conditions, and previous visits.
*   **Referrals**: Core workflows for internal, cross-department, or external referrals.

### 🔬 Diagnostic Modules
*   **Laboratory & Pathology**: Barcode tracking, sample collection, and report validation.
*   **Radiology / Imaging**: CT, MRI, and X-Ray scheduling, report uploads, and validations.

### 🚑 Patient Care Modules
*   **OPD (Outpatient)**: Lobby check-in queues, consultation flow tracking.
*   **IPD (Inpatient)**: Ward/bed maps, admission profiles, ward daily rounds, and nurse notes.
*   **Nursing**: Work stations, shift handovers, and Medication Administration Records (MAR).
*   **OT (Operation Theatre)**: Surgery orders scheduling, PAC checkups, pre-op checklists, recovery.
*   **Emergency & Triage**: Rapid case intake, ER bay stretcher grid, NEWS code assignments.
*   **Pharmacy**: Inventory logs, batch tracking, GRN receipts, and medicine dispensing.

### 💰 Administration & Finance Modules
*   **Billing**: Consolidated invoice desks, cashier checklist widgets, and refunds.
*   **Insurance**: Pre-authorizations log, TPA mappings, and claims submission status.
*   **Inventory & Procurement**: Purchase order lifecycles and vendor details.
*   **HRMS & Payroll**: Employee clock-in logs, leaves requests, and roster shifts.

---

## 👥 6. Role & Permission Architecture

Access control follows a strict hierarchical evaluation chain:
$$\text{User} \longrightarrow \text{Role} \longrightarrow \text{Permissions} \longrightarrow \text{Features} \longrightarrow \text{Menus} \longrightarrow \text{Dashboard}$$

### 6.1 Role Definitions (Seeded Choices)
*   **System Admin (ITC-001)**: Super admin. Full read/write system overrides.
*   **Hospital Admin (ADM-001)**: General operational control, staff fee edits, and leaves approvals.
*   **Receptionist (ADM-002)**: Front-desk registrations, scheduling, and payment collections.
*   **Doctor (MED-001)**: Consultation execution, EMR access, and surgery ordering.
*   **Nurse (MED-002)**: Vitals tracking and MAR administration.
*   **Billing Executive (FIN-001)**: Cash counter operations and settlement.
*   **Pharmacist (MED-006)**: Medicine validation and stock dispensing.
*   **Lab Technician (MED-004)** / **Radiologist (MED-005)**: Diagnostic execution and report validation.
*   **Emergency Staff (EMR-002)**: ER triage and bay dispatcher.

### 6.2 Permission Guard Enforcement
Permissions govern which API endpoints are callable (e.g., `CREATE_APPOINTMENT`, `VIEW_PATIENT`, `DISPENSE_MEDICINE`, `MANAGE_BILLING`).
*   **Dynamic Navigation**: The UI receives the user’s combined permissions at login to hide or display menus and widgets dynamically.
*   **No Hardcoding**: Dashboards, sidebars, and actions are configuration-driven, not hardcoded.

---

## 🔄 7. Workflow & Subscription Architecture

### 7.1 Outpatient / Inpatient Workflow Sequence
$$\text{Appointment} \longrightarrow \text{Consultation} \longrightarrow \text{Prescription} \longrightarrow \text{Lab Order} \longrightarrow \text{Billing} \longrightarrow \text{Payment} \longrightarrow \text{Discharge}$$
Each step is event-driven, specialty-specific, and branch-configurable.

### 7.2 Subscription Configuration Flow
$$\text{Tenant Onboarding} \longrightarrow \text{Module Selection} \longrightarrow \text{Generate Subscription} \longrightarrow \text{Configure Branches} \longrightarrow \text{Assign Users} \longrightarrow \text{Go Live}$$
*   **Supported billing models**: Monthly Subscription, Annual, Pay Per User, Pay Per Branch, or Pay Per Transaction.

---

## ⚙️ 8. Integration Architecture & Link Strategy

To prevent coupling conflicts and ensure real-time synchronization between the **CMS Database** and the **HMS Transactional Database**, the system uses an **API-Driven Synchronous Synchronization** pattern.

```
 [CMS Administrative Dashboard]
               │
               ▼
   1. Write Configuration State
               │
               ▼
       [CMS Database]
               │
               ▼
   2. Propagate configuration immediately via REST API
               │
               ▼
 ┌────────────────────────────────────────────────────────┐
 │ HMS Backend REST Gateway (POST /admin/config/toggle)   │
 └──────────────────────────┬─────────────────────────────┘
                            │
                            ▼
   3. Update HMS Database & Execute PG Trigger
                            │
                            ▼
      [Active Departments & RLS Filters Synced]
```

1.  **Direct Propagation**: Any module/department toggle saved in the CMS database is immediately pushed to the HMS transaction database via an HTTP REST API endpoint (`POST /admin/config/branches/{id}/modules/{code}/toggle`).
2.  **Instantaneous Validation**: HMS validates the operation's dependency checks. If the HMS rejects the change (e.g., trying to disable `MOD-IPD` while `MOD-OT` is active), the CMS rolls back the local database change and returns the validation error message to the administrator.
3.  **Database-Level Trigger**: Once the HMS database registers the module toggle, an internal PostgreSQL trigger updates the `platform.branch_departments` table in real-time, activating or hiding the corresponding clinical dropdowns.

---

## 🗄️ 9. Database & Infrastructure Architecture

### 9.1 Database Separation
*   **Tenant Transactional Database**: Stores tenant-specific data (Patients, Appointments, Encounters, Billing, Inventory, Audit Logs).
*   **Shared Master Database**: Stores global catalog lookups, pricing plans, forms templates, default roles, and tenant subscription registry.

### 9.2 Infrastructure Layout
*   **DNS & Security**: Route53 ➔ Cloudflare (WAF/CDN) ➔ AWS Load Balancer.
*   **Container Compute**: AWS ECS Fargate or EKS hosting decoupled microservices.
*   **Database**: PostgreSQL RDS (Multi-AZ) + ElastiCache Redis for session cache.
*   **Asynchronous Messaging**: SQS for notification alerts and audit workers.
*   **Search & Analytics**: OpenSearch for rapid EMR indexing; Redshift for financial reporting.

---

## 🌟 Golden Rule
**Everything should be driven through configuration, subscriptions, catalogs, permissions, workflows, and feature flags — not by writing tenant-specific code.**

*Build once. Configure everywhere. Scale infinitely.* 🚀🏥
