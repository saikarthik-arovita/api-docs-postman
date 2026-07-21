# HMS — Lab Module Postman Collection Reference Guide
This document is automatically compiled from the [LAB_MODULE_POSTMAN_COLLECTION.json](file:///f:/LJB/ops-hms-ljb/postman/LAB_MODULE_POSTMAN_COLLECTION.json) collection file. It contains the exact HTTP method, path, headers, request bodies, and expected responses for every API endpoint.

## Folder: Laboratory Manager

### ➡️ List Lab Access Roles (GET /diagnostic-orders/roles)
* **HTTP Method**: `GET`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/roles`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ Create Lab Access Role (POST /diagnostic-orders/roles)
* **HTTP Method**: `POST`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/roles`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

#### Request Body:
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

---

### ➡️ List Audit Records (GET /diagnostic-orders/audit)
* **HTTP Method**: `GET`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/audit`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ Get Audit Details (GET /diagnostic-orders/audit/{id})
* **HTTP Method**: `GET`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/audit/{{id}}`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ Execute Audit Action (POST /diagnostic-orders/audit/actions)
* **HTTP Method**: `POST`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/audit/actions`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ Get Role Details (GET /diagnostic-orders/roles/{id})
* **HTTP Method**: `GET`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/roles/{{id}}`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ Update Role Details (PATCH /diagnostic-orders/roles/{id})
* **HTTP Method**: `PATCH`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/roles/{{id}}`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ Delete Role (DELETE /diagnostic-orders/roles/{id})
* **HTTP Method**: `DELETE`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/roles/{{id}}`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ Execute Role Action (POST /diagnostic-orders/roles/{id}/actions)
* **HTTP Method**: `POST`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/roles/{{id}}/actions`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ List Lab Invoices (GET /diagnostic-orders/billing/invoices)
* **HTTP Method**: `GET`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/billing/invoices`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ Get Billing Invoice (GET /diagnostic-orders/billing/invoices/{id})
* **HTTP Method**: `GET`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/billing/invoices/{{id}}`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ Update Billing Invoice (PATCH /diagnostic-orders/billing/invoices/{id})
* **HTTP Method**: `PATCH`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/billing/invoices/{{id}}`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ Execute Billing Invoice Action (POST /diagnostic-orders/billing/invoices/{id}/actions)
* **HTTP Method**: `POST`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/billing/invoices/{{id}}/actions`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ Create Lab Test (POST /diagnostic-orders/tests)
* **HTTP Method**: `POST`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/tests`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

#### Request Body:
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

---

### ➡️ List Config Settings (GET /diagnostic-orders/config)
* **HTTP Method**: `GET`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/config`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ Create Config Settings (POST /diagnostic-orders/config)
* **HTTP Method**: `POST`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/config`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

#### Request Body:
```json
{
  "templateName": "test_value",
  "category": "test_value",
  "layoutJson": "test_value"
}
```

---

### ➡️ Get Config Settings (GET /diagnostic-orders/config/{id})
* **HTTP Method**: `GET`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/config/{{id}}`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ Update Config Settings (PATCH /diagnostic-orders/config/{id})
* **HTTP Method**: `PATCH`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/config/{{id}}`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ Delete Config Settings (DELETE /diagnostic-orders/config/{id})
* **HTTP Method**: `DELETE`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/config/{{id}}`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ Execute Config Action (POST /diagnostic-orders/config/{id}/actions)
* **HTTP Method**: `POST`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/config/{{id}}/actions`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ Get Catalog Test (GET /diagnostic-orders/tests/{id})
* **HTTP Method**: `GET`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/tests/{{id}}`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ Update Catalog Test (PATCH /diagnostic-orders/tests/{id})
* **HTTP Method**: `PATCH`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/tests/{{id}}`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ Delete Catalog Test (DELETE /diagnostic-orders/tests/{id})
* **HTTP Method**: `DELETE`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/tests/{{id}}`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ List Lab Inventory (GET /diagnostic-orders/inventory)
* **HTTP Method**: `GET`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/inventory`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ Create Lab Item (POST /diagnostic-orders/inventory)
* **HTTP Method**: `POST`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/inventory`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

#### Request Body:
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

---

### ➡️ List Procurement Records (GET /diagnostic-orders/procurement)
* **HTTP Method**: `GET`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/procurement`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ Create Procurement Record (POST /diagnostic-orders/procurement)
* **HTTP Method**: `POST`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/procurement`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

#### Request Body:
```json
{
  "vendor_id": "test_value",
  "priority": "test_value",
  "lab_section": "test_value",
  "store_location": "test_value",
  "delivery_address": "test_value",
  "expected_delivery": "test_value",
  "notes": "test_value",
  "items": []
}
```

---

### ➡️ Get Procurement Record (GET /diagnostic-orders/procurement/{id})
* **HTTP Method**: `GET`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/procurement/{{id}}`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ Update Procurement Record (PATCH /diagnostic-orders/procurement/{id})
* **HTTP Method**: `PATCH`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/procurement/{{id}}`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ Execute Procurement Action (POST /diagnostic-orders/procurement/{id}/actions)
* **HTTP Method**: `POST`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/procurement/{{id}}/actions`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ Get Inventory Item (GET /diagnostic-orders/inventory/{id})
* **HTTP Method**: `GET`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/inventory/{{id}}`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ Update Inventory Item (PATCH /diagnostic-orders/inventory/{id})
* **HTTP Method**: `PATCH`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/inventory/{{id}}`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ Execute Inventory Action (POST /diagnostic-orders/inventory/{id}/actions)
* **HTTP Method**: `POST`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/inventory/{{id}}/actions`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ List Lab Staff (GET /diagnostic-orders/staff)
* **HTTP Method**: `GET`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/staff`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ Get Staff Profile (GET /diagnostic-orders/staff/{id})
* **HTTP Method**: `GET`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/staff/{{id}}`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ Update Staff Profile (PATCH /diagnostic-orders/staff/{id})
* **HTTP Method**: `PATCH`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/staff/{{id}}`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ Execute Staff Action (POST /diagnostic-orders/staff/{id}/actions)
* **HTTP Method**: `POST`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/staff/{{id}}/actions`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ List Lab Vendors (GET /diagnostic-orders/vendors)
* **HTTP Method**: `GET`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/vendors`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ Create Lab Vendor (POST /diagnostic-orders/vendors)
* **HTTP Method**: `POST`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/vendors`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

#### Request Body:
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

---

### ➡️ Get Vendor Profile (GET /diagnostic-orders/vendors/{id})
* **HTTP Method**: `GET`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/vendors/{{id}}`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ Update Vendor Profile (PATCH /diagnostic-orders/vendors/{id})
* **HTTP Method**: `PATCH`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/vendors/{{id}}`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ Delete Vendor Profile (DELETE /diagnostic-orders/vendors/{id})
* **HTTP Method**: `DELETE`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/vendors/{{id}}`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ Execute Vendor Action (POST /diagnostic-orders/vendors/{id}/actions)
* **HTTP Method**: `POST`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/vendors/{{id}}/actions`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

## Folder: Laboratory Registration Staff

### ➡️ List Lab Orders (GET /diagnostic-orders/orders)
* **HTTP Method**: `GET`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/orders`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ Create Registration (POST /diagnostic-orders/orders)
* **HTTP Method**: `POST`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/orders`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ Get Lookup Data (GET /diagnostic-orders/lookup)
* **HTTP Method**: `GET`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/lookup`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

## Folder: Laboratory Technician

### ➡️ Execute Order Action (POST /diagnostic-orders/orders/{id}/actions)
* **HTTP Method**: `POST`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/orders/{{id}}/actions`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---

### ➡️ Execute Orders Bulk Action (POST /diagnostic-orders/orders/bulk/actions)
* **HTTP Method**: `POST`
* **URL Endpoint**: `{{base_url}}/diagnostic-orders/orders/bulk/actions`

#### Request Headers:
| Header Key | Header Value | Description |
| :--- | :--- | :--- |
| `x-tenant-id` | `{{lab_id}}` |  |
| `Content-Type` | `application/json` |  |

---
