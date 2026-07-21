# API Documentation: Decoupled Walk-in Lab Intake & Payment-First Workflow

This document details the decoupled, payment-first walk-in laboratory intake process in the HMS Laboratory microservice. 

---

## Workflow Overview

The intake process is split into four progressive stages to isolate patient demographic creation, billing pricing verification, payment processing, and sample scanning operations:

```
[1. Patient Registration] ---> [2. Invoice Price Calculation] ---> [3. Pay & Create Order] ---> [4. Barcode Scan Lookup]
```

All endpoints require:
* **Base Path**: `https://api.arovita.com/diagnostic-orders`
* **Authorization**: `Bearer {{authToken}}` passed via the `Authorization` header.
* **Tenant Isolation**: `X-Tenant-Id` header to enforce database partition routing.

---

## 1. Walk-in Patient Registration

Creates a new patient profile inside the demographics table and generates a unique UHID.

* **HTTP Method**: `POST`
* **Path**: `/lab/walk-in` (or gateway path `/diagnostic-orders/lab/walk-in`)
* **Purpose**: Used at the registration desk when a walk-in patient arrives without any pre-existing profile.

### Headers
| Header | Required | Type | Description |
| :--- | :--- | :--- | :--- |
| `Authorization` | Yes | String | Bearer access token for Cognito validation. |
| `X-Tenant-Id` | Yes | UUID | Identifies the branch context (e.g. `46fc39d8-7c4e-4704-9430-f82d6dcfa34c`). |
| `Content-Type` | Yes | String | Must be `application/json`. |

### Request Fields
| Field Name | Required | Type | Description / Validation |
| :--- | :--- | :--- | :--- |
| `full_name` | Yes | String | Full name of the patient. |
| `age` | Yes | Integer | Patient age in years. |
| `date_of_birth` | Yes | String (Date) | Format: `YYYY-MM-DD`. |
| `gender` | Yes | String | Enums: `MALE`, `FEMALE`, `OTHER`. |
| `mobile_number` | Yes | String | Contact mobile number. |
| `address` | No | String | Physical contact address. |

#### Sample Request Body
```json
{
  "full_name": "Rajesh Kumar",
  "age": 45,
  "date_of_birth": "1981-05-15",
  "gender": "MALE",
  "mobile_number": "+919876543210",
  "address": "123 MG Road, Bangalore"
}
```

### Response Fields
| Field Name | Type | Description |
| :--- | :--- | :--- |
| `patient_id` | UUID | Generated unique identifier for the patient. Used in order creation. |
| `uhid` | String | Unique Health ID assigned (Prefix: `UH-` followed by 6 random alpha-numeric chars). |
| `full_name` | String | Confirmed display name. |

#### Sample Response Body (200 OK)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "patient_id": "26e74a42-4e8c-4492-b3fb-15994435af46",
    "uhid": "UH-6EB5F1",
    "full_name": "Rajesh Kumar"
  }
}
```

---

## 2. Price Calculation (Invoice Preview)

Calculates pricing totals for selected tests and packages, applying tax rates.

* **HTTP Method**: `POST`
* **Path**: `/lab/orders/calculate-price` (or gateway path `/diagnostic-orders/lab/orders/calculate-price`)
* **Purpose**: Used to present the price breakdown and invoice total to the patient before requesting payment.

### Request Fields
| Field Name | Required | Type | Description |
| :--- | :--- | :--- | :--- |
| `selected_tests` | Yes | Array | List of individual test objects. |
| `selected_tests[].test_id` | Yes | UUID | Unique test ID from `diagnostic.lab_tests`. |
| `selected_packages` | No | Array | List of test package objects. |
| `selected_packages[].package_id` | Yes | UUID | Unique package ID from `diagnostic.lab_test_packages`. |

#### Sample Request Body
```json
{
  "selected_tests": [
    {
      "test_id": "e5e82d23-96f8-4981-b068-52a12dc10de4"
    }
  ],
  "selected_packages": [
    {
      "package_id": "b5d66607-f4de-4ac4-bfed-c9b27adad890"
    }
  ]
}
```

### Response Fields
| Field Name | Type | Description |
| :--- | :--- | :--- |
| `subtotal` | Decimal | Cumulative base price of all tests and packages. |
| `tax` | Decimal | Calculated GST (18.00% of subtotal). |
| `total_amount` | Decimal | Final net amount payable by the patient (`subtotal + tax`). |

#### Sample Response Body (200 OK)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "subtotal": 1200.00,
    "tax": 216.00,
    "total_amount": 1416.00
  }
}
```

