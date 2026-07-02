# HMS — CMS Branch Configuration API Documentation

This document specifies the REST API endpoints required by the Content Management System (CMS) to dynamically manage branch-level department activations and module enablement. 

Since branch-department assignments and branch-module states are not predefined in the database initialization, the CMS uses these endpoints to query the master catalog and toggle features on a per-branch basis.

---

## 1. Global Conventions

### Base URL
All CMS configuration requests target the Admin configuration prefix:
```
https://<api-id>.execute-api.ap-south-1.amazonaws.com/<stage>/admin/config
```

### Authorization
Requires a valid Admin JWT Access Token passed in the headers:
```http
Authorization: Bearer <access_token>
```

---

## 2. Department Management API

Used to retrieve the global master list of departments and manage which departments are assigned/activated for a specific branch.

### 2.1 Get Master Departments List
Retrieve all available departments registered in the master catalog (`platform.departments`).

* **Endpoint:** `GET /admin/config/departments`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "departments": [
      {
        "id": "3d756c7b-5a64-423a-bb9d-d24491359b8a",
        "name": "Gynecology",
        "type": "CLINICAL",
        "is_active": true
      },
      {
        "id": "330a2b57-f895-4cbe-9d94-0403d3f4ee6b",
        "name": "Laboratory",
        "type": "DIAGNOSTICS",
        "is_active": true
      }
    ]
  }
}
```

---

### 2.2 Get Branch Departments Configuration
Retrieve the active status of departments for a specific facility/branch (`platform.branch_departments`).

* **Endpoint:** `GET /admin/config/branches/{branch_id}/departments`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "branch_id": "87fb739d-2ab3-4df4-bd9b-50cd6da8a113",
    "departments": [
      {
        "department_id": "3d756c7b-5a64-423a-bb9d-d24491359b8a",
        "name": "Gynecology",
        "type": "CLINICAL",
        "is_active": true,
        "assigned_at": "2026-06-15T10:04:12Z"
      },
      {
        "department_id": "330a2b57-f895-4cbe-9d94-0403d3f4ee6b",
        "name": "Laboratory",
        "type": "DIAGNOSTICS",
        "is_active": false,
        "assigned_at": "2026-06-15T10:04:12Z"
      }
    ]
  }
}
```

---

### 2.3 Toggle Branch Department Activation
Enable or disable a specific department for a branch. This modifies the `platform.branch_departments` mapping.

* **Endpoint:** `POST /admin/config/branches/{branch_id}/departments/{department_id}/toggle`
* **Request Body:**
```json
{
  "is_active": true
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "branch_id": "87fb739d-2ab3-4df4-bd9b-50cd6da8a113",
    "department_id": "3d756c7b-5a64-423a-bb9d-d24491359b8a",
    "is_active": true,
    "updated_at": "2026-06-15T10:05:00Z"
  },
  "message": "Department Gynecology has been successfully activated at this branch."
}
```

---

## 3. Module Configuration API

Used to enable/disable feature modules at individual branches. Enabling a module automatically registers its associated departments to the branch via triggers.

### 3.1 Get Master Modules List
Retrieve all available functional modules defined in `platform.modules` along with their department associations and dependency requirements.

* **Endpoint:** `GET /admin/config/modules`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "modules": [
      {
        "module_code": "MOD-IPD",
        "display_name": "Inpatient (IPD)",
        "description": "Admissions, wards, beds, daily rounds, discharge lifecycle",
        "depends_on": [],
        "sort_order": 20,
        "activates_departments": ["Nursing"]
      },
      {
        "module_code": "MOD-ICU",
        "display_name": "ICU / Critical Care",
        "description": "ICU bed management under IPD",
        "depends_on": ["MOD-IPD"],
        "sort_order": 21,
        "activates_departments": ["ICU / Critical Care"]
      }
    ]
  }
}
```

---

### 3.2 Get Branch Module Status
Retrieve the on/off state of all modules at a specific branch (`platform.branch_modules`).

* **Endpoint:** `GET /admin/config/branches/{branch_id}/modules`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "branch_id": "87fb739d-2ab3-4df4-bd9b-50cd6da8a113",
    "modules": [
      {
        "module_code": "MOD-IPD",
        "display_name": "Inpatient (IPD)",
        "is_active": true,
        "enabled_at": "2026-06-15T10:01:00Z",
        "disabled_at": null
      },
      {
        "module_code": "MOD-ICU",
        "display_name": "ICU / Critical Care",
        "is_active": false,
        "enabled_at": null,
        "disabled_at": "2026-06-15T10:02:15Z"
      }
    ]
  }
}
```

---

### 3.3 Toggle Branch Module Enablement
Toggle the activation of a module for a branch. 

> [!IMPORTANT]
> The database enforces dependency checks. For example, trying to enable `MOD-ICU` when `MOD-IPD` is disabled will return a validation error. Enabling a module automatically syncs its associated departments (`platform.module_departments`) to `platform.branch_departments` via triggers.

* **Endpoint:** `POST /admin/config/branches/{branch_id}/modules/{module_code}/toggle`
* **Request Body:**
```json
{
  "is_active": true,
  "notes": "Enabling ICU block on customer request"
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "branch_id": "87fb739d-2ab3-4df4-bd9b-50cd6da8a113",
    "module_code": "MOD-ICU",
    "is_active": true,
    "enabled_at": "2026-06-15T10:07:00Z",
    "departments_activated": ["ICU / Critical Care"]
  },
  "message": "Module MOD-ICU has been enabled. Synced associated departments."
}
```

* **Error Response (400 Bad Request - Dependency Failure):**
```json
{
  "success": false,
  "code": 400,
  "message": "Dependency Error: Module MOD-ICU requires MOD-IPD to be active first."
}
```
