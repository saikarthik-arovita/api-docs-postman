# Laboratory Manager Dashboard API Specifications
Version: 1.0.0
Workspace Target: HMS Diagnostics Microservice
Enforced Scopes: config.read, config.write, billing.view, audit.read, staff.write, vendors.write

This document details every logically exposed API endpoint designed for the **Laboratory Manager Dashboard**. It covers required permissions, path/query variables, validation schemas, database tables utilized, request models, and standard response payloads.

---

## Get Config Settings (GET /diagnostic-orders/config/{id})
- **HTTP Method**: `GET`
- **URL Pattern**: `/diagnostic-orders/config/{id}`
- **Required Permission Scope**: `None`
- **Module**: `Config`
- **Description**: Endpoint handler for get config settings.

### Path Parameters:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | True | UUID reference for id |

### Expected Success Response (HTTP 200):
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `diagnostic.lab_tests, diagnostic.test_packages, diagnostic.machines, diagnostic.lab_settings, diagnostic.report_templates, diagnostic.barcode_settings, diagnostic.printers`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## List Config Settings (GET /diagnostic-orders/config)
- **HTTP Method**: `GET`
- **URL Pattern**: `/diagnostic-orders/config`
- **Required Permission Scope**: `None`
- **Module**: `Config`
- **Description**: Endpoint handler for list config settings.

### Expected Success Response (HTTP 200):
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `diagnostic.lab_tests, diagnostic.test_packages, diagnostic.machines, diagnostic.lab_settings, diagnostic.report_templates, diagnostic.barcode_settings, diagnostic.printers`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## Get Role Details (GET /diagnostic-orders/roles/{id})
- **HTTP Method**: `GET`
- **URL Pattern**: `/diagnostic-orders/roles/{id}`
- **Required Permission Scope**: `None`
- **Module**: `Audit`
- **Description**: Endpoint handler for get role details.

### Path Parameters:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | True | UUID reference for id |

### Expected Success Response (HTTP 200):
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `diagnostic.audit_logs, diagnostic.compliance_violations, diagnostic.access_roles, diagnostic.waste_records, diagnostic.waste_disposals, diagnostic.system_events`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## List Lab Access Roles (GET /diagnostic-orders/roles)
- **HTTP Method**: `GET`
- **URL Pattern**: `/diagnostic-orders/roles`
- **Required Permission Scope**: `roles.view`
- **Module**: `Audit`
- **Description**: Endpoint handler for list lab access roles.

### Expected Success Response (HTTP 200):
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `diagnostic.audit_logs, diagnostic.compliance_violations, diagnostic.access_roles, diagnostic.waste_records, diagnostic.waste_disposals, diagnostic.system_events`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## Create Config Settings (POST /diagnostic-orders/config)
- **HTTP Method**: `POST`
- **URL Pattern**: `/diagnostic-orders/config`
- **Required Permission Scope**: `None`
- **Module**: `Config`
- **Description**: Endpoint handler for create config settings.

### Expected Success Response (HTTP 201):
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `diagnostic.lab_tests, diagnostic.test_packages, diagnostic.machines, diagnostic.lab_settings, diagnostic.report_templates, diagnostic.barcode_settings, diagnostic.printers`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## Delete Config Settings (DELETE /diagnostic-orders/config/{id})
- **HTTP Method**: `DELETE`
- **URL Pattern**: `/diagnostic-orders/config/{id}`
- **Required Permission Scope**: `None`
- **Module**: `Config`
- **Description**: Endpoint handler for delete config settings.

### Path Parameters:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | True | UUID reference for id |

### Expected Success Response (HTTP 200):
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `diagnostic.lab_tests, diagnostic.test_packages, diagnostic.machines, diagnostic.lab_settings, diagnostic.report_templates, diagnostic.barcode_settings, diagnostic.printers`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## Execute Config Action (POST /diagnostic-orders/config/{id}/actions)
- **HTTP Method**: `POST`
- **URL Pattern**: `/diagnostic-orders/config/{id}/actions`
- **Required Permission Scope**: `None`
- **Module**: `Config`
- **Description**: Endpoint handler for execute config action.

### Path Parameters:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | True | UUID reference for id |

