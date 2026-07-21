# API Documentation: Decoupled OPD Referral Lab Intake & Payment-First Workflow

This document details the decoupled, payment-first OPD Referral laboratory intake process in the HMS Laboratory microservice.

---

## Workflow Overview

The OPD Referral intake workflow separates patient demographics lookup, pricing calculation, visit context initialization, and order payment checkouts:

```
[1. Patient & Visit Lookup] ---> [2. Invoice Price Calculation] ---> [3. Submit Visit Details] ---> [4. Pay & Create Order]
```

All endpoints require:
* **Base Path**: `https://api.arovita.com/diagnostic-orders`
* **Authorization**: `Bearer {{authToken}}` passed via the `Authorization` header.
* **Tenant Isolation**: `X-Tenant-Id` header to enforce database partition routing.

---

## 1. Patient & Referral Lookup

Fetches demographics and referring doctor details (if any) for a given OPD visit number from the database.

* **HTTP Method**: `GET`
* **Path**: `/lab/opd-visits/lookup` (or gateway path `/diagnostic-orders/lab/opd-visits/lookup`)
* **Purpose**: Used when scanning or searching by the `OPD Visit No` to pre-fill the patient demographics and referring doctor fields.

### Headers
| Header | Required | Type | Description |
| :--- | :--- | :--- | :--- |
| `Authorization` | Yes | String | Bearer access token for Cognito validation. |
| `X-Tenant-Id` | Yes | UUID | Identifies the branch context. |

### Query Parameters
| Parameter | Required | Type | Description / Validation |
| :--- | :--- | :--- | :--- |
| `visit_number` | Yes | String | Active OPD visit number (e.g., `OPD-20260615-0001`). |

#### Sample Request URL
`GET https://api.arovita.com/diagnostic-orders/lab/opd-visits/lookup?visit_number=OPD-20260615-0001`

### Response Fields
| Field Name | Type | Description |
| :--- | :--- | :--- |
| `patient` | Object | Demographics including Full Name, UHID, Mobile, DOB, Gender, and address. |
| `referring_doctor` | Object / null | Consulting doctor details (Doctor Name, Department, Referral Code, notes). **Optional** (returns `null` if no referring doctor is assigned). |

#### Sample Response Body (200 OK)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "patient": {
      "patient_id": "e3268fdf-f9dc-48f5-a601-2cfe9b197f2b",
      "uhid": "PAT-2026-0005",
      "full_name": "E2E Simulation Patient",
      "age": 35,
      "date_of_birth": null,
      "gender": "MALE",
      "mobile_number": "+919953847134",
      "address": null,
      "registration_date": "2026-06-15"
    },
    "referring_doctor": {
      "doctor_name": "Anita Rao",
      "department_name": "Surgery",
      "referral_code": "REF-OPD-20260615-0001",
      "consultation_notes": null
    }
  }
}
```

---

## 2. Submit Visit Details

Registers the visit parameters separately and returns a validated `visit_id`.

* **HTTP Method**: `POST`
* **Path**: `/lab/orders/visit-details` (or gateway path `/diagnostic-orders/lab/orders/visit-details`)
* **Purpose**: Registers the visit details context parameters separately. The returned `visit_id` is then passed to the order creation endpoint.

### Headers
| Header | Required | Type | Description |
| :--- | :--- | :--- | :--- |
| `Authorization` | Yes | String | Bearer access token. |
| `X-Tenant-Id` | Yes | UUID | Identifies the branch context. |
| `Content-Type` | Yes | String | Must be `application/json`. |

### Request Fields
| Field Name | Required | Type | Description / Validation |
| :--- | :--- | :--- | :--- |
| `visit_type` | Yes | String | e.g. `OPD Referral`. |
| `token_number` | Yes | String | Token sequence value (e.g., `OPD-2419`). |
| `registration_time` | Yes | String | Time value (e.g., `10:45 AM`). |
| `priority` | Yes | String | Priority status (e.g., `Normal`, `Urgent`). |

#### Sample Request Body
```json
{
  "visit_type": "OPD Referral",
  "token_number": "OPD-2419",
  "registration_time": "10:45 AM",
  "priority": "Normal"
}
```

### Response Fields
| Field Name | Type | Description |
| :--- | :--- | :--- |
| `visit_id` | UUID | Generated visit context identifier to pass in the `pay-and-create` call. |

#### Sample Response Body (200 OK)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "visit_id": "46884bf4-50e1-491b-b26f-4108c280b4df",
    "visit_type": "OPD Referral",
    "token_number": "OPD-2419"
  }
}
```