---

## 3. Pay and Create Order

Finalizes payment details, records transactions, and commits the lab order.

* **HTTP Method**: `POST`
* **Path**: `/lab/orders/pay-and-create` (or gateway path `/diagnostic-orders/lab/orders/pay-and-create`)
* **Purpose**: Executed once the front desk collects payment. If successful, creates the order and items, generating a single shared barcode tracking token.

### Request Fields
| Field Name | Required | Type | Description / Validation |
| :--- | :--- | :--- | :--- |
| `patient_id` | Yes | UUID | Patient ID from Step 1. |
| `selected_tests` | Yes | Array | List of test objects (same as Step 2). |
| `selected_packages` | No | Array | List of package objects (same as Step 2). |
| `special_instructions` | No | String | Free-text preparation guidelines. |
| `fasting_required` | No | Boolean | Fasting indicator (Yes/No). |
| `priority` | No | String | Default: `Routine`. Enums: `Normal`, `Routine`, `Urgent`, `Stat`. |
| `payment_mode` | Yes | String | Enums: `UPI`, `CASH`, `CARD`. |
| `amount_paid` | Yes | Decimal | Payment amount received (must match Step 2 total). |
| `reference_no` | Yes | String | Transaction identifier. |

#### Sample Request Body
```json
{
  "patient_id": "26e74a42-4e8c-4492-b3fb-15994435af46",
  "visit_type": "Walk-in",
  "priority": "Routine",
  "selected_tests": [
    {
      "test_id": "e5e82d23-96f8-4981-b068-52a12dc10de4"
    }
  ],
  "selected_packages": [
    {
      "package_id": "b5d66607-f4de-4ac4-bfed-c9b27adad890"
    }
  ],
  "special_instructions": "Fasting 10-12 hours. Avoid heavy food...",
  "fasting_required": true,
  "payment_mode": "UPI",
  "amount_paid": 1416.00,
  "reference_no": "REF-UPI-PAY-FIRST"
}
```

### Response Fields
| Field Name | Type | Description |
| :--- | :--- | :--- |
| `order_id` | UUID | Generated order ID. |
| `lab_unique_id` | String | Sequence order tracker (e.g. `LABID-29216`). |
| `items` | Array | Generated tests under the order. |
| `items[].order_item_id`| UUID | Item ID for results and collection mapping. |
| `items[].barcode` | String | **Single shared barcode tracking identifier** (e.g. `BC-10039`) assigned across all items in the order. |

#### Sample Response Body (200 OK)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "order_id": "d6d1dc99-b840-450b-8936-f03d9e892159",
    "patient_id": "26e74a42-4e8c-4492-b3fb-15994435af46",
    "lab_unique_id": "LABID-29216",
    "status": "PENDING",
    "total_amount": 1416.00,
    "payment_status": "PAID",
    "items": [
      {
        "order_item_id": "83a308ed-4ed5-4b5e-a0b6-85699911b60d",
        "test_id": "e5e82d23-96f8-4981-b068-52a12dc10de4",
        "barcode": "BC-10039"
      },
      {
        "order_item_id": "8a3170a7-31ac-4d6c-bc6b-b1d267c9a016",
        "test_id": "1a0ef01c-36ac-4c8f-a451-110f9d60d393",
        "barcode": "BC-10039"
      }
    ]
  }
}
```

---

## 4. Barcode Scan Lookup

Returns patient details, tests, and processing states upon barcode scanner trigger.

* **HTTP Method**: `GET`
* **Path**: `/lab/samples` (or gateway path `/diagnostic-orders/lab/samples`)
* **Purpose**: Triggered by barcode scanner integration at the collection counter or processing bench.

### Query Parameters
| Parameter | Required | Type | Description |
| :--- | :--- | :--- | :--- |
| `barcode` | Yes | String | Scanned barcode value (e.g. `BC-10039`). |

#### Sample Request URL
`GET https://api.arovita.com/diagnostic-orders/lab/samples?barcode=BC-10039`