### Expected Success Response (HTTP 201):
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `diagnostic.lab_tests, diagnostic.test_packages, diagnostic.machines, diagnostic.lab_settings, diagnostic.report_templates, diagnostic.barcode_settings, diagnostic.printers`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## Update Config Settings (PATCH /diagnostic-orders/config/{id})
- **HTTP Method**: `PATCH`
- **URL Pattern**: `/diagnostic-orders/config/{id}`
- **Required Permission Scope**: `None`
- **Module**: `Config`
- **Description**: Endpoint handler for update config settings.

### Path Parameters:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | True | UUID reference for id |

### Expected Success Response (HTTP 200):
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `diagnostic.lab_tests, diagnostic.test_packages, diagnostic.machines, diagnostic.lab_settings, diagnostic.report_templates, diagnostic.barcode_settings, diagnostic.printers`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## Create Lab Test (POST /diagnostic-orders/tests)
- **HTTP Method**: `POST`
- **URL Pattern**: `/diagnostic-orders/tests`
- **Required Permission Scope**: `config.write`
- **Module**: `Config`
- **Description**: Endpoint handler for create lab test.

### Request Body Schema:
| Parameter | Type | Required? | Description |
|-----------|------|-----------|-------------|
| testName | string | Yes | Name of laboratory investigation |
| code | string | Yes | Unique test code configuration |
| departmentId | string | No | Department UUID reference |
| category | string | No | Hematology, Biochemistry, etc. |
| referenceRange | string | No | Standard reference range values |
| unit | string | No | Measurement units |
| sampleType | string | No | BLOOD, URINE, CSF, etc. |
| tat | integer | No | Turnaround time hours |
| price | number | No | Base rate price |

### Request Example:
```json
{
  "testName": "test_value",
  "code": "test_value",
  "departmentId": "test_value",
  "category": "test_value",
  "referenceRange": "test_value",
  "unit": "test_value",
  "sampleType": "test_value",
  "tat": 0,
  "price": 0.0
}
```

### Expected Success Response (HTTP 201):
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `diagnostic.lab_tests, diagnostic.test_packages, diagnostic.machines, diagnostic.lab_settings, diagnostic.report_templates, diagnostic.barcode_settings, diagnostic.printers`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## Delete Catalog Test (DELETE /diagnostic-orders/tests/{id})
- **HTTP Method**: `DELETE`
- **URL Pattern**: `/diagnostic-orders/tests/{id}`
- **Required Permission Scope**: `None`
- **Module**: `Config`
- **Description**: Endpoint handler for delete catalog test.

### Path Parameters:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | True | UUID reference for id |

### Expected Success Response (HTTP 200):
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `diagnostic.lab_tests, diagnostic.test_packages, diagnostic.machines, diagnostic.lab_settings, diagnostic.report_templates, diagnostic.barcode_settings, diagnostic.printers`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## Update Catalog Test (PATCH /diagnostic-orders/tests/{id})
- **HTTP Method**: `PATCH`
- **URL Pattern**: `/diagnostic-orders/tests/{id}`
- **Required Permission Scope**: `None`
- **Module**: `Config`
- **Description**: Endpoint handler for update catalog test.

### Path Parameters:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | True | UUID reference for id |

### Expected Success Response (HTTP 200):
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `diagnostic.lab_tests, diagnostic.test_packages, diagnostic.machines, diagnostic.lab_settings, diagnostic.report_templates, diagnostic.barcode_settings, diagnostic.printers`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## Create Lab Access Role (POST /diagnostic-orders/roles)
- **HTTP Method**: `POST`
- **URL Pattern**: `/diagnostic-orders/roles`
- **Required Permission Scope**: `roles.write`
- **Module**: `Audit`
- **Description**: Endpoint handler for create lab access role.

### Request Body Schema:
| Parameter | Type | Required? | Description |
|-----------|------|-----------|-------------|
| roleName | string | Yes | The name of the custom role |
| description | string | No | Detailed description of role responsibilities |
| status | string | No | ACTIVE or INACTIVE |
| maxUsers | integer | No | Maximum users allowed to hold this role |
| permissions | array | No | Array of permission names assigned to role |
| sessionTimeout | integer | No | Session timeout in minutes |
| mfaRequired | boolean | No | MFA required flag |