---

## 3. Pay and Create Order

Confirms payment details and links the generated `visit_id` to finalize the lab order.

* **HTTP Method**: `POST`
* **Path**: `/lab/orders/pay-and-create` (or gateway path `/diagnostic-orders/lab/orders/pay-and-create`)
* **Purpose**: Processes payment and creates the order. The referring doctor and visit details are aggregated inside the database `clinical_notes` audit column, and only **one single shared barcode** is generated across all items in the order.

### Request Fields
| Field Name | Required | Type | Description / Validation |
| :--- | :--- | :--- | :--- |
| `patient_id` | Yes | UUID | Patient ID from Step 1. |
| `visit_id` | Yes | UUID | Generated visit context ID from Step 2. |
| `visit_type` | Yes | String | Must be `OPD Referral` to map to `CONSULTATION` encounter type. |
| `selected_tests` | Yes | Array | List of test objects. |
| `selected_packages` | No | Array | List of package objects. |
| `special_instructions` | No | String | Free-text preparation guidelines. |
| `fasting_required` | No | Boolean | Fasting indicator (Yes/No). |
| `payment_mode` | Yes | String | Enums: `UPI`, `CASH`, `CARD`. |
| `amount_paid` | Yes | Decimal | Payment amount received. |
| `reference_no` | Yes | String | Transaction identifier. |
| `referring_doctor` | No | Object | Referring doctor details (doctor_name, department_name, referral_code). |

#### Sample Request Body
```json
{
  "patient_id": "e3268fdf-f9dc-48f5-a601-2cfe9b197f2b",
  "visit_id": "46884bf4-50e1-491b-b26f-4108c280b4df",
  "visit_type": "OPD Referral",
  "selected_tests": [
    {
      "test_id": "e5e82d23-96f8-4981-b068-52a12dc10de4"
    }
  ],
  "special_instructions": "Fasting 10-12 hours.",
  "fasting_required": true,
  "payment_mode": "UPI",
  "amount_paid": 368.00,
  "reference_no": "REF-UPI-OPD",
  "referring_doctor": {
    "doctor_name": "Anita Rao",
    "department_name": "Surgery",
    "referral_code": "REF-OPD-20260615-0001",
    "consultation_notes": null
  }
}
```

### Response Fields
| Field Name | Type | Description |
| :--- | :--- | :--- |
| `order_id` | UUID | Generated order ID. |
| `lab_unique_id` | String | Sequence order tracker (e.g. `LABID-29222`). |
| `items` | Array | Generated tests under the order, sharing a single barcode. |

#### Sample Response Body (200 OK)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "order_id": "5d052c60-6cf4-4865-892b-de541e33b9de",
    "patient_id": "e3268fdf-f9dc-48f5-a601-2cfe9b197f2b",
    "lab_unique_id": "LABID-29222",
    "status": "PENDING",
    "total_amount": 368.00,
    "payment_status": "PAID",
    "items": [
      {
        "order_item_id": "f2715fa1-a949-4871-be16-c1754e7d73cf",
        "test_id": "e5e82d23-96f8-4981-b068-52a12dc10de4",
        "barcode": "BC-10045"
      }
    ]
  }
}
```
