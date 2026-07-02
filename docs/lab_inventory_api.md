# Laboratory Inventory & Procurement API Documentation

Comprehensive API documentation for managing laboratory catalogs, lot batches, quality control records, test mappings, Purchase Orders, Goods Received Notes, and Vendor Invoices.

---

## Part 1: Laboratory Catalog & Stock Endpoints

### 1. List Lab Inventory
List items registered in the lab item master.

* **Route**: `GET /diagnostic-orders/lab/inventory`
* **Permission Guard**: `diagnostics:order:view`

#### Request Headers
| Header | Type | Value | Mandatory |
| :--- | :--- | :--- | :--- |
| `Authorization` | String | `Bearer {{token}}` | Yes |

#### Query Parameters
| Parameter | Type | Description | Mandatory |
| :--- | :--- | :--- | :--- |
| `search` | String | Search query matched against item name or code | No |
| `category` | String | Filter by category (e.g. `REAGENT`, `CONSUMABLE`) | No |
| `section` | String | Filter by lab section (e.g. `Haematology`, `Microbiology`) | No |

#### Example Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "id": "df86d618-0977-4579-a061-e75b1d14d4dd",
      "tenant_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
      "item_code": "LAB-RE-CBC-001",
      "item_name": "Hematology Reagent Diluent",
      "category": "REAGENT",
      "sub_category": null,
      "description": null,
      "hazard_class": null,
      "uom": "BOTTLE",
      "pack_size": null,
      "tests_per_unit": null,
      "current_stock": "10.0000",
      "reorder_level": "10.0000",
      "minimum_stock": "5.0000",
      "assigned_section": "Haematology",
      "is_active": true,
      "created_at": "2026-06-29 13:20:10.634640+05:30",
      "updated_at": "2026-06-29 13:20:25.257243+05:30"
    }
  ]
}
```

---

### 2. Create Lab Catalog Item
Register a new master reagent or supply item.

* **Route**: `POST /diagnostic-orders/lab/inventory`
* **Permission Guard**: `diagnostics:test:manage`

#### Request Headers
| Header | Type | Value | Mandatory |
| :--- | :--- | :--- | :--- |
| `Authorization` | String | `Bearer {{token}}` | Yes |
| `Content-Type` | String | `application/json` | Yes |

#### Request Body (JSON)
| Attribute | Type | Description | Mandatory |
| :--- | :--- | :--- | :--- |
| `item_code` | String | Unique code identifier (e.g., `LAB-RE-CBC-001`) | Yes |
| `item_name` | String | Descriptive name | Yes |
| `category` | String | Item category (e.g. `REAGENT`, `EQUIPMENT_SPARE`) | Yes |
| `assigned_section` | String | Laboratory section (e.g. `Haematology`, `Pathology`) | Yes |
| `uom` | String | Unit of measure (e.g. `BOTTLE`, `KIT`, `STRIPS`) | Yes |
| `reorder_level` | Numeric | Reorder threshold level | No (default `0`) |
| `minimum_stock` | Numeric | Minimum safe stock level | No (default `0`) |
| `sub_category` | String | Sub-category | No |
| `description` | String | Description | No |
| `hazard_class` | String | MSDS hazard classification (e.g., `CORROSIVE`) | No |
| `pack_size` | String | Description of box/pack content details | No |
| `tests_per_unit` | Integer | Calculated tests supported by one UOM unit | No |

#### Example Request Body
```json
{
  "item_code": "LAB-RE-CBC-001",
  "item_name": "Hematology Reagent Diluent",
  "category": "REAGENT",
  "assigned_section": "Haematology",
  "uom": "BOTTLE",
  "reorder_level": 10,
  "minimum_stock": 5
}
```

#### Example Response (`201 Created`)
```json
{
  "success": true,
  "code": 201,
  "message": "Lab inventory item created successfully",
  "data": {
    "id": "df86d618-0977-4579-a061-e75b1d14d4dd",
    "item_code": "LAB-RE-CBC-001",
    "item_name": "Hematology Reagent Diluent",
    "category": "REAGENT",
    "assigned_section": "Haematology",
    "uom": "BOTTLE",
    "reorder_level": 10.0,
    "minimum_stock": 5.0
  }
}
```

---

### 3. Get Stock Alerts
Fetch stock health analytics and warning thresholds.

* **Route**: `GET /diagnostic-orders/lab/inventory/alerts`
* **Permission Guard**: `diagnostics:dashboard`

#### Request Headers
| Header | Type | Value | Mandatory |
| :--- | :--- | :--- | :--- |
| `Authorization` | String | `Bearer {{token}}` | Yes |

#### Example Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "stock_health_percentage": 0,
    "items": [
      {
        "item_name": "Hematology Reagent Diluent",
        "status": "WARNING",
        "current_stock": "10.0000",
        "min_stock": "5.0000",
        "uom": "BOTTLE"
      }
    ]
  }
}
```