### Request Example:
```json
{
  "roleName": "test_value",
  "description": "test_value",
  "status": "test_value",
  "maxUsers": 0,
  "permissions": [],
  "sessionTimeout": 0,
  "mfaRequired": false
}
```

### Expected Success Response (HTTP 201):
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `diagnostic.audit_logs, diagnostic.compliance_violations, diagnostic.access_roles, diagnostic.waste_records, diagnostic.waste_disposals, diagnostic.system_events`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## Delete Role (DELETE /diagnostic-orders/roles/{id})
- **HTTP Method**: `DELETE`
- **URL Pattern**: `/diagnostic-orders/roles/{id}`
- **Required Permission Scope**: `None`
- **Module**: `Audit`
- **Description**: Endpoint handler for delete role.

### Path Parameters:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | True | UUID reference for id |

### Expected Success Response (HTTP 200):
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `diagnostic.audit_logs, diagnostic.compliance_violations, diagnostic.access_roles, diagnostic.waste_records, diagnostic.waste_disposals, diagnostic.system_events`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## Execute Role Action (POST /diagnostic-orders/roles/{id}/actions)
- **HTTP Method**: `POST`
- **URL Pattern**: `/diagnostic-orders/roles/{id}/actions`
- **Required Permission Scope**: `None`
- **Module**: `Audit`
- **Description**: Endpoint handler for execute role action.

### Path Parameters:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | True | UUID reference for id |

### Expected Success Response (HTTP 201):
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `diagnostic.audit_logs, diagnostic.compliance_violations, diagnostic.access_roles, diagnostic.waste_records, diagnostic.waste_disposals, diagnostic.system_events`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## Update Role Details (PATCH /diagnostic-orders/roles/{id})
- **HTTP Method**: `PATCH`
- **URL Pattern**: `/diagnostic-orders/roles/{id}`
- **Required Permission Scope**: `None`
- **Module**: `Audit`
- **Description**: Endpoint handler for update role details.

### Path Parameters:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | True | UUID reference for id |

### Expected Success Response (HTTP 200):
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `diagnostic.audit_logs, diagnostic.compliance_violations, diagnostic.access_roles, diagnostic.waste_records, diagnostic.waste_disposals, diagnostic.system_events`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## Create Lab Item (POST /diagnostic-orders/inventory)
- **HTTP Method**: `POST`
- **URL Pattern**: `/diagnostic-orders/inventory`
- **Required Permission Scope**: `diagnostics:test:manage`
- **Module**: `Manager`
- **Description**: Endpoint handler for create lab item.

### Request Body Schema:
| Parameter | Type | Required? | Description |
|-----------|------|-----------|-------------|
| item_code | string | Yes | Field description |
| item_name | string | Yes | Field description |
| category | string | Yes | Field description |
| sub_category | string | No | Field description |
| description | string | No | Field description |
| cas_number | string | No | Field description |
| molecular_formula | string | No | Field description |
| concentration | string | No | Field description |
| purity_grade | string | No | Field description |
| hazard_class | string | No | Field description |
| msds_available | boolean | No | Field description |
| requires_coc | boolean | No | Field description |
| uom | string | No | Field description |
| pack_size | string | No | Field description |
| tests_per_unit | integer | No | Field description |
| storage_condition | string | No | Field description |
| storage_temperature_min | number | No | Field description |
| storage_temperature_max | number | No | Field description |
| is_light_sensitive | boolean | No | Field description |
| is_moisture_sensitive | boolean | No | Field description |
| shelf_life_months | integer | No | Field description |
| brand | string | No | Field description |
| manufacturer | string | No | Field description |
| catalogue_number | string | No | Field description |
| barcode | string | No | Field description |
| preferred_vendor_id | string | No | Field description |
| mrp | number | No | Field description |
| gst_percent | number | No | Field description |
| lead_time_days | integer | No | Field description |
| reorder_level | integer | No | Field description |
| minimum_stock | integer | No | Field description |
| maximum_stock | integer | No | Field description |
| warehouse_id | string | No | Field description |
| warehouse_name | string | No | Field description |
| rack_shelf_bin | string | No | Field description |
| assigned_department_id | string | No | Field description |
| assigned_section | string | No | Field description |
| linked_equipment_name | string | No | Field description |
| linked_equipment_model | string | No | Field description |
| is_controlled | boolean | No | Field description |
| regulatory_approval | string | No | Field description |
| recall_tracking | boolean | No | Field description |

