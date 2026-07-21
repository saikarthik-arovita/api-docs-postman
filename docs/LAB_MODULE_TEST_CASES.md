# Laboratory Module API QA Test Cases Specification
Version: 1.0.0
Author: Senior QA Engineer

---

## 1. Role Permission Matrix Verification
Verify that users can access resources matching their JWT Cognito client scopes. All unauthorized requests must return `403 Forbidden` response templates.

| Test Case ID | Endpoint URL | Role | Header Permissions | Expected Status | Action/Details |
|--------------|--------------|------|--------------------|-----------------|----------------|
| **QA-AUTH-001** | `/diagnostic-orders/lab/orders` | Reg Staff | `diagnostics:order:view` | `200 OK` | Access order list successfully |
| **QA-AUTH-002** | `/lab/config/tests` | Reg Staff | `diagnostics:order:view` | `403 Forbidden` | Registration staff cannot access config catalog |
| **QA-AUTH-003** | `/lab/config/tests` | Lab Manager | `config.write` | `201 Created` | Manager creates new lab tests |
| **QA-AUTH-004** | `/lab/results/{id}/status` | Technician | `diagnostics:technician:write` | `403 Forbidden` | Technician cannot sign off/validate results |

---

## 2. Dynamic Input Validation Boundaries
Verify that invalid data, bad UUIDs, and missing fields return `422 Unprocessable Entity` or `400 Bad Request`.

* **Case ID**: **QA-VAL-101**
  * **Endpoint**: `POST /diagnostic-orders/lab/registrations`
  * **Payload**: `{"patient_id": "invalid-uuid", "doctor_id": "8b7f50c4-fa3a-4aef-ab27-f42b7fb0e54f"}`
  * **Assert**: Response is `400 Bad Request` or `422 Unprocessable Entity` with validation detail messages pointing to invalid UUID formatting.

* **Case ID**: **QA-VAL-102**
  * **Endpoint**: `POST /lab/config/tests`
  * **Payload**: `{"testName": "Hb", "code": "H"}`
  * **Assert**: Response contains parameter validation failure details due to code length being under the minimum 2-character requirement.

---

## 3. Database State Post-Assert Rules
Ensure database tables are correctly populated after write actions:

* **Case ID**: **QA-DB-201**
  * **Action**: `POST /diagnostic-orders/lab/registrations`
  * **Verify**: Run PostgreSQL select query on `diagnostic.diagnostic_orders` where patient_id matches payload. Ensure is_active is `True` and status is `IN_PROGRESS`.

* **Case ID**: **QA-DB-202**
  * **Action**: `PATCH /lab/results/{id}/status`
  * **Verify**: Verify that the validated record exists in `diagnostic.test_results` and `validated_by` maps to the pathologist user UUID.
