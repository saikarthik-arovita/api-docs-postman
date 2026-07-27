# HMS Service Catalog API Documentation

The Service Catalog module provides administrative control over all clinical services, diagnostic tests, consultations, packages, and surgical procedures. It manages pricing, insurance eligibility, doctor/hospital revenue splits, tiered room-category pricing, and compliance audit histories.

---

## Endpoint Index

1. [GET /admin/service-catalogue/summary](#1-get-service-catalogue-summary)
2. [GET /admin/service-catalogue/records](#2-get-list-service-catalogue-records)
3. [POST /admin/service-catalogue/records](#3-post-create-service-catalogue-record)
4. [GET /admin/service-catalogue/records/{service_id}](#4-get-single-service-catalogue-record)
5. [PATCH /admin/service-catalogue/records/{service_id}](#5-patch-update-service-catalogue-record)
6. [POST /admin/service-catalogue/records/{service_id}/pricing-tiers](#6-post-configure-service-pricing-tiers)
7. [GET /admin/service-catalogue/records/{service_id}/audit-trail](#7-get-service-audit-trail)
8. [GET /admin/service-catalogue/records/{service_id}/pricing-history](#8-get-service-pricing-history)

---

## 1. GET Service Catalogue Summary
Retrieve hospital-wide metrics for the Service Catalog, including active item counts, category percentages, and top services by volume.

* **Method**: `GET`
* **URL**: `/admin/service-catalogue/summary`
* **Headers**:
  * `Authorization`: `Bearer <token>`
  * `x-tenant-id`: `<tenant_id>`

### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "total_catalogue_items": 248,
    "active_services": 215,
    "speciality_packages": 142,
    "diagnostic_tests": 108,
    "total_monthly_revenue": "125000.00",
    "services_by_category": [
      {
        "category": "PREVENTIVE",
        "percentage": 12.0,
        "count": 30
      },
      {
        "category": "DIAGNOSTIC",
        "percentage": 35.0,
        "count": 87
      }
    ],
    "top_services_by_volume": [
      {
        "service": "Cardiology Consultation",
        "count": 420
      }
    ]
  }
}
```

---

## 2. GET List Service Catalogue Records
Retrieve a paginated, filterable list of clinical services.

* **Method**: `GET`
* **URL**: `/admin/service-catalogue/records`
* **Query Parameters**:
  * `category` (*Optional*): Filter by service category (`CLINICAL`, `DIAGNOSTIC`, `SURGICAL`, `PREVENTIVE`, `EMERGENCY`).
  * `department_id` (*Optional*): Filter by clinical department UUID.
  * `status` (*Optional*): Filter by status (`ACTIVE`, `INACTIVE`, `SEASONAL`, `SUSPENDED`).
  * `search` (*Optional*): Search query matching service name or code.
  * `page` (*Optional*, Default: `1`): Page number.
  * `page_size` (*Optional*, Default: `10`): Items limit.

### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "id": "0b4c69fb-63d4-4f16-aa28-96f841d54485",
        "service_name": "General Consultation",
        "category": "PREVENTIVE",
        "department_id": "46d19155-09a1-462e-b256-ce2a5741b091",
        "department_name": "Surgery",
        "service_code": "SERV-20260727",
        "base_fee": "650.00",
        "doctor_share_pct": "60.00",
        "hospital_share_pct": "40.00",
        "tax_rate_pct": "5.00",
        "status": "Active",
        "is_active": true,
        "availability_status": "ACTIVE",
        "description": "Standard general medical consultation",
        "emergency_surcharge": "0.00",
        "after_hours_surcharge": "0.00",
        "avg_duration_minutes": 15,
        "max_daily_capacity": 40,
        "is_insurance_covered": true,
        "tpa_approval_required": false
      }
    ],
    "meta": {
      "total": 1,
      "page": 1,
      "page_size": 10,
      "total_pages": 1
    }
  }
}
```

---

## 3. POST Create Service Catalogue Record
Create a new clinical service catalog entry.

* **Method**: `POST`
* **URL**: `/admin/service-catalogue/records`
* **Request Body**:
```json
{
  "service_name": "QA General Consultation",
  "category": "PREVENTIVE",
  "department_id": "46d19155-09a1-462e-b256-ce2a5741b091",
  "base_fee": 650.00,
  "doctor_share_pct": 60.00,
  "hospital_share_pct": 40.00,
  "tax_rate_pct": 5.00,
  "availability_status": "ACTIVE",
  "description": "Standard QA general medical consultation"
}
```

### Response (`201 Created`)
```json
{
  "success": true,
  "code": 201,
  "data": {
    "id": "0b4c69fb-63d4-4f16-aa28-96f841d54485",
    "service_name": "QA General Consultation",
    "service_code": "SERV-20260727",
    "base_fee": "650.00",
    "doctor_share_pct": "60.00",
    "hospital_share_pct": "40.00",
    "tax_rate_pct": "5.00",
    "availability_status": "ACTIVE"
  }
}
```

---

## 4. GET Single Service Catalogue Record
Retrieve complete configuration details of a single service by its ID.

* **Method**: `GET`
* **URL**: `/admin/service-catalogue/records/{service_id}`

### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "0b4c69fb-63d4-4f16-aa28-96f841d54485",
    "service_name": "QA General Consultation",
    "category": "PREVENTIVE",
    "department_id": "46d19155-09a1-462e-b256-ce2a5741b091",
    "department_name": "Surgery",
    "service_code": "SERV-20260727",
    "base_fee": "650.00",
    "doctor_share_pct": "60.00",
    "hospital_share_pct": "40.00",
    "tax_rate_pct": "5.00",
    "status": "Active",
    "is_active": true,
    "availability_status": "ACTIVE",
    "description": "Standard QA general medical consultation",
    "emergency_surcharge": "0.00",
    "after_hours_surcharge": "0.00",
    "avg_duration_minutes": 15,
    "max_daily_capacity": 40,
    "is_insurance_covered": true,
    "tpa_approval_required": false
  }
}
```

---

## 5. PATCH Update Service Catalogue Record
Partially update service details. Modifying the `base_fee` will trigger pricing history log insertion.

* **Method**: `PATCH`
* **URL**: `/admin/service-catalogue/records/{service_id}`
* **Request Body**:
```json
{
  "base_fee": 700.00,
  "change_reason": "Adjust fee for annual pricing review"
}
```

### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "id": "0b4c69fb-63d4-4f16-aa28-96f841d54485",
    "service_name": "QA General Consultation",
    "base_fee": "700.00",
    "updated_at": "2026-07-27T11:21:36Z"
  }
}
```

---

## 6. POST Configure Service Pricing Tiers
Configure ward-specific or TPA-specific pricing overrides for a service.

* **Method**: `POST`
* **URL**: `/admin/service-catalogue/records/{service_id}/pricing-tiers`
* **Request Body**:
```json
{
  "tiers": [
    {
      "room_category": "ICU",
      "custom_fee": 1200.00,
      "tpa_id": null,
      "is_active": true
    }
  ]
}
```

### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "success": true,
    "message": "Pricing tiers applied successfully"
  }
}
```

---

## 7. GET Service Audit Trail
Fetch audit history tracking modifications to this service catalog record.

* **Method**: `GET`
* **URL**: `/admin/service-catalogue/records/{service_id}/audit-trail`

### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "id": "af9bed60-a8ba-4ed9-9787-c6bf2116262a",
      "service_id": "0b4c69fb-63d4-4f16-aa28-96f841d54485",
      "action_type": "UPDATED",
      "old_data": {
        "base_fee": "650.00"
      },
      "new_data": {
        "base_fee": "700.00"
      },
      "performed_by": "902d2bc4-f5ee-45df-98bd-674cd7bb0eef",
      "timestamp": "2026-07-27T11:21:36.727Z"
    }
  ]
}
```

---

## 8. GET Service Pricing History
Retrieve historical changes to this service's base fee tariff.

* **Method**: `GET`
* **URL**: `/admin/service-catalogue/records/{service_id}/pricing-history`

### Response (`200 OK`)
```json
{
  "success": true,
  "code": 200,
  "data": [
    {
      "id": "20f39a9e-82ac-4e2c-b6fc-dabd261d0b75",
      "service_id": "0b4c69fb-63d4-4f16-aa28-96f841d54485",
      "old_base_fee": "650.00",
      "new_base_fee": "700.00",
      "change_reason": "Adjust fee for annual pricing review",
      "changed_by": "902d2bc4-f5ee-45df-98bd-674cd7bb0eef",
      "changed_at": "2026-07-27T11:21:36.727Z"
    }
  ]
}
```
