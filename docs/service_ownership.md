# HMS Service Ownership Reference

> **Last Updated:** June 2026
> This document is the canonical reference for which microservice owns which API domain.
> When in doubt about where an endpoint lives, check here first.

---

## Service Map

| Service | Directory | Base Paths | Owns |
|---------|-----------|-----------|------|
| **OPD** | `services/opd` | `/opd/...` | Appointments, Queue, OPD Visit lifecycle, Doctors, Departments, Scheduling, Walk-in, Emergency, Tasks, Surgery orders, Billing summary, Documents, Alerts, **Reception Dashboard** |
| **IPD** | `services/ipd` | `/ipd/...` | Admissions, Beds, Wards, Nursing assignments, Observations, Transfers, Discharge, DAMA, Death management |
| **Patient** | `services/patient` | `/patients/...` | Patient registration, UHID, Patient records, Search, Patient Vitals |
| **Clinical** | `services/clinical` | `/clinical/...` | Consultations, SOAP Notes, Clinical forms, Service orders, OT sessions |
| **Doctors** | `services/doctors` | `/doctors/...` | Doctor profiles, Schedules |
| **Auth** | `services/auth` | `/auth/...`, `/hrms/...`, `/organization/...` | Authentication, JWT tokens, RBAC roles & permissions, HRMS staff, Branch switching |
| **Admin** | `services/admin` | `/admin/...` | Tenant admin, Module activation, Branch config, Settings |
| **Billing** | `services/billing` | `/billing/...` | Invoices, Payment records, Billing items |
| **Pharmacy** | `services/pharmacy` | `/pharmacy/...` | Pharmacy orders, Drug dispensing, Inventory, Prescriptions |
| **Lab** | `services/lab` | `/lab/...` | Lab test orders, Results, Reports |
| **Nurse** | `services/nurse` | `/nurse/...` | Nursing tasks, Shift management, Medication administration |

---

## Decommissioned Services

| Service | Status | Migration |
|---------|--------|-----------|
| **OPS** | ✅ Deleted (June 2026) | All `/ops/...` endpoints moved to `opd` or `ipd`. See [ops_apis_update.md](./ops_apis_update.md) for full migration map. |

---

## Domain Ownership Rules

### OPD owns
- Everything that happens **before** a patient is admitted (outpatient journey)
- Reception dashboard and daily operational view
- Doctor queue and scheduling

### IPD owns
- Everything that happens **after** a patient is admitted (inpatient journey)
- Beds and wards
- Nursing and discharge

### Cross-Service Reads (allowed, not owned)
These are read-only cross-service lookups — the data is owned by the listed service, but others may read it:

| Data | Owner | Who may read |
|------|-------|-------------|
| Patient admissions list | IPD | OPD (for doctor's patient view) |
| Patient vitals | Patient/Clinical | IPD nurses |
| Service orders | Clinical | OPD, IPD |

---

## API Base Path Conventions

All services follow this convention:
```
/{service-prefix}/{resource}/{id}/{sub-resource}
```

Examples:
- `POST /opd/appointments` — OPD service, appointments resource
- `GET /ipd/admissions/{id}/notes` — IPD service, admission sub-resource
- `POST /patients/register` — Patient service

> **Note:** The OPS service used `/ops/...` paths which were transparently mapped to `/opd/...` internally. Now that OPS is gone, all paths are true `/opd/...` or `/ipd/...` paths served by the correct service.