### Request Example:
```json
{
  "item_code": "test_value",
  "item_name": "test_value",
  "category": "test_value",
  "sub_category": "test_value",
  "description": "test_value",
  "cas_number": "test_value",
  "molecular_formula": "test_value",
  "concentration": "test_value",
  "purity_grade": "test_value",
  "hazard_class": "test_value",
  "msds_available": false,
  "requires_coc": false,
  "uom": "test_value",
  "pack_size": "test_value",
  "tests_per_unit": 0,
  "storage_condition": "test_value",
  "storage_temperature_min": 0.0,
  "storage_temperature_max": 0.0,
  "is_light_sensitive": false,
  "is_moisture_sensitive": false,
  "shelf_life_months": 0,
  "brand": "test_value",
  "manufacturer": "test_value",
  "catalogue_number": "test_value",
  "barcode": "test_value",
  "preferred_vendor_id": "test_value",
  "mrp": 0.0,
  "gst_percent": 0.0,
  "lead_time_days": 0,
  "reorder_level": 0,
  "minimum_stock": 0,
  "maximum_stock": 0,
  "warehouse_id": "test_value",
  "warehouse_name": "test_value",
  "rack_shelf_bin": "test_value",
  "assigned_department_id": "test_value",
  "assigned_section": "test_value",
  "linked_equipment_name": "test_value",
  "linked_equipment_model": "test_value",
  "is_controlled": false,
  "regulatory_approval": "test_value",
  "recall_tracking": false
}
```

### Expected Success Response (HTTP 201):
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `diagnostic.lab_inventory, diagnostic.lab_inventory_batches, diagnostic.lab_inventory_mappings, diagnostic.lab_qc_records`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## Execute Inventory Action (POST /diagnostic-orders/inventory/{id}/actions)
- **HTTP Method**: `POST`
- **URL Pattern**: `/diagnostic-orders/inventory/{id}/actions`
- **Required Permission Scope**: `None`
- **Module**: `Manager`
- **Description**: Endpoint handler for execute inventory action.

### Path Parameters:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | True | UUID reference for id |

### Expected Success Response (HTTP 201):
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `diagnostic.lab_inventory, diagnostic.lab_inventory_batches, diagnostic.lab_inventory_mappings, diagnostic.lab_qc_records`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## Get Inventory Item (GET /diagnostic-orders/inventory/{id})
- **HTTP Method**: `GET`
- **URL Pattern**: `/diagnostic-orders/inventory/{id}`
- **Required Permission Scope**: `None`
- **Module**: `Manager`
- **Description**: Endpoint handler for get inventory item.

### Path Parameters:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | True | UUID reference for id |

### Expected Success Response (HTTP 200):
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `diagnostic.lab_inventory, diagnostic.lab_inventory_batches, diagnostic.lab_inventory_mappings, diagnostic.lab_qc_records`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## List Lab Inventory (GET /diagnostic-orders/inventory)
- **HTTP Method**: `GET`
- **URL Pattern**: `/diagnostic-orders/inventory`
- **Required Permission Scope**: `diagnostics:order:view`
- **Module**: `Manager`
- **Description**: Endpoint handler for list lab inventory.

### Expected Success Response (HTTP 200):
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `diagnostic.lab_inventory, diagnostic.lab_inventory_batches, diagnostic.lab_inventory_mappings, diagnostic.lab_qc_records`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## Update Inventory Item (PATCH /diagnostic-orders/inventory/{id})
- **HTTP Method**: `PATCH`
- **URL Pattern**: `/diagnostic-orders/inventory/{id}`
- **Required Permission Scope**: `None`
- **Module**: `Manager`
- **Description**: Endpoint handler for update inventory item.

### Path Parameters:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | True | UUID reference for id |

