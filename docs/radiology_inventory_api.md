# Radiology Inventory API Documentation

Postman requests and backend routes for managing radiology inventory consumables (contrast gels, films, catheters) allocated to the Radiology department.

---

## 1. List Radiology Inventory
Retrieve all active inventory items assigned to the Radiology department.

* **Route**: `GET /diagnostic-orders/radiology/inventory`
* **Permission Guard**: `diagnostics:order:view`

### Request Headers
| Header | Type | Value | Mandatory |
| :--- | :--- | :--- | :--- |
| `Authorization` | String | `Bearer {{token}}` | Yes |

### Query Parameters
| Parameter | Type | Description | Mandatory |
| :--- | :--- | :--- | :--- |
| `search` | String | Filter inventory items by matching name (case-insensitive) | No |

### Response Example (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "id": "896c6030-739f-4713-bd54-c9a4bcc656ec",
      "tenant_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
      "item_name": "TEST Contrast Gel",
      "current_stock": 95.0,
      "item_code": "TEST-CG-001",
      "category": "Radiology Consumables",
      "reorder_level": 20,
      "minimum_stock": 10,
      "assigned_department": "896c6030-739f-4713-bd54-c9a4bcc656ec",
      "is_active": true
    }
  ]
}
```

---

## 2. Get Inventory Alerts
Fetch the overall stock health percentage and retrieve a list of items currently under warning thresholds.

* **Route**: `GET /diagnostic-orders/radiology/inventory/alerts`
* **Permission Guard**: `diagnostics:dashboard`

### Request Headers
| Header | Type | Value | Mandatory |
| :--- | :--- | :--- | :--- |
| `Authorization` | String | `Bearer {{token}}` | Yes |

### Response Example (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "stock_health_percentage": 0,
    "items": [
      {
        "item_name": "TEST Contrast Gel",
        "status": "WARNING",
        "current_stock": 10,
        "min_stock": 10,
        "uom": "BOTTLE"
      }
    ]
  }
}
```

---

## 3. Consume Inventory Items
Manually deduct stock for a specific radiology scan procedure.

* **Route**: `POST /diagnostic-orders/radiology/inventory/consume`
* **Permission Guard**: `diagnostics:imaging:result:enter`

### Request Headers
| Header | Type | Value | Mandatory |
| :--- | :--- | :--- | :--- |
| `Authorization` | String | `Bearer {{token}}` | Yes |
| `Content-Type` | String | `application/json` | Yes |

### Request Body (JSON)
| Attribute | Type | Description | Mandatory |
| :--- | :--- | :--- | :--- |
| `imaging_order_item_id` | String (UUID) | The ID of the radiology scan order item | Yes |
| `items` | Array | List of items to consume | Yes |
| `items[].item_id` | String (UUID) | The ID of the inventory catalog item | Yes |
| `items[].quantity` | Numeric | The decimal quantity to deduct from stock | Yes |
| `notes` | String | Comments or description for the consumption event | No |

### Response Example (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "message": "Consumables consumed successfully"
}
```

### Error Scenarios
* **`409 Conflict`**: If the target radiology scan item status is `'CANCELLED'`.
```json
{
  "success": false,
  "message": "Cannot consume stock for a cancelled scan item '635adea1-03e5-43f1-b286-f1a381052ab9'",
  "errors": [],
  "code": 409
}
```
* **`404 Not Found`**: If the scan item or any catalog item ID is invalid.
* **`422 Unprocessable Entity`**: If validation fails (e.g. quantity is zero or negative).