---

### 4. Create Lab Test Mapping
Link an inventory reagent to a clinical lab test for automatic validation-time deduction.

* **Route**: `POST /diagnostic-orders/lab/inventory/mappings`
* **Permission Guard**: `diagnostics:test:manage`

#### Request Headers
| Header | Type | Value | Mandatory |
| :--- | :--- | :--- | :--- |
| `Authorization` | String | `Bearer {{token}}` | Yes |
| `Content-Type` | String | `application/json` | Yes |

#### Request Body (JSON)
| Attribute | Type | Description | Mandatory |
| :--- | :--- | :--- | :--- |
| `item_id` | String (UUID) | Master inventory item ID | Yes |
| `test_id` | String (UUID) | Clinical lab test catalog ID | Yes |
| `qty_per_test` | Numeric | Quantity consumed per single test run | Yes |
| `is_optional` | Boolean | True if the reagent is not mandatory for execution | No (default `false`) |

#### Example Request Body
```json
{
  "item_id": "df86d618-0977-4579-a061-e75b1d14d4dd",
  "test_id": "16a43dca-eb71-4b85-ab0a-3b984816cadf",
  "qty_per_test": 1.5,
  "is_optional": false
}
```

#### Example Response (`201 Created`)
```json
{
  "success": true,
  "code": 201,
  "message": "Lab item test mapping created successfully",
  "data": {
    "id": "a0c416f8-fa7c-4966-b343-e5794e0c3384",
    "item_id": "df86d618-0977-4579-a061-e75b1d14d4dd",
    "test_id": "16a43dca-eb71-4b85-ab0a-3b984816cadf",
    "qty_per_test": 1.5,
    "is_optional": false
  }
}
```

---

### 5. Consume Lab Items
Manually deduct stock for wastage, training, or manual runs.

* **Route**: `POST /diagnostic-orders/lab/inventory/consume`
* **Permission Guard**: `diagnostics:lab:collect`

#### Request Headers
| Header | Type | Value | Mandatory |
| :--- | :--- | :--- | :--- |
| `Authorization` | String | `Bearer {{token}}` | Yes |
| `Content-Type` | String | `application/json` | Yes |

#### Request Body (JSON)
| Attribute | Type | Description | Mandatory |
| :--- | :--- | :--- | :--- |
| `consumption_type` | String | Consumption code (`MANUAL`, `WASTAGE`, `QC`, `TEST`) | Yes |
| `items` | Array | Consumed item details | Yes |
| `items[].item_id` | String (UUID) | Inventory item ID | Yes |
| `items[].quantity` | Numeric | Quantity to deduct | Yes |
| `lab_order_item_id` | String (UUID) | Associated patient order item ID | No |
| `notes` | String | Description comments | No |

#### Example Request Body
```json
{
  "consumption_type": "MANUAL",
  "notes": "Manual check-out for training",
  "items": [
    {
      "item_id": "df86d618-0977-4579-a061-e75b1d14d4dd",
      "quantity": 2.0
    }
  ]
}
```