### Expected Success Response (HTTP 200):
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `diagnostic.lab_inventory, diagnostic.lab_inventory_batches, diagnostic.lab_inventory_mappings, diagnostic.lab_qc_records`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## Create Procurement Record (POST /diagnostic-orders/procurement)
- **HTTP Method**: `POST`
- **URL Pattern**: `/diagnostic-orders/procurement`
- **Required Permission Scope**: `None`
- **Module**: `Manager`
- **Description**: Endpoint handler for create procurement record.

### Expected Success Response (HTTP 201):
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `diagnostic.lab_inventory, diagnostic.lab_inventory_batches, diagnostic.lab_inventory_mappings, diagnostic.lab_qc_records`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## Execute Procurement Action (POST /diagnostic-orders/procurement/{id}/actions)
- **HTTP Method**: `POST`
- **URL Pattern**: `/diagnostic-orders/procurement/{id}/actions`
- **Required Permission Scope**: `None`
- **Module**: `Manager`
- **Description**: Endpoint handler for execute procurement action.

### Path Parameters:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | True | UUID reference for id |

### Expected Success Response (HTTP 201):
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `diagnostic.lab_inventory, diagnostic.lab_inventory_batches, diagnostic.lab_inventory_mappings, diagnostic.lab_qc_records`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## Get Procurement Record (GET /diagnostic-orders/procurement/{id})
- **HTTP Method**: `GET`
- **URL Pattern**: `/diagnostic-orders/procurement/{id}`
- **Required Permission Scope**: `None`
- **Module**: `Manager`
- **Description**: Endpoint handler for get procurement record.

### Path Parameters:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | True | UUID reference for id |

### Expected Success Response (HTTP 200):
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `diagnostic.lab_inventory, diagnostic.lab_inventory_batches, diagnostic.lab_inventory_mappings, diagnostic.lab_qc_records`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## List Procurement Records (GET /diagnostic-orders/procurement)
- **HTTP Method**: `GET`
- **URL Pattern**: `/diagnostic-orders/procurement`
- **Required Permission Scope**: `None`
- **Module**: `Manager`
- **Description**: Endpoint handler for list procurement records.

### Expected Success Response (HTTP 200):
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `diagnostic.lab_inventory, diagnostic.lab_inventory_batches, diagnostic.lab_inventory_mappings, diagnostic.lab_qc_records`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## Update Procurement Record (PATCH /diagnostic-orders/procurement/{id})
- **HTTP Method**: `PATCH`
- **URL Pattern**: `/diagnostic-orders/procurement/{id}`
- **Required Permission Scope**: `None`
- **Module**: `Manager`
- **Description**: Endpoint handler for update procurement record.

### Path Parameters:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | True | UUID reference for id |

### Expected Success Response (HTTP 200):
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `diagnostic.lab_inventory, diagnostic.lab_inventory_batches, diagnostic.lab_inventory_mappings, diagnostic.lab_qc_records`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## Create Lab Vendor (POST /diagnostic-orders/vendors)
- **HTTP Method**: `POST`
- **URL Pattern**: `/diagnostic-orders/vendors`
- **Required Permission Scope**: `diagnostics:test:manage`
- **Module**: `Vendors`
- **Description**: Endpoint handler for create lab vendor.

