# HMS CMS Backend Specification Guide

This document specifies the backend requirements for the Content Management System (CMS) Branch and Module Configuration. It outlines the database schemas, trigger-based synchronization rules, initial branch seeding patterns, and REST API endpoints.

---

## 1. Database Schema & Triggers

All tables reside in the `platform` schema. The CMS service will read/write to these tables to configure the branch capabilities.

### 1.1 Database Tables (DDL)

#### 1. platform.departments
The global master catalog of all clinical, support, and administrative departments.
*   `id` (UUID, Primary Key, Default: `gen_random_uuid()`)
*   `name` (VARCHAR(100), Unique, Not Null)
*   `description` (TEXT)
*   `type` (VARCHAR(50), Not Null) -- e.g., `'CLINICAL'`, `'DIAGNOSTICS'`, `'SUPPORT'`, `'ADMIN'`
*   `is_active` (BOOLEAN, Default: `TRUE`)

#### 2. platform.branch_departments
Mapping table indicating which departments are active at which branch.
*   `id` (UUID, Primary Key, Default: `gen_random_uuid()`)
*   `branch_id` (UUID, Foreign Key ➔ `platform.facilities(id)`, Not Null)
*   `department_id` (UUID, Foreign Key ➔ `platform.departments(id)`, Not Null)
*   `is_active` (BOOLEAN, Not Null, Default: `TRUE`)
*   `created_at` / `updated_at` (TIMESTAMPTZ, Default: `NOW()`)
*   **Constraints**: Unique composite constraint on `(branch_id, department_id)`

#### 3. platform.modules
The master catalog of plug-and-play feature modules.
*   `module_code` (VARCHAR(30), Primary Key) -- e.g., `'MOD-IPD'`, `'MOD-OT'`
*   `display_name` (VARCHAR(100), Not Null)
*   `description` (TEXT)
*   `depends_on` (VARCHAR(30)[] Array, Default: `'{}'`) -- Module codes required to be active first
*   `sort_order` (SMALLINT, Default: `0`)
*   `is_active` (BOOLEAN, Default: `TRUE`)

#### 4. platform.module_departments
Predefined mapping linking each module to the departments it controls.
*   `module_code` (VARCHAR(30), Foreign Key ➔ `platform.modules(module_code)`, Not Null)
*   `department_name` (VARCHAR(100), Not Null) -- Matches `platform.departments.name`
*   **Constraints**: Composite Primary Key `(module_code, department_name)`

#### 5. platform.branch_modules
Saves the active modules configured for each individual branch.
*   `id` (UUID, Primary Key, Default: `gen_random_uuid()`)
*   `branch_id` (UUID, Foreign Key ➔ `platform.facilities(id)`, Not Null)
*   `module_code` (VARCHAR(30), Foreign Key ➔ `platform.modules(module_code)`, Not Null)
*   `is_active` (BOOLEAN, Default: `TRUE`)
*   `enabled_by` (UUID, Foreign Key ➔ `identity.users(id)`)
*   `enabled_at` (TIMESTAMPTZ, Default: `NOW()`)
*   `disabled_by` (UUID, Foreign Key ➔ `identity.users(id)`)
*   `disabled_at` (TIMESTAMPTZ)
*   `notes` (TEXT)
*   **Constraints**: Unique composite constraint on `(branch_id, module_code)`

---

### 1.2 Automated Sync Trigger
When `platform.branch_modules` is modified, the database executes a PL/pgSQL trigger function to sync `platform.branch_departments`. 

*   **INSERT / UPDATE to active (`is_active = TRUE`)**:
    Upsert all department associations from `platform.module_departments` into `platform.branch_departments` for that branch, set `is_active = TRUE`.
*   **UPDATE to inactive (`is_active = FALSE`)**:
    Set `is_active = FALSE` for the module's departments in `platform.branch_departments` **only if** no other active module assigned to that branch shares the same department (preventing conflicts).

---

## 2. Organization & 7 Branches Seed Mapping

When onboarding the parent organization and its **7 initial branches** (`SLJB-001` to `SLJB-007`), the initial module states must be seeded into the database:

### 2.1 Branch Classification Map
*   **Branch 1 (`SLJB-001` - Bengaluru)**: Full-Service Hospital.
    *   *Active Modules*: All modules enabled (`MOD-ADMIN`, `MOD-BILLING`, `MOD-OPD`, `MOD-IPD`, `MOD-ICU`, `MOD-NURSING`, `MOD-OT`, `MOD-PHARMACY`, `MOD-LAB`, `MOD-RADIOLOGY`, `MOD-EMERGENCY`, `MOD-PHYSIO`, `MOD-DIETETICS`).
*   **Branch 2 (`SLJB-002` - Chitradurga)**: District General Hospital.
    *   *Active Modules*: Basic clinical stack, no OT, no ICU, no support therapists (`MOD-ADMIN`, `MOD-BILLING`, `MOD-OPD`, `MOD-IPD`, `MOD-NURSING`, `MOD-PHARMACY`, `MOD-LAB`, `MOD-RADIOLOGY`, `MOD-EMERGENCY`).