#### Example Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "message": "Lab items consumed successfully",
  "data": {
    "success": true,
    "message": "Lab consumables consumed successfully"
  }
}
```

---

### 6. Create Lab QC Record
Record daily equipment calibration or lot checking QC run results.

* **Route**: `POST /diagnostic-orders/lab/qc-records`
* **Permission Guard**: `diagnostics:lab:result:enter`

#### Request Headers
| Header | Type | Value | Mandatory |
| :--- | :--- | :--- | :--- |
| `Authorization` | String | `Bearer {{token}}` | Yes |
| `Content-Type` | String | `application/json` | Yes |

#### Request Body (JSON)
| Attribute | Type | Description | Mandatory |
| :--- | :--- | :--- | :--- |
| `item_id` | String (UUID) | Associated inventory catalog item ID | Yes |
| `qc_level` | String | Run tier level (`LOW`, `NORMAL`, `HIGH`) | Yes |
| `expected_value` | Numeric | Expected control value | Yes |
| `observed_value` | Numeric | Real observed control reading | Yes |
| `acceptable_range_min` | Numeric | Minimum allowed reading | Yes |
| `acceptable_range_max` | Numeric | Maximum allowed reading | Yes |
| `batch_id` | String (UUID) | Associated batch lot record ID | No |
| `result` | String | Outcome status override (`PASS`, `FAIL`) | No |

#### Example Request Body
```json
{
  "item_id": "df86d618-0977-4579-a061-e75b1d14d4dd",
  "qc_level": "NORMAL",
  "expected_value": 5.0,
  "observed_value": 4.9,
  "acceptable_range_min": 4.5,
  "acceptable_range_max": 5.5
}
```

#### Example Response (`201 Created`)
```json
{
  "success": true,
  "code": 201,
  "message": "QC record saved successfully",
  "data": {
    "id": "0f7aaa67-75f6-4da8-9207-2fd7d401576e",
    "performed_at": "2026-06-29 13:20:13.215357+05:30",
    "item_id": "df86d618-0977-4579-a061-e75b1d14d4dd",
    "batch_id": null,
    "qc_level": "NORMAL",
    "expected_value": 5.0,
    "observed_value": 4.9,
    "acceptable_range_min": 4.5,
    "acceptable_range_max": 5.5,
    "result": "PASS"
  }
}
```

---

## Part 2: Procurement Cycle Endpoints

### 7. Create Purchase Order (PO)
Initiate a purchase request requisition for lab goods.

* **Route**: `POST /diagnostic-orders/lab/procurement/po`
* **Permission Guard**: `diagnostics:test:manage`

#### Request Headers
| Header | Type | Value | Mandatory |
| :--- | :--- | :--- | :--- |
| `Authorization` | String | `Bearer {{token}}` | Yes |
| `Content-Type` | String | `application/json` | Yes |

#### Request Body (JSON)
| Attribute | Type | Description | Mandatory |
| :--- | :--- | :--- | :--- |
| `priority` | String | Requisition priority (`ROUTINE`, `HIGH`, `EMERGENCY`) | Yes |
| `lab_section` | String | Intended department section | Yes |
| `items` | Array | Purchase list items | Yes |
| `items[].item_id` | String (UUID) | Requisition item ID | Yes |
| `items[].item_name` | String | Supplier item description name | Yes |
| `items[].final_qty` | Numeric | Ordered item quantity | Yes |
| `items[].unit_price` | Numeric | Item unit rate | Yes |
| `items[].gst_percent` | Numeric | Tax component percentage | Yes |
| `vendor_id` | String (UUID) | Intended vendor reference ID | No |
| `notes` | String | Coordinator comments | No |

#### Example Request Body
```json
{
  "priority": "HIGH",
  "lab_section": "Haematology",
  "items": [
    {
      "item_id": "df86d618-0977-4579-a061-e75b1d14d4dd",
      "item_name": "Hematology Reagent Diluent",
      "final_qty": 10,
      "unit_price": 150.0,
      "gst_percent": 18.0
    }
  ]
}
```

#### Example Response (`201 Created`)
```json
{
  "success": true,
  "code": 201,
  "message": "Purchase Order created successfully",
  "data": {
    "id": "ad023d7c-d2bb-41e5-a5c4-83ca3f9d5150",
    "po_number": "PO-LAB-00001",
    "status": "DRAFT",
    "priority": "HIGH",
    "lab_section": "Haematology"
  }
}
```

---

### 8. Update Purchase Order
Modify details or transition PO state.

* **Route**: `PUT /diagnostic-orders/lab/procurement/po/:id`
* **Permission Guard**: `diagnostics:test:manage`

#### Path Parameters
* `id` (Mandatory): Purchase Order UUID string.

#### Request Headers
| Header | Type | Value | Mandatory |
| :--- | :--- | :--- | :--- |
| `Authorization` | String | `Bearer {{token}}` | Yes |
| `Content-Type` | String | `application/json` | Yes |

#### Request Body (JSON)
| Attribute | Type | Description | Mandatory |
| :--- | :--- | :--- | :--- |
| `status` | String | Updated status (`APPROVED`, `CANCELLED`, `RECEIVED`) | No |
| `notes` | String | Processing notes | No |

#### Example Request Body
```json
{
  "status": "APPROVED",
  "notes": "PO Approved for procurement"
}
```

#### Example Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "message": "Purchase Order updated successfully",
  "data": {
    "id": "ad023d7c-d2bb-41e5-a5c4-83ca3f9d5150",
    "po_number": "PO-LAB-00001",
    "status": "APPROVED",
    "notes": "PO Approved for procurement"
  }
}
```

---

### 9. Create Goods Received Note (GRN)
Log delivery shipment receipt. **Note: This action automatically creates inventory batches, increments item stock levels, and flags the associated PO as RECEIVED.**