### Request Body Schema:
| Parameter | Type | Required? | Description |
|-----------|------|-----------|-------------|
| vendor_name | string | Yes | Field description |
| vendor_code | string | No | Field description |
| vendor_type | string | No | Field description |
| vendor_category | string | No | Field description |
| business_type | string | No | Field description |
| year_established | integer | No | Field description |
| website_url | string | No | Field description |
| company_description | string | No | Field description |
| primary_contact | string | No | Field description |
| designation | string | No | Field description |
| mobile | string | No | Field description |
| alternate_mobile | string | No | Field description |
| official_email | string | Yes | Field description |
| support_email | string | No | Field description |
| landline | string | No | Field description |
| whatsapp | string | No | Field description |
| office_address | string | No | Field description |
| city | string | No | Field description |
| state | string | No | Field description |
| pin_code | string | No | Field description |
| gst_number | string | No | Field description |
| pan_number | string | No | Field description |
| cin_number | string | No | Field description |
| drug_license_number | string | No | Field description |
| drug_license_expiry | string | No | Field description |
| bank_name | string | No | Field description |
| account_holder | string | No | Field description |
| account_number | string | No | Field description |
| ifsc_code | string | No | Field description |
| branch_name | string | No | Field description |
| upi_id | string | No | Field description |
| payment_terms | string | No | Field description |
| credit_period_days | integer | No | Field description |
| preferred_payment_mode | string | No | Field description |
| advance_payment_allowed | boolean | No | Field description |
| delivery_coverage | string | No | Field description |
| avg_delivery_hours | integer | No | Field description |
| min_order_value | number | No | Field description |
| emergency_supply | boolean | No | Field description |
| cold_chain_supported | boolean | No | Field description |
| installation_support | boolean | No | Field description |
| amc_support | boolean | No | Field description |
| nabl_iso_certified | boolean | No | Field description |
| gmp_certified | boolean | No | Field description |
| fda_approved | boolean | No | Field description |
| biomedical_waste_compliance | boolean | No | Field description |
| risk_level | string | No | Field description |
| approval_authority | string | No | Field description |
| procurement_notes | string | No | Field description |
| vendor_status | string | No | Field description |
| is_active | boolean | No | Field description |

### Request Example:
```json
{
  "vendor_name": "test_value",
  "vendor_code": "test_value",
  "vendor_type": "test_value",
  "vendor_category": "test_value",
  "business_type": "test_value",
  "year_established": 0,
  "website_url": "test_value",
  "company_description": "test_value",
  "primary_contact": "test_value",
  "designation": "test_value",
  "mobile": "test_value",
  "alternate_mobile": "test_value",
  "official_email": "test_value",
  "support_email": "test_value",
  "landline": "test_value",
  "whatsapp": "test_value",
  "office_address": "test_value",
  "city": "test_value",
  "state": "test_value",
  "pin_code": "test_value",
  "gst_number": "test_value",
  "pan_number": "test_value",
  "cin_number": "test_value",
  "drug_license_number": "test_value",
  "drug_license_expiry": "test_value",
  "bank_name": "test_value",
  "account_holder": "test_value",
  "account_number": "test_value",
  "ifsc_code": "test_value",
  "branch_name": "test_value",
  "upi_id": "test_value",
  "payment_terms": "test_value",
  "credit_period_days": 0,
  "preferred_payment_mode": "test_value",
  "advance_payment_allowed": false,
  "delivery_coverage": "test_value",
  "avg_delivery_hours": 0,
  "min_order_value": 0.0,
  "emergency_supply": false,
  "cold_chain_supported": false,
  "installation_support": false,
  "amc_support": false,
  "nabl_iso_certified": false,
  "gmp_certified": false,
  "fda_approved": false,
  "biomedical_waste_compliance": false,
  "risk_level": "test_value",
  "approval_authority": "test_value",
  "procurement_notes": "test_value",
  "vendor_status": "test_value",
  "is_active": false
}
```

### Expected Success Response (HTTP 201):
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `pharmacy.vendors, pharmacy.vendor_contracts, pharmacy.vendor_documents, pharmacy.vendor_performance`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## Delete Vendor Profile (DELETE /diagnostic-orders/vendors/{id})
- **HTTP Method**: `DELETE`
- **URL Pattern**: `/diagnostic-orders/vendors/{id}`
- **Required Permission Scope**: `None`
- **Module**: `Vendors`
- **Description**: Endpoint handler for delete vendor profile.

### Path Parameters:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | True | UUID reference for id |

### Expected Success Response (HTTP 200):
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `pharmacy.vendors, pharmacy.vendor_contracts, pharmacy.vendor_documents, pharmacy.vendor_performance`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## Execute Vendor Action (POST /diagnostic-orders/vendors/{id}/actions)
- **HTTP Method**: `POST`
- **URL Pattern**: `/diagnostic-orders/vendors/{id}/actions`
- **Required Permission Scope**: `None`
- **Module**: `Vendors`
- **Description**: Endpoint handler for execute vendor action.

### Path Parameters:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | True | UUID reference for id |