*   **Branch 3 (`SLJB-003` - Chennai)**: Mid-size Multi-Specialty.
    *   *Active Modules*: Full stack except ICU and dietetics (`MOD-ADMIN`, `MOD-BILLING`, `MOD-OPD`, `MOD-IPD`, `MOD-NURSING`, `MOD-OT`, `MOD-PHARMACY`, `MOD-LAB`, `MOD-RADIOLOGY`, `MOD-EMERGENCY`, `MOD-PHYSIO`).
*   **Branches 4 to 7 (`SLJB-004` to `SLJB-007` - Small Clinics)**: Outpatient-only facilities.
    *   *Active Modules*: OPD, basic pharmacy, lab, emergency triage (`MOD-ADMIN`, `MOD-BILLING`, `MOD-OPD`, `MOD-PHARMACY`, `MOD-LAB`, `MOD-EMERGENCY`). No IPD, no Nursing, no OT.

---

## 3. Core Business Logic Rules

The API or service layer must validate module transitions before updating `platform.branch_modules`.

### 3.1 Module Activation Checks (Dependency Rules)
Before enabling a module (`is_active: true`) at a branch:
1.  Read the `depends_on` array of the target module from `platform.modules`.
2.  Verify that all module codes in the array are present and active (`is_active = TRUE`) in `platform.branch_modules` for that specific `branch_id`.
3.  *Action*: If any dependency is missing, reject the activation with a `400 Bad Request` explaining which required modules are not active.

### 3.2 Module Deactivation Checks (Reverse Dependency Rules)
Before disabling a module (`is_active: false`) at a branch:
1.  Check if any other active module in `platform.branch_modules` for that `branch_id` lists the target module in its `depends_on` array.
2.  *Action*: If a dependency relies on the target, block deactivation and return a `400 Bad Request` listing the active dependent modules that must be disabled first.

---

## 4. REST API Endpoint Specifications

All endpoints require administrative authorization tokens (`Authorization: Bearer <token>`).

### 4.1 GET /admin/config/departments
Retrieve the global master list of departments.
*   **Query Params**: None.
*   **Success Response (`200 OK`)**:
    ```json
    {
      "success": true,
      "data": {
        "departments": [
          {
            "id": "3d756c7b-5a64-423a-bb9d-d24491359b8a",
            "name": "General Medicine",
            "type": "CLINICAL",
            "is_active": true
          }
        ]
      }
    }
    ```

### 4.2 GET /admin/config/branches/{branch_id}/departments
Retrieve the assigned departments configuration for a specific branch.
*   **Success Response (`200 OK`)**:
    ```json
    {
      "success": true,
      "data": {
        "branch_id": "87fb739d-2ab3-4df4-bd9b-50cd6da8a113",
        "departments": [
          {
            "department_id": "3d756c7b-5a64-423a-bb9d-d24491359b8a",
            "name": "General Medicine",
            "type": "CLINICAL",
            "is_active": true,
            "assigned_at": "2026-06-15T10:04:12Z"
          }
        ]
      }
    }
    ```

### 4.3 POST /admin/config/branches/{branch_id}/departments/{department_id}/toggle
Directly enable or disable a specific department for a branch.
*   **Request JSON Body**:
    ```json
    {
      "is_active": true
    }
    ```
*   **Success Response (`200 OK`)**:
    ```json
    {
      "success": true,
      "data": {
        "branch_id": "87fb739d-2ab3-4df4-bd9b-50cd6da8a113",
        "department_id": "3d756c7b-5a64-423a-bb9d-d24491359b8a",
        "is_active": true,
        "updated_at": "2026-06-15T10:05:00Z"
      }
    }
    ```

### 4.4 GET /admin/config/modules
Retrieve the master modules list along with their dependency arrays and the departments they activate.
*   **Success Response (`200 OK`)**:
    ```json
    {
      "success": true,
      "data": {
        "modules": [
          {
            "module_code": "MOD-ICU",
            "display_name": "ICU / Critical Care",
            "description": "ICU bed management",
            "depends_on": ["MOD-IPD"],
            "sort_order": 21,
            "activates_departments": ["ICU / Critical Care"]
          }
        ]
      }
    }
    ```

### 4.5 GET /admin/config/branches/{branch_id}/modules
Get the status of all modules at a specific branch.
*   **Success Response (`200 OK`)**:
    ```json
    {
      "success": true,
      "data": {
        "branch_id": "87fb739d-2ab3-4df4-bd9b-50cd6da8a113",
        "modules": [
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

### 4.6 POST /admin/config/branches/{branch_id}/modules/{module_code}/toggle
Toggle the activation of a module for a branch.
*   **Request JSON Body**:
    ```json
    {
      "is_active": true,
      "notes": "Enable ICU ward"
    }
    ```
*   **Success Response (`200 OK`)**:
    ```json
    {
      "success": true,
      "data": {
        "branch_id": "87fb739d-2ab3-4df4-bd9b-50cd6da8a113",
        "module_code": "MOD-ICU",
        "is_active": true,
        "enabled_at": "2026-06-15T10:07:00Z",
        "disabled_at": null,
        "departments_synced": ["ICU / Critical Care"]
      },
      "message": "Module MOD-ICU enabled. Synced departments successfully."
    }
    ```
*   **Error Response (`400 Bad Request`)**:
    ```json
    {
      "success": false,
      "code": 400,
      "message": "Dependency Error: Module MOD-ICU requires MOD-IPD to be active first."
    }
    ```
