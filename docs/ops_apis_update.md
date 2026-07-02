# HMS OPS Service — Deprecation Notice

> **⚠️ The `ops` service has been fully decommissioned.**
>
> All endpoints previously served under `/ops/...` or `/opd/...` by the OPS Lambda have been migrated to the correct domain-owning services. The `services/ops/` directory has been deleted from the repository.

---

## Migration Map

All OPS endpoints are now available at the same path under the correct service:

### Reception Dashboard → OPD Service

| Old (OPS) | New (OPD) | Notes |
|-----------|-----------|-------|
| `GET /ops/dashboard` | `GET /opd/dashboard` | Now served by `services/opd` |
| `GET /ops/dashboard/date/<date>` | `GET /opd/dashboard/date/<date>` | Now served by `services/opd` |

### Appointments, Queue & Scheduling → OPD Service

| Old (OPS) | New (OPD) |
|-----------|-----------|
| `GET /ops/appointments/{id}` | `GET /opd/appointments/{id}` |
| `GET /ops/appointments` | `GET /opd/appointments` |
| `GET /ops/appointments/today` | `GET /opd/appointments/today` |
| `GET /ops/appointments/date/<date>` | `GET /opd/appointments/date/<date>` |
| `PATCH /ops/appointments/{id}/status` | `PATCH /opd/appointments/{id}/status` |
| `POST /ops/appointments/{id}/emergency` | `POST /opd/appointments/{id}/emergency` |
| `GET /ops/queue/{doctor_id}` | `GET /opd/queue/{doctor_id}` |
| `GET /ops/queue/{doctor_id}/token-preview` | `GET /opd/queue/{doctor_id}/token-preview` |
| `GET /ops/appointments/{id}/position` | `GET /opd/appointments/{id}/position` |
| `GET /ops/departments` | `GET /opd/departments` |
| `GET /ops/departments/{id}/doctors` | `GET /opd/departments/{id}/doctors` |
| `GET /ops/doctors` | `GET /opd/doctors` |
| `GET /ops/doctors/availability` | `GET /opd/doctors/availability` |

### Surgery → OPD Service

| Old (OPS) | New (OPD) |
|-----------|-----------|
| `PATCH /ops/surgery/{id}/schedule` | `PATCH /opd/surgery/{id}/schedule` |
| `PATCH /ops/surgery/{id}/status` | `PATCH /opd/surgery/{id}/status` |
| `GET /ops/surgery/catalogue` | `GET /opd/surgery/catalogue` |
| `GET /ops/surgery` | `GET /opd/surgery` |

### Workflow → OPD Service

| Old (OPS) | New (OPD) |
|-----------|-----------|
| `POST /ops/workflow/post-consultation` | `POST /opd/workflow/post-consultation` |

### Admissions, Beds & Wards → IPD Service

| Old (OPS) | New (IPD) |
|-----------|-----------|
| `POST /ops/admissions` | `POST /ipd/admissions` |
| `GET /ops/admissions` | `GET /ipd/admissions` |
| `GET /ops/admissions/{id}` | `GET /ipd/admissions/{id}` |
| `POST /ops/admissions/{id}/status` | `POST /ipd/admissions/{id}/status` |
| `POST /ops/admissions/{id}/notes` | `POST /ipd/admissions/{id}/notes` |
| `GET /ops/admissions/{id}/notes` | `GET /ipd/admissions/{id}/notes` |
| `GET /ops/beds` | `GET /ipd/beds` |
| `GET /ops/beds/availability` | `GET /ipd/beds/availability` |
| `GET /ops/wards` | `GET /ipd/wards` |
| `GET /ops/wards/occupancy` | `GET /ipd/wards/occupancy` |

---

## Current Service Ownership

```
services/
├── opd/        ← Appointments, Queue, Emergency, OPD Visit, Doctors,
│                 Departments, Tasks, Surgery, Billing,
│                 Documents, Alerts, Walk-in, Reception Dashboard
├── ipd/        ← Admissions, Beds, Wards, Discharge, DAMA,
│                 Death, Observations, Nursing
├── patient/    ← Patient Registration, UHID, Records, Vitals
├── clinical/   ← Consultations, SOAP Notes, Forms
├── doctors/    ← Doctor Profiles, Schedules
├── auth/       ← Authentication, RBAC, HRMS
├── admin/      ← Tenant Admin, Branch Config, Module Activation
├── billing/    ← Billing Summaries, Invoices
├── pharmacy/   ← Pharmacy Orders, Dispensing, Prescriptions
├── lab/        ← Lab Orders, Results
├── nurse/      ← Nursing Tasks, Shift Management
└── clinical/   ← EMR, Consultations, Service Orders
```

---

## CI/CD

The `ops` service has been removed from the GitHub Actions build workflow. The following services are currently available for deployment:

`auth` | `admin` | `patient` | `billing` | `pharmacy` | `lab` | `doctors` | `nurse` | `opd` | `ipd` | `clinical`

---

*Decommissioned: June 2026*