### Expected Success Response (HTTP 201):
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `pharmacy.vendors, pharmacy.vendor_contracts, pharmacy.vendor_documents, pharmacy.vendor_performance`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## Get Vendor Profile (GET /diagnostic-orders/vendors/{id})
- **HTTP Method**: `GET`
- **URL Pattern**: `/diagnostic-orders/vendors/{id}`
- **Required Permission Scope**: `None`
- **Module**: `Vendors`
- **Description**: Endpoint handler for get vendor profile.

### Path Parameters:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | True | UUID reference for id |

### Expected Success Response (HTTP 200):
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `pharmacy.vendors, pharmacy.vendor_contracts, pharmacy.vendor_documents, pharmacy.vendor_performance`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## List Lab Vendors (GET /diagnostic-orders/vendors)
- **HTTP Method**: `GET`
- **URL Pattern**: `/diagnostic-orders/vendors`
- **Required Permission Scope**: `diagnostics:order:view`
- **Module**: `Vendors`
- **Description**: Endpoint handler for list lab vendors.

### Expected Success Response (HTTP 200):
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `pharmacy.vendors, pharmacy.vendor_contracts, pharmacy.vendor_documents, pharmacy.vendor_performance`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## Update Vendor Profile (PATCH /diagnostic-orders/vendors/{id})
- **HTTP Method**: `PATCH`
- **URL Pattern**: `/diagnostic-orders/vendors/{id}`
- **Required Permission Scope**: `None`
- **Module**: `Vendors`
- **Description**: Endpoint handler for update vendor profile.

### Path Parameters:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | True | UUID reference for id |

### Expected Success Response (HTTP 200):
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `pharmacy.vendors, pharmacy.vendor_contracts, pharmacy.vendor_documents, pharmacy.vendor_performance`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## Execute Billing Invoice Action (POST /diagnostic-orders/billing/invoices/{id}/actions)
- **HTTP Method**: `POST`
- **URL Pattern**: `/diagnostic-orders/billing/invoices/{id}/actions`
- **Required Permission Scope**: `None`
- **Module**: `Billing`
- **Description**: Endpoint handler for execute billing invoice action.

### Path Parameters:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | True | UUID reference for id |

### Expected Success Response (HTTP 201):
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `revenue.bills, revenue.billing_items, revenue.billing_refunds`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## Get Billing Invoice (GET /diagnostic-orders/billing/invoices/{id})
- **HTTP Method**: `GET`
- **URL Pattern**: `/diagnostic-orders/billing/invoices/{id}`
- **Required Permission Scope**: `None`
- **Module**: `Billing`
- **Description**: Endpoint handler for get billing invoice.

### Path Parameters:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | True | UUID reference for id |

### Expected Success Response (HTTP 200):
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `revenue.bills, revenue.billing_items, revenue.billing_refunds`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## List Lab Invoices (GET /diagnostic-orders/billing/invoices)
- **HTTP Method**: `GET`
- **URL Pattern**: `/diagnostic-orders/billing/invoices`
- **Required Permission Scope**: `diagnostics:order:view`
- **Module**: `Manager`
- **Description**: Endpoint handler for list lab invoices.

### Expected Success Response (HTTP 200):
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `diagnostic.lab_inventory, diagnostic.lab_inventory_batches, diagnostic.lab_inventory_mappings, diagnostic.lab_qc_records`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## Update Billing Invoice (PATCH /diagnostic-orders/billing/invoices/{id})
- **HTTP Method**: `PATCH`
- **URL Pattern**: `/diagnostic-orders/billing/invoices/{id}`
- **Required Permission Scope**: `None`
- **Module**: `Billing`
- **Description**: Endpoint handler for update billing invoice.

### Path Parameters:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | True | UUID reference for id |

### Expected Success Response (HTTP 200):
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `revenue.bills, revenue.billing_items, revenue.billing_refunds`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## Execute Audit Action (POST /diagnostic-orders/audit/actions)
- **HTTP Method**: `POST`
- **URL Pattern**: `/diagnostic-orders/audit/actions`
- **Required Permission Scope**: `None`
- **Module**: `Audit`
- **Description**: Endpoint handler for execute audit action.