* **Route**: `POST /diagnostic-orders/lab/procurement/grn`
* **Permission Guard**: `diagnostics:test:manage`

#### Request Headers
| Header | Type | Value | Mandatory |
| :--- | :--- | :--- | :--- |
| `Authorization` | String | `Bearer {{token}}` | Yes |
| `Content-Type` | String | `application/json` | Yes |

#### Request Body (JSON)
| Attribute | Type | Description | Mandatory |
| :--- | :--- | :--- | :--- |
| `po_id` | String (UUID) | Associated Purchase Order ID | Yes |
| `items` | Array | Received item lot items | Yes |
| `items[].item_id` | String (UUID) | Received inventory item ID | Yes |
| `items[].item_name` | String | Product details label | Yes |
| `items[].received_qty` | Numeric | Accepted batch shipment count | Yes |
| `items[].batch_number` | String | Vendor product lot/batch number label | Yes |
| `items[].expiry_date` | Date String | Lot expiration date (`YYYY-MM-DD`) | Yes |
| `items[].unit_price` | Numeric | Unit price | Yes |
| `items[].lot_number` | String | Manufacturer lot tag | No |
| `items[].manufacturing_date` | Date String | Manufacture date | No |
| `items[].coa_received` | Boolean | Certificate of Analysis verification status | No (default `false`) |
| `items[].qc_status` | String | Quality status flag (`PASSED`, `PENDING`) | No (default `PENDING`) |

#### Example Request Body
```json
{
  "po_id": "ad023d7c-d2bb-41e5-a5c4-83ca3f9d5150",
  "items": [
    {
      "item_id": "df86d618-0977-4579-a061-e75b1d14d4dd",
      "item_name": "Hematology Reagent Diluent",
      "received_qty": 10,
      "batch_number": "BATCH-LAB-001",
      "expiry_date": "2030-12-31",
      "unit_price": 150.0
    }
  ]
}
```

#### Example Response (`201 Created`)
```json
{
  "success": true,
  "code": 201,
  "message": "Goods Received Note processed successfully",
  "data": {
    "id": "5d203877-0cd2-4ce2-a7c5-876581ac6515",
    "grn_number": "GRN-LAB-00001",
    "po_id": "ad023d7c-d2bb-41e5-a5c4-83ca3f9d5150",
    "received_at": "2026-06-29 13:20:17.440367+05:30"
  }
}
```

---

### 10. Create Vendor Invoice
Save billing documentation from vendors matching the PO and GRN.

* **Route**: `POST /diagnostic-orders/lab/procurement/invoice`
* **Permission Guard**: `diagnostics:test:manage`

#### Request Headers
| Header | Type | Value | Mandatory |
| :--- | :--- | :--- | :--- |
| `Authorization` | String | `Bearer {{token}}` | Yes |
| `Content-Type` | String | `application/json` | Yes |

#### Request Body (JSON)
| Attribute | Type | Description | Mandatory |
| :--- | :--- | :--- | :--- |
| `invoice_number` | String | Vendor invoice billing reference | Yes |
| `po_id` | String (UUID) | Associated Purchase Order ID | Yes |
| `grn_id` | String (UUID) | Associated Goods Received Note ID | Yes |
| `invoice_amount` | Numeric | Subtotal amount | Yes |
| `gst_amount` | Numeric | Tax component total amount | Yes |
| `total_amount` | Numeric | Invoice grand total | Yes |
| `vendor_id` | String (UUID) | Vendor reference ID | No |
| `invoice_date` | Date String | Bill statement date | No |

#### Example Request Body
```json
{
  "invoice_number": "INV-12345",
  "po_id": "ad023d7c-d2bb-41e5-a5c4-83ca3f9d5150",
  "grn_id": "5d203877-0cd2-4ce2-a7c5-876581ac6515",
  "invoice_amount": 1500.0,
  "gst_amount": 270.0,
  "total_amount": 1770.0
}
```

#### Example Response (`201 Created`)
```json
{
  "success": true,
  "code": 201,
  "message": "Vendor Invoice created successfully",
  "data": {
    "id": "4a553d3c-eba9-4b28-887d-da1fe86017cc",
    "invoice_number": "INV-12345",
    "po_id": "ad023d7c-d2bb-41e5-a5c4-83ca3f9d5150",
    "grn_id": "5d203877-0cd2-4ce2-a7c5-876581ac6515",
    "invoice_amount": 1500.0,
    "gst_amount": 270.0,
    "total_amount": 1770.0
  }
}
```