### Response Fields
| Field Name | Type | Description |
| :--- | :--- | :--- |
| `patient_number` | String | Mobile number of the patient. |
| `patient_uhid` | String | Patient UHID from Step 1. |
| `test_name` | String | Name of the test matching the barcode. |
| `status` | String | Current processing status of the sample. |

#### Sample Response Body (200 OK)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "total": 2,
    "page": 1,
    "limit": 10,
    "items": [
      {
        "sample_id": "8a3170a7-31ac-4d6c-bc6b-b1d267c9a016",
        "barcode": "BC-10039",
        "patient_name": "Test Payment First Walkin",
        "patient_uhid": "UH-6EB5F1",
        "patient_number": "+919900990099",
        "test_name": "CBC (Complete Blood Count)",
        "status": "PENDING",
        "priority": "ROUTINE",
        "created_at": "2026-07-17 16:10:33.171229+05:30"
      },
      {
        "sample_id": "83a308ed-4ed5-4b5e-a0b6-85699911b60d",
        "barcode": "BC-10039",
        "patient_name": "Test Payment First Walkin",
        "patient_uhid": "UH-6EB5F1",
        "patient_number": "+919900990099",
        "test_name": "Lipid Profile",
        "status": "PENDING",
        "priority": "ROUTINE",
        "created_at": "2026-07-17 16:10:33.171229+05:30"
      }
    ]
  }
}
```

---

## 5. OPD Referral Patient & Visit Lookup

Fetches demographics and referring doctor details (if any) for a given OPD visit number.

* **HTTP Method**: `GET`
* **Path**: `/lab/opd-visits/lookup` (or gateway path `/diagnostic-orders/lab/opd-visits/lookup`)
* **Purpose**: Used when scanning or searching by the `OPD Visit No` to pre-fill the patient demographics and referring doctor fields.

### Query Parameters
| Parameter | Required | Type | Description |
| :--- | :--- | :--- | :--- |
| `visit_number` | Yes | String | Active OPD visit number (e.g. `OPD-20260615-0001`). |

#### Sample Request URL
`GET https://api.arovita.com/diagnostic-orders/lab/opd-visits/lookup?visit_number=OPD-20260615-0001`

### Response Fields
| Field Name | Type | Description |
| :--- | :--- | :--- |
| `patient` | Object | Demographics including Full Name, UHID, Mobile, DOB, Gender, etc. |
| `referring_doctor` | Object (Optional) | Assigned consulting doctor's name, specialty department, and notes. Set to `null` if none assigned. |

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

## 6. Submit Visit Details (OPD Referral)

Registers/saves the visit details context and returns a generated `visit_id`.

* **HTTP Method**: `POST`
* **Path**: `/lab/orders/visit-details` (or gateway path `/diagnostic-orders/lab/orders/visit-details`)
* **Purpose**: Registers the visit details context parameters separately. The returned `visit_id` is then passed to the order creation endpoint.

### Request Fields
| Field Name | Required | Type | Description / Validation |
| :--- | :--- | :--- | :--- |
| `visit_type` | Yes | String | e.g. `OPD Referral`. |
| `token_number` | Yes | String | Token sequence value (e.g. `OPD-2419`). |
| `registration_time` | Yes | String | Time value (e.g. `10:45 AM`). |
| `priority` | Yes | String | Priority status (e.g. `Normal`, `Urgent`). |

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
| `visit_id` | UUID | Generated visit context identifier to pass in `pay-and-create` call. |

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