### Expected Success Response (HTTP 201):
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `diagnostic.audit_logs, diagnostic.compliance_violations, diagnostic.access_roles, diagnostic.waste_records, diagnostic.waste_disposals, diagnostic.system_events`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## Get Audit Details (GET /diagnostic-orders/audit/{id})
- **HTTP Method**: `GET`
- **URL Pattern**: `/diagnostic-orders/audit/{id}`
- **Required Permission Scope**: `None`
- **Module**: `Audit`
- **Description**: Endpoint handler for get audit details.

### Path Parameters:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | True | UUID reference for id |

### Expected Success Response (HTTP 200):
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `diagnostic.audit_logs, diagnostic.compliance_violations, diagnostic.access_roles, diagnostic.waste_records, diagnostic.waste_disposals, diagnostic.system_events`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## List Audit Records (GET /diagnostic-orders/audit)
- **HTTP Method**: `GET`
- **URL Pattern**: `/diagnostic-orders/audit`
- **Required Permission Scope**: `None`
- **Module**: `Audit`
- **Description**: Endpoint handler for list audit records.

### Expected Success Response (HTTP 200):
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `diagnostic.audit_logs, diagnostic.compliance_violations, diagnostic.access_roles, diagnostic.waste_records, diagnostic.waste_disposals, diagnostic.system_events`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## Execute Staff Action (POST /diagnostic-orders/staff/{id}/actions)
- **HTTP Method**: `POST`
- **URL Pattern**: `/diagnostic-orders/staff/{id}/actions`
- **Required Permission Scope**: `None`
- **Module**: `Staff`
- **Description**: Endpoint handler for execute staff action.

### Path Parameters:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | True | UUID reference for id |

### Expected Success Response (HTTP 201):
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `diagnostic.staff_assignments, diagnostic.staff_shifts, diagnostic.shift_swaps, diagnostic.staff_attendance`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## Get Catalog Test (GET /diagnostic-orders/tests/{id})
- **HTTP Method**: `GET`
- **URL Pattern**: `/diagnostic-orders/tests/{id}`
- **Required Permission Scope**: `None`
- **Module**: `Config`
- **Description**: Endpoint handler for get catalog test.

### Path Parameters:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | True | UUID reference for id |

### Expected Success Response (HTTP 200):
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `diagnostic.lab_tests, diagnostic.test_packages, diagnostic.machines, diagnostic.lab_settings, diagnostic.report_templates, diagnostic.barcode_settings, diagnostic.printers`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## Get Staff Profile (GET /diagnostic-orders/staff/{id})
- **HTTP Method**: `GET`
- **URL Pattern**: `/diagnostic-orders/staff/{id}`
- **Required Permission Scope**: `None`
- **Module**: `Staff`
- **Description**: Endpoint handler for get staff profile.

### Path Parameters:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | True | UUID reference for id |

### Expected Success Response (HTTP 200):
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `diagnostic.staff_assignments, diagnostic.staff_shifts, diagnostic.shift_swaps, diagnostic.staff_attendance`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## List Lab Staff (GET /diagnostic-orders/staff)
- **HTTP Method**: `GET`
- **URL Pattern**: `/diagnostic-orders/staff`
- **Required Permission Scope**: `staff.read`
- **Module**: `Staff`
- **Description**: Endpoint handler for list lab staff.

### Expected Success Response (HTTP 200):
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `diagnostic.staff_assignments, diagnostic.staff_shifts, diagnostic.shift_swaps, diagnostic.staff_attendance`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

## Update Staff Profile (PATCH /diagnostic-orders/staff/{id})
- **HTTP Method**: `PATCH`
- **URL Pattern**: `/diagnostic-orders/staff/{id}`
- **Required Permission Scope**: `None`
- **Module**: `Staff`
- **Description**: Endpoint handler for update staff profile.

### Path Parameters:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | True | UUID reference for id |

### Expected Success Response (HTTP 200):
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "bff7e889-ff76-463a-ae62-386e97802b26",
    "status": "COMPLETED",
    "updated_at": "2026-07-15T10:00:00Z"
  }
}
```

### Database & Security Context:
- **Database Tables**: `diagnostic.staff_assignments, diagnostic.staff_shifts, diagnostic.shift_swaps, diagnostic.staff_attendance`
- **Audit Event Registered**: Yes, triggers a system event log through the unified `AuditEventLogger` context.

---

