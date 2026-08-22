# HMS — Lab Service API Documentation

This document covers all laboratory technician operations, sample queues, real-time dashboard analytics, catalog inventory management, test mapping, quality control checks, and the procurement lifecycle (POs, GRNs, and Vendor Invoices) for the HMS **Lab Service** (integrated with diagnostics).

---

## 1. Global Conventions

### Base URL
All requests are sent to the Diagnostics Service API Gateway stage:
```
https://<api-id>.execute-api.ap-south-1.amazonaws.com/<stage>
```
All routes are prefixed with `/diagnostic-orders`.

### Authorization Header
All endpoints require a valid JWT Access Token passed in the `Authorization` header:
```http
Authorization: Bearer <access_token>
```

### Universal Response Envelope
All API responses follow the standard envelope format:

**Success Response (200 OK / 201 Created):**
```json
{
  "success": true,
  "data": { ... },
  "message": "Optional descriptive success message"
}
```

---

## 1.5 Patient Search, Registration, Booking & Flexible Billing

### 1.5.1 Search Registered Patients & Active Lab Tests
Searches the patient demographics registry. Enriches each returned patient with their count of active diagnostic orders, and an array of all active lab tests and packages currently in progress.

* **Endpoint:** `GET /lab/patients` *(Alias: `GET /diagnostic-orders/lab/patients`)*
* **Required Permission:** `diagnostics:order:view`
* **Query Parameters:**
  | Parameter | Type | Required? | Description |
  | :--- | :--- | :--- | :--- |
  | `q` | String | Optional | Search query matching Patient Name, UHID, or Mobile Number. If omitted, returns recent patients. |

* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": "93643ddd-0d0d-491d-83d6-37f3bcde518d",
      "uhid": "PAT-2026-0042",
      "full_name": "Rohan Deshmukh",
      "phone": "9876543210",
      "dob": "1992-04-15",
      "age": 34,
      "gender": "MALE",
      "blood_group": "O+",
      "address": "42 MG Road, Bangalore",
      "has_active_tests": true,
      "active_orders_count": 1,
      "active_lab_tests": [
        {
          "order_id": "c8542c47-f3e5-4ec0-8f24-b9f7e58c32f2",
          "lab_unique_id": "LABID-29181",
          "order_item_id": "df86d618-0977-4579-a061-e75b1d14d4dd",
          "test_id": "16a43dca-eb71-4b85-ab0a-3b984816cadf",
          "test_name": "Complete Blood Count",
          "test_code": "CBC",
          "test_category": "Hematology",
          "specimen_type": "Whole Blood",
          "status": "SAMPLE_COLLECTED",
          "priority": "ROUTINE",
          "barcode": "BC-10029",
          "token_number": "TKN-1004",
          "created_at": "2026-08-22T10:40:00Z"
        }
      ],
      "last_order_date": "2026-08-22T10:40:00Z"
    }
  ]
}
```

---

### 1.5.2 Patient Registration
Registers a new patient with standard `PAT-{YYYY}-{seq}` UHID generation, aligning directly with the **Patient Information — New Registration** form specifications.

* **Endpoint:** `POST /lab/patients` *(Alias: `POST /diagnostic-orders/lab/patients`)*
* **Required Permission:** `diagnostics:order:view`
* **Request Body (JSON):**
```json
{
  "id_type": "Aadhar Card",
  "id_number": "4920 1827 3849",
  "first_name": "Rajesh",
  "last_name": "Kumar",
  "date_of_birth": "1978-03-07",
  "age": 42,
  "gender": "Male",
  "blood_group": "B+",
  "phone": "+91 87654 3210",
  "pincode": "560003",
  "street_name": "24, 3rd Cross",
  "city": "Bengaluru",
  "district": "Bengaluru Urban",
  "state": "Karnataka",
  "country": "India",
  "email": "rajeshk@email.com",
  "emergency_contacts": [
    {
      "relationship": "Mother",
      "name": "Heer Kumar",
      "contact_number": "+91 78901 23456"
    }
  ],
  "consent_confirmed": true
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Patient registered successfully",
  "data": {
    "patient_id": "83d637f3-bcde-491d-9364-3ddd0d0d518d",
    "id": "83d637f3-bcde-491d-9364-3ddd0d0d518d",
    "uhid": "PAT-2026-6482",
    "patient_number": "PAT-2026-6482",
    "first_name": "Rajesh",
    "last_name": "Kumar",
    "full_name": "Rajesh Kumar",
    "id_type": "Aadhar Card",
    "id_number": "4920 1827 3849",
    "phone": "+91 87654 3210",
    "email": "rajeshk@email.com",
    "dob": "1978-03-07",
    "age": 42,
    "gender": "MALE",
    "blood_group": "B+",
    "street_name": "24, 3rd Cross",
    "city": "Bengaluru",
    "district": "Bengaluru Urban",
    "state": "Karnataka",
    "pincode": "560003",
    "country": "India",
    "address": "24, 3rd Cross, Bengaluru, Bengaluru Urban, Karnataka - 560003, India",
    "emergency_contacts": [
      {
        "relationship": "Mother",
        "name": "Heer Kumar",
        "contact_number": "+91 78901 23456"
      }
    ],
    "consent_confirmed": true,
    "created_at": "2026-08-22 12:28:10.123456+05:30"
  }
}
```

---

### 1.5.3 List Tests and Test Packages Catalog
Retrieves individual lab tests and bundled test packages.

* **Endpoints:**
  * Tests: `GET /lab/tests` *(Alias: `GET /diagnostic-orders/lab/tests`)*
  * Packages: `GET /lab/packages` *(Alias: `GET /diagnostic-orders/lab/packages`)*
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": "b5d66607-f4de-4ac4-bfed-c9b27adad890",
      "package_id": "b5d66607-f4de-4ac4-bfed-c9b27adad890",
      "package_name": "Comprehensive Executive Health Check",
      "package_code": "PKG-EXEC-01",
      "price": 1200.00,
      "discount_percentage": 10.00,
      "is_active": true,
      "tests": [
        {
          "test_id": "16a43dca-eb71-4b85-ab0a-3b984816cadf",
          "test_name": "Complete Blood Count",
          "category": "Hematology",
          "price": 350.00
        },
        {
          "test_id": "42e82d23-96f8-4981-b068-52a12dc10de4",
          "test_name": "Lipid Profile",
          "category": "Biochemistry",
          "price": 650.00
        }
      ]
    }
  ]
}
```

---

### 1.5.4 Calculate Order Price (Invoice Preview)
Calculates subtotal, package discounts, and GST (18%) for arbitrary combinations of individual tests and packages.

* **Endpoint:** `POST /lab/orders/calculate-price`
* **Request Body:**
```json
{
  "selected_tests": [
    { "test_id": "16a43dca-eb71-4b85-ab0a-3b984816cadf" }
  ],
  "selected_packages": [
    { "package_id": "b5d66607-f4de-4ac4-bfed-c9b27adad890" }
  ]
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "subtotal": 1550.00,
    "discount_amount": 120.00,
    "taxable_amount": 1430.00,
    "tax": 257.40,
    "tax_amount": 257.40,
    "total_amount": 1687.40,
    "tests": [
      {
        "test_id": "16a43dca-eb71-4b85-ab0a-3b984816cadf",
        "test_name": "Complete Blood Count",
        "price": 350.00
      }
    ],
    "packages": [
      {
        "package_id": "b5d66607-f4de-4ac4-bfed-c9b27adad890",
        "package_name": "Comprehensive Executive Health Check",
        "price": 1200.00,
        "discount": 120.00,
        "test_ids": [
          "16a43dca-eb71-4b85-ab0a-3b984816cadf",
          "42e82d23-96f8-4981-b068-52a12dc10de4"
        ]
      }
    ]
  }
}
```

---

### 1.5.5 Create Registration / Order (Draft, Instant Pay, Pay Later, Cards, UPI, Insurance)
Supports draft booking (`action: "DRAFT"`), multi-test and multi-package combinations, and flexible payment methods:
- **`CASH`**: Directly confirmed with `PAID` status.
- **`UPI`**: Directly confirmed; takes `reference_no` (optional).
- **`CARD`**: Directly confirmed; takes `reference_no` (optional).
- **`PAY_LATER`**: Directly confirmed; adds total to patient's billing ledger as an outstanding balance with status `PAY_LATER` / `OPEN`.
- **`INSURANCE` / `OTHERS`**: Recorded with policy claim details.

* **Endpoints:**
  * `POST /lab/registrations` *(Draft or Instant Register & Pay)*
  * `POST /lab/orders/pay-and-create`
* **Required Permission:** `diagnostics:order:view`

#### Request Body
```json
{
  "action": "REGISTER_AND_PAY",
  "patient_id": "83d637f3-bcde-491d-9364-3ddd0d0d518d",
  "visit_type": "Walk-in",
  "priority": "Routine",
  "fasting_required": true,
  "special_instructions": "12 hours fasting required",
  "selected_tests": [
    { "test_id": "16a43dca-eb71-4b85-ab0a-3b984816cadf" }
  ],
  "selected_packages": [
    { "package_id": "b5d66607-f4de-4ac4-bfed-c9b27adad890" }
  ],
  "payment": {
    "payment_mode": "UPI",
    "reference_no": "TXN-UPI-99201"
  }
}
```

#### Success Response (201 Created / 200 OK)
Returns the complete payload formatted for the **"Lab Registration Completed"** UI card with synchronized identifiers, barcode, waiting token, and all item UUIDs:
```json
{
  "success": true,
  "message": "Registration completed successfully",
  "data": {
    "order_id": "73aaa2f0-e1d0-442f-9f9b-4bf0d58a08a5",
    "id": "73aaa2f0-e1d0-442f-9f9b-4bf0d58a08a5",
    "labid": "LABID-29184",
    "patient_lab_id": "LABID-29184",
    "lab_unique_id": "LABID-29184",
    "lab_id": "LAB-2419",
    "order_number": "LAB-2419",
    "barcode": "BC-2419",
    "barcode_id": "7151abcf-5a89-4339-8c6a-b49d21c5084f",
    "token_number": "LAB-2419",
    "registration_date": "2026-04-20",
    "registration_time": "10:45:00",
    "registration_datetime": "10:45 AM, 20 April 2026",
    "patient_id": "401fcd56-3bd0-4247-8ece-d2cc47b616dd",
    "patient_name": "Ananya Sharma",
    "uhid": "ARV-2024-8932",
    "age": 32,
    "gender": "Female",
    "phone_number": "+91 9938283477",
    "contact": "+91 9938283477",
    "visit_type": "Walk-in",
    "priority": "Routine",
    "pathologist": "Dr. Sarah Smith",
    "doctor_name": "Dr. Sarah Smith",
    "status": "CONFIRMED",
    "payment_status": "PAID",
    "payment_mode": "UPI",
    "reference_no": "UPI-TXN-2026-9911",
    "billing": {
      "subtotal": 1500.00,
      "discount_amount": 0.00,
      "tax_amount": 250.00,
      "total_amount": 1750.00,
      "amount_paid": 1750.00,
      "outstanding_balance": 0.00
    },
    "total_amount": 1750.00,
    "test_names": [
      "CBC (Complete Blood Count)",
      "Lipid Profile",
      "HbA1c"
    ],
    "tests": [
      {
        "order_item_id": "7151abcf-5a89-4339-8c6a-b49d21c5084f",
        "id": "7151abcf-5a89-4339-8c6a-b49d21c5084f",
        "test_id": "2570848a-74ce-48db-b1f7-1fc450e060c5",
        "test_name": "CBC (Complete Blood Count)",
        "test_code": "HEM-001",
        "test_category": "Haematology",
        "specimen_type": "Whole Blood",
        "test_price": 500.00,
        "price": 500.00,
        "status": "PENDING",
        "barcode": "BC-2419",
        "barcode_id": "7151abcf-5a89-4339-8c6a-b49d21c5084f",
        "token_number": "LAB-2419"
      },
      {
        "order_item_id": "427c9384-0ce8-4599-952a-acf6afb85541",
        "id": "427c9384-0ce8-4599-952a-acf6afb85541",
        "test_id": "1a0ef01c-36ac-4c8f-a451-110f9d60d393",
        "test_name": "Lipid Profile",
        "test_code": "BIO-004",
        "test_category": "Biochemistry",
        "specimen_type": "Serum",
        "test_price": 750.00,
        "price": 750.00,
        "status": "PENDING",
        "barcode": "BC-2419",
        "barcode_id": "7151abcf-5a89-4339-8c6a-b49d21c5084f",
        "token_number": "LAB-2419"
      },
      {
        "order_item_id": "9b183fd4-8b01-4475-ae90-7bbec9267104",
        "id": "9b183fd4-8b01-4475-ae90-7bbec9267104",
        "test_id": "b301fd45-9852-45e0-91cd-ef9034561001",
        "test_name": "HbA1c",
        "test_code": "BIO-008",
        "test_category": "Biochemistry",
        "specimen_type": "Whole Blood",
        "test_price": 500.00,
        "price": 500.00,
        "status": "PENDING",
        "barcode": "BC-2419",
        "barcode_id": "7151abcf-5a89-4339-8c6a-b49d21c5084f",
        "token_number": "LAB-2419"
      }
    ],
    "packages": []
  }
}
```

---

## 1.6 Order Management & Addendum Reports APIs

### 1.6.1 Order Management Summary Statistics (KPI Cards)
Retrieves summary metric counters displayed across top KPI cards on the Order Management dashboard.

* **Endpoint:** `GET /lab/orders/statistics` *(Aliases: `GET /diagnostic-orders/lab/orders/statistics`, `GET /lab/dashboard`)*
* **Required Permission:** `diagnostics:order:view`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "total_orders": 342,
    "pending": 47,
    "completed": 89,
    "in_transit": 28,
    "processing": 163,
    "reviewed": 31
  }
}
```

---

### 1.6.2 Order Management Table List
Retrieves the paginated, searchable, and filterable laboratory orders list.

* **Endpoint:** `GET /lab/orders` *(Alias: `GET /diagnostic-orders/lab/orders`)*
* **Required Permission:** `diagnostics:order:view`
* **Query Parameters:**
  | Parameter | Type | Required? | Description |
  | :--- | :--- | :--- | :--- |
  | `search` / `q` | String | Optional | Search by Patient Name, UHID, Phone, or Barcode |
  | `status` | String | Optional | Filter by status: `Waiting` (`PENDING`), `Inprocess` (`IN_PROGRESS`), `Completed` (`COMPLETED`) |
  | `patient_type` | String | Optional | `Walk-In`, `Hospital`, `Referral` |
  | `collector_id` | UUID | Optional | Filter by phlebotomist / collector staff |
  | `date` | String | Optional | Date filter (`YYYY-MM-DD`) |
  | `page` | Integer | Optional | Page number (Default: `1`) |
  | `limit` | Integer | Optional | Items per page (Default: `10`) |

* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "order_id": "c8542c47-f3e5-4ec0-8f24-b9f7e58c32f2",
        "order_item_id": "df86d618-0977-4579-a061-e75b1d14d4dd",
        "patient_id": "d1d96f70-b397-457c-8954-c8728e03ca0c",
        "patient_name": "Rahul Verma",
        "uhid": "UHID-9821",
        "age": 34,
        "gender": "F",
        "barcode": "BC-881234",
        "test_names": [
          "ESR (Erythrocyte Sedimentation Rate)"
        ],
        "collector": {
          "name": "ARAV SHARMA",
          "role": "Pathologist"
        },
        "patient_type": "Walk-In",
        "status": "Completed",
        "created_at": "2026-04-20T09:30:00Z"
      },
      {
        "order_id": "a9184dca-8319-48fe-99d8-1fc450e060c5",
        "order_item_id": "16a43dca-eb71-4b85-ab0a-3b984816cadf",
        "patient_id": "b16cc357-fb70-404f-aa70-51bb20763cd7",
        "patient_name": "Rajesh Sharma",
        "uhid": "UHID-9821",
        "age": 34,
        "gender": "F",
        "barcode": "BC-881234",
        "test_names": [
          "CBC",
          "ESR"
        ],
        "collector": {
          "name": "ARAV SHARMA",
          "role": "Lab Technician"
        },
        "patient_type": "Hospital",
        "status": "Inprocess",
        "created_at": "2026-04-20T10:15:00Z"
      }
    ],
    "pagination": {
      "total_records": 124,
      "page": 1,
      "limit": 10,
      "total_pages": 13
    }
  }
}
```

---

### 1.6.3 Addendum Reports Table List
Retrieves the paginated and searchable post-validation Addendum Reports audit table.

* **Endpoint:** `GET /lab/addendums` *(Alias: `GET /diagnostic-orders/lab/addendums`)*
* **Required Permission:** `diagnostics:order:view`
* **Query Parameters:**
  | Parameter | Type | Required? | Description |
  | :--- | :--- | :--- | :--- |
  | `search` / `q` | String | Optional | Search by Report ID, Patient Name, or UHID |
  | `status` | String | Optional | Filter by status: `Pending`, `Approved`, `Rejected` |
  | `amendment_type`| String | Optional | `Value Correction`, `Comment Added`, `Test Result Updated`, `Typographical Correction`, `Additional Findings` |
  | `page` | Integer | Optional | Page number (Default: `1`) |
  | `limit` | Integer | Optional | Items per page (Default: `10`) |

* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "addendum_id": "5d78936b-aaa0-457c-beb3-2d13853fc909",
        "report_id": "REP-10245",
        "lab_result_id": "401fcd56-3bd0-4247-8ece-d2cc47b616dd",
        "patient_name": "Rajesh Kumar",
        "patient_uhid": "PAT-2026-6483",
        "original_report_date": "16 Jul 2026",
        "addendum_date": "16 Jul 2026",
        "amendment_type": "Value Correction",
        "reason": "Analyzer recalibration",
        "requested_by": "Dr. Mehta",
        "status": "Approved",
        "created_at": "2026-07-16T14:20:00Z"
      }
    ],
    "pagination": {
      "total_records": 48,
      "page": 1,
      "limit": 10,
      "total_pages": 5
    }
  }
}
---

### 1.6.4 Reschedule Lab Tests & Packages
Reschedules one or more selected tests, order items, or bundled packages under an active order with a revised date, time, and reason. All entity references strictly enforce UUID standards.

* **Endpoint:** `POST /lab/orders/{id}/reschedule` *(Alias: `POST /diagnostic-orders/lab/orders/{id}/reschedule`)*
* **Required Permission:** `diagnostics:order:write`
* **Request Body (JSON):**
```json
{
  "selected_tests": [
    {
      "test_id": "2570848a-74ce-48db-b1f7-1fc450e060c5"
    }
  ],
  "selected_packages": [
    {
      "package_id": "b5d66607-f4de-4ac4-bfed-c9b27adad890"
    }
  ],
  "rescheduled_date": "2026-08-25",
  "rescheduled_time": "10:30 AM",
  "reason": "Patient requested morning fasting slot"
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Lab test(s) rescheduled successfully",
  "data": {
    "order_id": "812ef6f8-ffce-45c0-8000-3979b96c898f",
    "lab_id": "LABID-29354",
    "rescheduled_date": "2026-08-25",
    "rescheduled_time": "10:30 AM",
    "reason": "Patient requested morning fasting slot",
    "rescheduled_items": [
      {
        "order_item_id": "3bb24083-5e74-47cb-aeeb-a1e28f13bed7",
        "test_id": "1a0ef01c-36ac-4c8f-a451-110f9d60d393",
        "test_name": "Special CBC Parameters Verification",
        "barcode": "BC-56817",
        "status": "SCHEDULED",
        "rescheduled_date": "2026-08-25",
        "rescheduled_time": "10:30 AM",
        "reason": "Patient requested morning fasting slot",
        "updated_at": "2026-08-22 13:02:30.659419+05:30"
      },
      {
        "order_item_id": "b2bb9ab9-24f5-4ccc-99f1-715f7731cb2e",
        "test_id": "2570848a-74ce-48db-b1f7-1fc450e060c5",
        "test_name": "Erythrocyte Sedimentation Rate (ESR)",
        "barcode": "BC-56817",
        "status": "SCHEDULED",
        "rescheduled_date": "2026-08-25",
        "rescheduled_time": "10:30 AM",
        "reason": "Patient requested morning fasting slot",
        "updated_at": "2026-08-22 13:02:30.659419+05:30"
      }
    ],
    "rescheduled_packages": [
      {
        "package_id": "b5d66607-f4de-4ac4-bfed-c9b27adad890"
      }
    ]
  }
}
```

---

### 1.6.5 Cancel Lab Tests & Orders
Cancels specified tests, packages, or an entire diagnostic order with a mandatory cancellation audit reason. All entity references strictly enforce UUID standards.

* **Endpoint:** `POST /lab/orders/{id}/cancel` *(Alias: `POST /diagnostic-orders/lab/orders/{id}/cancel`)*
* **Required Permission:** `diagnostics:order:write`
* **Request Body (JSON):**
```json
{
  "selected_tests": [
    {
      "test_id": "2570848a-74ce-48db-b1f7-1fc450e060c5"
    }
  ],
  "reason": "Patient decided not to take individual ESR test"
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Lab test(s) cancelled successfully",
  "data": {
    "order_id": "85fc6d09-9da0-4bbc-9482-832113d3998b",
    "order_number": "LABID-29360",
    "lab_id": "LABID-29360",
    "order_status": "PARTIALLY_CANCELLED",
    "cancellation_reason": "Patient decided not to take individual ESR test",
    "payment_details": {
      "payment_method": "UPI",
      "reference_no": "UPI-TXN-2026-9911",
      "invoice_number": "INV-LAB-BDEB8852",
      "original_total_amount": 1829.00,
      "original_amount_paid": 1829.00,
      "previous_outstanding": 0.00
    },
    "refund_details": {
      "refund_amount": 413.00,
      "refund_status": "REFUND_DUE",
      "refund_mode": "UPI (Original Payment Method)",
      "refund_reference_no": "REF-UPI-27855955",
      "outstanding_balance_adjusted": 0.00,
      "remaining_order_balance": 1416.00
    },
    "cancelled_items": [
      {
        "order_item_id": "0b2de937-4e21-40c1-aa41-65bb4d59cd07",
        "test_id": "2570848a-74ce-48db-b1f7-1fc450e060c5",
        "test_name": "Erythrocyte Sedimentation Rate (ESR)",
        "test_price": 350.00,
        "refund_amount": 413.00,
        "status": "CANCELLED",
        "reason": "Patient decided not to take individual ESR test",
        "cancelled_at": "2026-08-22 13:06:29.213111+05:30"
      }
    ],
    "cancelled_packages": []
  }
}
---

### 1.7 Patient Bills & Invoices APIs

#### 1.7.1 List All Bills for a Specific Patient
Retrieves all historical and active bills, payment receipts, and outstanding dues for a specific patient.

* **Endpoint:** `GET /lab/patients/{id}/bills` *(Aliases: `GET /lab/patients/{id}/invoices`, `GET /diagnostic-orders/lab/patients/{id}/bills`)*
* **Required Permission:** `diagnostics:billing:view`
* **Query Parameters:** `status`, `payment_method`, `start_date`, `end_date`, `page`, `limit`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "bill_id": "621b59c1-f2e0-44c0-b60b-e9f2a18de59f",
        "invoice_id": "621b59c1-f2e0-44c0-b60b-e9f2a18de59f",
        "invoice_number": "INV-LAB-D4BAB345",
        "patient_id": "96077f2b-2fae-4f8d-999a-742d6a9d290d",
        "patient_name": "Rajesh Kumar",
        "patient_uhid": "PAT-2026-6495",
        "patient_age": 42,
        "patient_gender": "MALE",
        "patient_phone": "9823461993",
        "rx_id": "fad29ebf-f072-4f4f-9e46-0ab1663b7e71",
        "order_number": "LABID-29372",
        "rx_number": "LABID-29372",
        "payment_method": "UPI",
        "payment_mode": "UPI",
        "amount": 1829.00,
        "total_amount": 1829.00,
        "subtotal_amount": 1550.00,
        "discount_amount": 0.00,
        "tax_amount": 279.00,
        "paid_amount": 1829.00,
        "outstanding_amount": 0.00,
        "status": "PAID",
        "invoice_date": "2026-08-22",
        "created_at": "2026-08-22T13:25:41.733580+05:30"
      }
    ],
    "total": 1
  }
}
```

#### 1.7.2 General Invoices Endpoint with Query Filters
Lists all diagnostic bills and invoices across the laboratory with support for patient filtering (`patient_id`, `uhid`), search keywords, status, and date range.

* **Endpoint:** `GET /lab/invoices`
* **Gateway Route Aliases:** `GET /lab/billing/invoices`, `GET /lab/bills`, `GET /diagnostic-orders/billing/invoices`
* **Required Permission:** `diagnostics:billing:view`
* **Filter by Patient UUID:** `GET /lab/invoices?patient_id=96077f2b-2fae-4f8d-999a-742d6a9d290d`
* **Filter by UHID:** `GET /lab/invoices?uhid=PAT-2026-6495`
* **Filter by Search:** `GET /lab/invoices?search=Rajesh`
* **Filter by Status & Mode:** `GET /lab/invoices?patient_id=96077f2b-2fae-4f8d-999a-742d6a9d290d&status=PAID&payment_method=UPI&page=1&limit=10`

##### Success Response (200 OK)
```json
{
  "success": true,
  "code": 200,
  "data": {
    "items": [
      {
        "bill_id": "621b59c1-f2e0-44c0-b60b-e9f2a18de59f",
        "invoice_id": "621b59c1-f2e0-44c0-b60b-e9f2a18de59f",
        "invoice_number": "INV-LAB-D4BAB345",
        "patient_id": "96077f2b-2fae-4f8d-999a-742d6a9d290d",
        "patient_name": "Rajesh Kumar",
        "patient_uhid": "PAT-2026-6495",
        "patient_age": 42,
        "patient_gender": "MALE",
        "patient_phone": "9823461993",
        "rx_id": "fad29ebf-f072-4f4f-9e46-0ab1663b7e71",
        "order_number": "LABID-29372",
        "rx_number": "LABID-29372",
        "payment_method": "UPI",
        "payment_mode": "UPI",
        "amount": 1829.00,
        "total_amount": 1829.00,
        "subtotal_amount": 1550.00,
        "discount_amount": 0.00,
        "tax_amount": 279.00,
        "paid_amount": 1829.00,
        "outstanding_amount": 0.00,
        "status": "PAID",
        "invoice_date": "2026-08-22",
        "created_at": "2026-08-22T13:25:41.733580+05:30"
      },
      {
        "bill_id": "fcf20c85-b528-4765-9701-a9f643a75f33",
        "invoice_id": "fcf20c85-b528-4765-9701-a9f643a75f33",
        "invoice_number": "INV-LAB-83C689FB",
        "patient_id": "96077f2b-2fae-4f8d-999a-742d6a9d290d",
        "patient_name": "Rajesh Kumar",
        "patient_uhid": "PAT-2026-6495",
        "patient_age": 42,
        "patient_gender": "MALE",
        "patient_phone": "9823461993",
        "rx_id": "709af13a-22fd-405f-9300-2e8fa865daf5",
        "order_number": "LABID-29371",
        "rx_number": "LABID-29371",
        "payment_method": null,
        "payment_mode": null,
        "amount": 1829.00,
        "total_amount": 1829.00,
        "subtotal_amount": 1550.00,
        "discount_amount": 0.00,
        "tax_amount": 279.00,
        "paid_amount": 0.00,
        "outstanding_amount": 1829.00,
        "status": "OPEN",
        "invoice_date": "2026-08-22",
        "created_at": "2026-08-22T13:25:40.231401+05:30"
      }
    ],
    "total": 2
  }
}
```

#### 1.7.3 "Filter Options" Modal Integration
Direct mapping between the UI Filter Options modal and API query parameters:

* **Status Options (`status`):** `Completed`, `Pending`, `Paid`, `Cancelled` (supports multi-selection via comma: `status=Paid,Completed`).
* **Date Range (`from_date`, `to_date`):** Format `YYYY-MM-DD` (aliases: `startDate`, `endDate`).
* **Payment Method (`payment_method`):** `UPI`, `Card`, `Cash` (supports multi-selection via comma: `payment_method=UPI,Card`).

#### 1.7.4 Download / Print Bill Receipt (`GET /lab/invoices/{id}`)
Retrieves the complete receipt structure required for downloading/printing the **LAB REGISTRATION & TEST ORDER** invoice:

* **Endpoint:** `GET /lab/invoices/{id}`
* **Gateway Route Aliases:** `GET /lab/bills/{id}`, `GET /diagnostic-orders/billing/invoices/{id}`
* **Required Permission:** `diagnostics:billing:view`
* **Response Details:** Returns `hospital_info`, `patient_info` (with `age_gender`, contact), `lab_info` (with report ID, barcode `BC-2419`, pathologist, `special_instructions`, `fasting_required`), `billing_summary` (subtotal, 18% GST, discount, total amount paid), full `tests` array (with test code, category, specimen type, and individual test price), `instructions` (dynamic array containing custom order preparation instructions, fasting advisories, and document tracking notes), and `authorized_representative` note.

##### Example Request:
```http
GET /lab/invoices/1072c717-eeee-4165-8c22-983e6703f599
Authorization: Bearer <access_token>
```

#### 1.7.5 Sample Queue & Active Filters Modal (`GET /lab/sample-queue`)
Powers the **Sample Queue** table and its **Active Filters Modal**:

* **Endpoint:** `GET /lab/sample-queue`
* **Gateway Route Aliases:** `GET /lab/samples`, `GET /lab/orders/samples`, `GET /diagnostic-orders/lab/samples`, `GET /diagnostic-orders/lab/sample-queue`
* **Required Permission:** `diagnostics:order:view`
* **Behavioral Defaults & Rules:**
  * **Default View:** Automatically filters for tests registered on the **Current Day** (`CURRENT_DATE`).
  * **Default Ordering:** Displays **Latest Orders First** (`ORDER BY created_at DESC`).
  * **Past History Access:** Users can query any past date/range (`from_date`, `to_date`, `date`) or all history (`all_time=true`).
  * **Future Date Restriction:** Any query for a future date (e.g. tomorrow or next week) automatically returns an **empty queue** (`total: 0`, `items: []`).
* **Query Filters Supported:**
  * **Search (`search` / `q`):** Order ID (`ORD-10482`, `LABID-29184`), Patient Name, UHID, Barcode, Phone, Test Name.
  * **Test Type (`test_type` / `category`):** Dropdown filter (`All Test Types`, `Haematology`, `Biochemistry`, `CBC + ESR`, `Lipid Profile`, `Urine Routine`).
  * **Priority (`priority`):** `STAT (Urgent)`, `Routine` (multi-select: `priority=STAT,Routine`).
  * **Status (`status`):** `Pending`, `Processing`, `In-Process`, `Verification`, `Completed`, `Waiting` (multi-select: `status=Processing,Verification`).
  * **Date Range (`from_date`, `to_date`, `date`, `all_time`):** Format `YYYY-MM-DD`. Defaults to Today.
  * **Pagination (`page`, `limit`):** Configurable rows per page (e.g. `limit=5`).
* **Table Column Outputs:** `order_id`, `patient_name`, `test_type`, `date_time`, `status` (`Completed`, `In-Process`, `Waiting`, `Pending`, `Verification`), `priority` (`STAT (Urgent)`, `Routine`), `raw_order_id` (action link `👁`), and `pagination` metadata.

##### Example Request:
```

#### 1.7.6 Laboratory Packages & Multi-Test Bundles (`GET /lab/packages`)
Lists pre-configured multi-test health packages (e.g. Executive Health Checkup, Diabetic Care, Cardiac Profile) with test counts and bundled child tests:

* **Endpoint:** `GET /lab/packages` (List/Search) and `GET /lab/packages/{id}` (Details)
* **Gateway Route Aliases:** `GET /diagnostic-orders/lab/packages`, `GET /lab/catalog?type=packages`
* **Required Permission:** `diagnostics:order:view` or `config.read`
* **Query Parameters:** `search` (name or code), `page`, `limit`.

##### Example Request:
```http
GET /lab/packages?search=Executive&limit=10
Authorization: Bearer <access_token>
```

---

## 2. Laboratory Operations & Workflow Endpoints

These endpoints manage the workflow transitions of a lab order item:
`PENDING` -> `IN_PROGRESS` -> `REPORT_READY` -> `VALIDATED` / `COMPLETED` (or `CANCELLED` / `ADDENDUM`)

### 2.0.1 Start Testing & Process Sample (Move from SCHEDULED to IN_PROGRESS)
Transitions an individual lab test item from `SCHEDULED` to `IN_PROGRESS` when analyzer testing begins.

* **Endpoint:** `PATCH /lab/samples/{id}/process`
* **Gateway Route Aliases:** `PATCH /diagnostic-orders/lab/samples/{id}/process`, `POST /lab/samples/{id}/process`, `PATCH /lab/items/{id}/process`
* **Required Permission:** `diagnostics:lab:collect` or `diagnostics:order:write`
* **Request Body (JSON):**
  | Field | Type | Requirement | Description |
  | :--- | :--- | :--- | :--- |
  | `machine_name` | `string` | Optional | Analyzer / Bench name (e.g. `"Sysmex XN-1000"`). Saved directly without requiring pre-configured machine setup. |
  | `notes` | `string` | Optional | Technician bench processing notes / remarks. |
  | `machine_id` | `UUID` | Optional | Analyzer machine UUID (if configured). |

##### Example Request:
```http
PATCH /lab/samples/d9b609af-1d54-4a11-a24d-46e4f6feab4d/process
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "machine_name": "Sysmex XN-1000",
  "notes": "Sample loaded onto Sysmex XN-1000 for CBC analysis"
}
```

### 2.1 Get Lab Order Details
Retrieves diagnostic order header details along with enriched lab items and results.

* **Endpoint:** `GET /diagnostic-orders/lab/{id}`
* **Required Permission:** `diagnostics:order:view`
* **Path Parameters:**
  | Parameter | Type | Required? | Description |
  | :--- | :--- | :--- | :--- |
  | `id` | UUID | **Mandatory** | The diagnostic order identifier |

* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "c8542c47-f3e5-4ec0-8f24-b9f7e58c32f2",
    "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "patient_id": "d1d96f70-b397-457c-8954-c8728e03ca0c",
    "patient_name": "Karan Sharma",
    "patient_mrn": "UH-83821",
    "doctor_id": "doc-uuid-2222",
    "doctor_name": "Dr. Ramesh Nair",
    "status": "IN_PROGRESS",
    "order_type": "LAB",
    "created_at": "2026-06-23T10:40:00Z",
    "lab_items": [
      {
        "id": "df86d618-0977-4579-a061-e75b1d14d4dd",
        "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
        "diagnostic_order_id": "c8542c47-f3e5-4ec0-8f24-b9f7e58c32f2",
        "test_id": "16a43dca-eb71-4b85-ab0a-3b984816cadf",
        "test_name": "Complete Blood Count",
        "test_code": "CBC",
        "test_category": "Hematology",
        "specimen_type": "Whole Blood",
        "test_price": 350.00,
        "status": "SAMPLE_COLLECTED",
        "barcode": "BAR-10029",
        "notes": "Fast track processing requested",
        "sample_collected_at": "2026-06-23T11:00:00Z",
        "sample_collected_by": "user-uuid-tech",
        "sample_collected_by_name": "Amit Sharma",
        "processed_at": null,
        "processed_by": null,
        "result": null,
        "created_at": "2026-06-23T10:40:00Z",
        "updated_at": "2026-06-23T11:00:00Z"
      }
    ]
  }
}
```

---

### 2.2 Collect Sample
Updates status to `SAMPLE_COLLECTED` and assigns the container barcode.

* **Endpoint:** `PATCH /diagnostic-orders/lab/{id}/collect-sample`
* **Required Permission:** `diagnostics:lab:collect`
* **Path Parameters:**
  | Parameter | Type | Required? | Description |
  | :--- | :--- | :--- | :--- |
  | `id` | UUID | **Mandatory** | The lab order item identifier |

* **Request Body (JSON):**
  | Field | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `barcode` | String | **Mandatory** | Physical container barcode | min length 1, max length 60 |
  | `notes` | String | Optional | Collection notes/remarks | max length 500 |

* **Example Request:**
```json
{
  "barcode": "BAR-10029",
  "notes": "Patient fasted for 12 hours"
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Sample collected successfully",
  "data": {
    "id": "df86d618-0977-4579-a061-e75b1d14d4dd",
    "status": "SAMPLE_COLLECTED",
    "barcode": "BAR-10029",
    "notes": "Patient fasted for 12 hours",
    "sample_collected_at": "2026-06-23T11:00:00Z",
    "sample_collected_by": "user-uuid-tech",
    "updated_at": "2026-06-23T11:00:00Z"
  }
}
```

---

### 2.3 Process Sample
Updates status to `IN_PROGRESS` signifying processing of specimen has started.

* **Endpoint:** `PATCH /diagnostic-orders/lab/{id}/process`
* **Required Permission:** `diagnostics:lab:collect`
* **Path Parameters:**
  | Parameter | Type | Required? | Description |
  | :--- | :--- | :--- | :--- |
  | `id` | UUID | **Mandatory** | The lab order item identifier |

* **Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Sample processing started",
  "data": {
    "id": "df86d618-0977-4579-a061-e75b1d14d4dd",
    "status": "IN_PROGRESS",
    "processed_at": "2026-06-23T11:15:00Z",
    "processed_by": "user-uuid-tech",
    "updated_at": "2026-06-23T11:15:00Z"
  }
}
```

---

### 2.4 Enter Result
Saves the results of the lab test and transitions the item status to `COMPLETED`.

* **Endpoint:** `POST /diagnostic-orders/lab/results/{item_id}`
* **Required Permission:** `diagnostics:lab:result:enter`
* **Path Parameters:**
  | Parameter | Type | Required? | Description |
  | :--- | :--- | :--- | :--- |
  | `item_id` | UUID | **Mandatory** | The lab order item identifier |

* **Request Body (JSON):**
  | Field | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `result_value` | String | Optional | Numeric/textual single test value | |
  | `result_json` | Object | Optional | Structured parameters/readings in JSON | |
  | `report_url` | String | Optional | URL to report file upload (e.g. S3) | |
  | `abnormal_flag`| Boolean | Optional | Flag if value is outside normal range | Default: `false` |
  | `critical_flag`| Boolean | Optional | Flag if value is critically high/low | Default: `false` |
  | `comments` | String | Optional | General observations / technician notes | Max length 1000 |

* **Example Request:**
```json
{
  "result_value": "13.8",
  "result_json": {
    "hemoglobin": 13.8,
    "wbc": 7200,
    "platelets": 250000
  },
  "abnormal_flag": false,
  "critical_flag": false,
  "comments": "CBC looks normal"
}
```
* **Example Response (201 Created):**
```json
{
  "success": true,
  "message": "Lab test result saved successfully",
  "data": {
    "id": "result-uuid-001122",
    "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "lab_order_item_id": "df86d618-0977-4579-a061-e75b1d14d4dd",
    "result_value": "13.8",
    "result_json": {
      "hemoglobin": 13.8,
      "wbc": 7200,
      "platelets": 250000
    },
    "report_url": null,
    "abnormal_flag": false,
    "critical_flag": false,
    "entered_by": "user-uuid-tech",
    "comments": "CBC looks normal",
    "created_at": "2026-06-23T11:45:00Z",
    "updated_at": "2026-06-23T11:45:00Z"
  }
}
```

---

### 2.5 Get Lab Result
Retrieves a lab result by either the result UUID or the lab order item UUID.

* **Endpoint:** `GET /diagnostic-orders/lab/results/{id}`
* **Required Permission:** `diagnostics:order:view`
* **Path Parameters:**
  | Parameter | Type | Required? | Description |
  | :--- | :--- | :--- | :--- |
  | `id` | UUID | **Mandatory** | The result identifier or the lab order item identifier |

* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "result-uuid-001122",
    "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "lab_order_item_id": "df86d618-0977-4579-a061-e75b1d14d4dd",
    "result_value": "13.8",
    "result_json": {
      "hemoglobin": 13.8,
      "wbc": 7200,
      "platelets": 250000
    },
    "report_url": null,
    "abnormal_flag": false,
    "critical_flag": false,
    "entered_by": "user-uuid-tech",
    "validated_by": null,
    "validated_at": null,
    "comments": "CBC looks normal",
    "created_at": "2026-06-23T11:45:00Z",
    "updated_at": "2026-06-23T11:45:00Z"
  }
}
```

---

### 2.6 Validate Result
Clinician signs off on the test results. **Note: This action automatically triggers the deduction of the mapped reagents/consumables from the stock.**

* **Endpoint:** `PATCH /diagnostic-orders/lab/results/{id}/validate`
* **Required Permission:** `diagnostics:lab:result:validate`
* **Path Parameters:**
  | Parameter | Type | Required? | Description |
  | :--- | :--- | :--- | :--- |
  | `id` | UUID | **Mandatory** | The result identifier |

* **Request Body (JSON):**
  | Field | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `comments` | String | Optional | Pathologist clinical sign-off notes | Max length 1000 |

* **Example Request:**
```json
{
  "comments": "Results validated and released."
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Lab report validated successfully",
  "data": {
    "id": "result-uuid-001122",
    "branch_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "lab_order_item_id": "df86d618-0977-4579-a061-e75b1d14d4dd",
    "validated_by": "doc-uuid-pathologist",
    "validated_at": "2026-06-23T12:00:00Z",
    "comments": "Results validated and released.",
    "updated_at": "2026-06-23T12:00:00Z"
  }
}
```

---

### 2.7 Cancel Lab Item
Cancels an active lab order item, providing a reason audit trail.

* **Endpoint:** `PATCH /diagnostic-orders/lab/{id}/cancel`
* **Required Permission:** `diagnostics:lab:collect`
* **Path Parameters:**
  | Parameter | Type | Required? | Description |
  | :--- | :--- | :--- | :--- |
  | `id` | UUID | **Mandatory** | The lab order item identifier |

* **Request Body (JSON):**
  | Field | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `reason` | String | **Mandatory** | Reason for cancellation | |

* **Example Request:**
```json
{
  "reason": "Specimen hemolyzed; patient refused second draw."
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Lab item cancelled successfully",
  "data": {
    "id": "df86d618-0977-4579-a061-e75b1d14d4dd",
    "status": "CANCELLED",
    "updated_at": "2026-06-23T12:10:00Z"
  }
}
```

---

## 3. Dashboard & Catalog Lookups

### 3.1 Get Lab Dashboard Statistics
Retrieves operational dashboard KPIs (orders, samples, results counts and alert states).

* **Endpoint:** `GET /diagnostic-orders/lab/dashboard`
* **Required Permission:** None (Authenticated only)
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "pending_orders": 8,
    "in_progress_orders": 12,
    "completed_orders": 45,
    "cancelled_orders": 3,
    "pending_samples": 4,
    "sample_collected": 6,
    "in_progress": 8,
    "completed": 45,
    "cancelled": 3,
    "total_results": 45,
    "abnormal_results": 7,
    "critical_results": 2,
    "pending_validation": 5
  }
}
```

---

### 3.2 List Lab Tests
Lists clinical lab tests available in the diagnostics catalog.

* **Endpoint:** `GET /diagnostic-orders/lab/tests`
* **Required Permission:** None (Authenticated only)
* **Query Parameters:**
  | Parameter | Type | Required? | Description |
  | :--- | :--- | :--- | :--- |
  | `category` | String | Optional | Filter by test category (e.g. `Biochemistry`, `Hematology`) |

* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": "16a43dca-eb71-4b85-ab0a-3b984816cadf",
      "test_name": "Complete Blood Count",
      "test_code": "CBC",
      "category": "Hematology",
      "specimen_type": "Whole Blood",
      "price": 350.00,
      "is_active": true
    }
  ]
}
```

---

## 4. Laboratory Catalog & Stock Management

These endpoints handle the item master catalog, stock mapping to test procedures, stock deduction, and quality controls.

### 4.1 List Lab Inventory
List items registered in the lab inventory master.

* **Endpoint:** `GET /diagnostic-orders/lab/inventory`
* **Required Permission:** `diagnostics:order:view`
* **Query Parameters:**
  | Parameter | Type | Required? | Description |
  | :--- | :--- | :--- | :--- |
  | `search` | String | Optional | Match text against item name or item code |
  | `category` | String | Optional | Filter by category (e.g. `REAGENT`, `CONSUMABLE`) |
  | `section` | String | Optional | Filter by lab section (e.g. `Haematology`, `Biochemistry`) |

* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": "df86d618-0977-4579-a061-e75b1d14d4dd",
      "tenant_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
      "item_code": "LAB-RE-CBC-001",
      "item_name": "Hematology Reagent Diluent",
      "category": "REAGENT",
      "sub_category": null,
      "hazard_class": "NONE",
      "uom": "BOTTLE",
      "pack_size": "5L",
      "current_stock": 10,
      "reorder_level": 10,
      "minimum_stock": 5,
      "assigned_section": "Haematology",
      "is_active": true,
      "created_at": "2026-06-29T13:20:10Z",
      "updated_at": "2026-06-29T13:20:25Z"
    }
  ]
}
```

---

### 4.2 Create Lab Catalog Item
Creates a new reagent, consumable, or spare part item in the inventory catalog.

* **Endpoint:** `POST /diagnostic-orders/lab/inventory`
* **Required Permission:** `diagnostics:test:manage`
* **Request Body (JSON):**
  | Field | Type | Required? | Description | Constraints / Default |
  | :--- | :--- | :--- | :--- | :--- |
  | `item_code` | String | **Mandatory** | Unique alphanumeric item code | |
  | `item_name` | String | **Mandatory** | Descriptive item name | |
  | `category` | String | **Mandatory** | Category classifier | e.g. `REAGENT`, `CONSUMABLE`, `SPARE` |
  | `assigned_section` | String | Optional | Lab division section | e.g. `Haematology`, `Microbiology` |
  | `uom` | String | Optional | Unit of measure | Default: `"PIECE"` |
  | `reorder_level` | Integer | Optional | Threshold to flag warning | Default: `0` |
  | `minimum_stock` | Integer | Optional | Safety stock level | Default: `0` |
  | `maximum_stock` | Integer | Optional | Limit stock level | |
  | `sub_category` | String | Optional | Sub-classification | |
  | `description` | String | Optional | Detailed item specification | |
  | `hazard_class` | String | Optional | Hazard rating | Default: `"NONE"` |
  | `msds_available` | Boolean | Optional | MSDS checklist sheet | Default: `false` |
  | `requires_coc` | Boolean | Optional | Certificate of Conformance req | Default: `false` |
  | `pack_size` | String | Optional | Box or bottle package contents | |
  | `tests_per_unit` | Integer | Optional | Estimated tests supported per UOM | |
  | `storage_condition` | String | Optional | Storage location standard | Default: `"ROOM_TEMPERATURE"` |
  | `storage_temperature_min`| Float | Optional | Min allowed storage temperature | |
  | `storage_temperature_max`| Float | Optional | Max allowed storage temperature | |
  | `is_light_sensitive` | Boolean | Optional | Light sensitivity flag | Default: `false` |
  | `is_moisture_sensitive` | Boolean | Optional | Moisture sensitivity flag | Default: `false` |
  | `shelf_life_months` | Integer | Optional | Lifespan in months | |
  | `brand` | String | Optional | Manufacturer brand label | |
  | `manufacturer` | String | Optional | Maker organization name | |
  | `catalogue_number` | String | Optional | Supplier cat number | |
  | `barcode` | String | Optional | Item static barcode | |
  | `preferred_vendor_id` | UUID | Optional | Preferred vendor identifier | |
  | `mrp` | Float | Optional | Maximum Retail Price | |
  | `gst_percent` | Float | Optional | Tax rate | |
  | `lead_time_days` | Integer | Optional | Standard procurement days | Default: `7` |
  | `warehouse_id` | UUID | Optional | Warehouse location ID | |
  | `rack_shelf_bin` | String | Optional | Shelf bin path coordinates | |

* **Example Request:**
```json
{
  "item_code": "LAB-RE-CBC-001",
  "item_name": "Hematology Reagent Diluent",
  "category": "REAGENT",
  "assigned_section": "Haematology",
  "uom": "BOTTLE",
  "reorder_level": 10,
  "minimum_stock": 5,
  "storage_condition": "REFRIGERATED",
  "storage_temperature_min": 2.0,
  "storage_temperature_max": 8.0,
  "lead_time_days": 5
}
```
* **Example Response (201 Created):**
```json
{
  "success": true,
  "message": "Lab inventory item created successfully",
  "data": {
    "id": "df86d618-0977-4579-a061-e75b1d14d4dd",
    "item_code": "LAB-RE-CBC-001",
    "item_name": "Hematology Reagent Diluent",
    "category": "REAGENT",
    "assigned_section": "Haematology",
    "uom": "BOTTLE",
    "reorder_level": 10.0,
    "minimum_stock": 5.0,
    "storage_condition": "REFRIGERATED",
    "is_active": true,
    "created_at": "2026-06-29T13:20:10Z"
  }
}
```

---

### 4.3 Get Stock Alerts
Fetches inventory items that have crossed their reorder or minimum levels, along with overall catalog health statistics.

* **Endpoint:** `GET /diagnostic-orders/lab/inventory/alerts`
* **Required Permission:** `diagnostics:dashboard`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "stock_health_percentage": 75,
    "items": [
      {
        "item_name": "Hematology Reagent Diluent",
        "status": "WARNING",
        "current_stock": 10,
        "min_stock": 5,
        "uom": "BOTTLE"
      },
      {
        "item_name": "EDTA Tubes 3mL",
        "status": "CRITICAL",
        "current_stock": 2,
        "min_stock": 100,
        "uom": "PIECE"
      }
    ]
  }
}
```

---

### 4.4 Consume Lab Items
Manually logs consumption of catalog items (e.g. for training, manual tests, wastage write-offs).

* **Endpoint:** `POST /diagnostic-orders/lab/inventory/consume`
* **Required Permission:** `diagnostics:lab:collect`
* **Request Body (JSON):**
  | Field | Type | Required? | Description | Constraints / Default |
  | :--- | :--- | :--- | :--- | :--- |
  | `consumption_type` | String | Optional | Purpose of consumption | e.g. `"MANUAL"`, `"WASTAGE"`, `"QC"`, `"TEST"`. Default: `"MANUAL"` |
  | `notes` | String | Optional | Processing remarks | |
  | `lab_order_item_id` | UUID | Optional | Associated patient order item ID | |
  | `items` | Array | **Mandatory** | List of consumed stock items | Must contain at least one item |

* **LabConsumeItemInput structure:**
  | Field | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `item_id` | UUID | **Mandatory** | Inventory catalog item ID | |
  | `quantity` | Float | **Mandatory** | Quantity to consume/deduct | Must be > 0.0 |

* **Example Request:**
```json
{
  "consumption_type": "WASTAGE",
  "notes": "Accidental spill of diluent bottle",
  "items": [
    {
      "item_id": "df86d618-0977-4579-a061-e75b1d14d4dd",
      "quantity": 1.0
    }
  ]
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Lab items consumed successfully",
  "data": {
    "success": true,
    "message": "Lab consumables consumed successfully"
  }
}
```

---

### 4.5 Create Lab Test Mapping
Maps a catalog inventory item (e.g. chemical reagent) to a clinical lab test procedure. This enables **automatic FEFO-based stock deduction** when a result is validated.

* **Endpoint:** `POST /diagnostic-orders/lab/inventory/mappings`
* **Required Permission:** `diagnostics:test:manage`
* **Request Body (JSON):**
  | Field | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `item_id` | UUID | **Mandatory** | Master catalog inventory item ID | |
  | `test_id` | UUID | **Mandatory** | Clinical laboratory test catalog ID | |
  | `qty_per_test` | Float | **Mandatory** | Unit consumption quantity | Must be > 0.0 |
  | `is_optional` | Boolean | Optional | Whether this item consumption is optional | Default: `false` |

* **Example Request:**
```json
{
  "item_id": "df86d618-0977-4579-a061-e75b1d14d4dd",
  "test_id": "16a43dca-eb71-4b85-ab0a-3b984816cadf",
  "qty_per_test": 0.05,
  "is_optional": false
}
```
* **Example Response (201 Created):**
```json
{
  "success": true,
  "message": "Lab item test mapping created successfully",
  "data": {
    "id": "mapping-uuid-9988",
    "item_id": "df86d618-0977-4579-a061-e75b1d14d4dd",
    "test_id": "16a43dca-eb71-4b85-ab0a-3b984816cadf",
    "qty_per_test": 0.05,
    "is_optional": false
  }
}
```

---

### 4.6 Create Lab QC Record
Records a quality control calibrator or lot run result. **Note: If a QC reading falls outside the acceptable minimum and maximum range, a `QC_FAILURE` alert is automatically triggered.**

* **Endpoint:** `POST /diagnostic-orders/lab/qc-records`
* **Required Permission:** `diagnostics:lab:result:enter`
* **Request Body (JSON):**
  | Field | Type | Required? | Description | Constraints / Default |
  | :--- | :--- | :--- | :--- | :--- |
  | `item_id` | UUID | Optional | Associated inventory item ID | |
  | `batch_id` | UUID | Optional | Associated batch lot record ID | |
  | `test_id` | UUID | Optional | Clinical lab test catalog ID | |
  | `equipment_name` | String | Optional | Instrument/analyzer equipment label | |
  | `lab_section` | String | Optional | Lab division section | |
  | `qc_level` | String | Optional | QC control tier | `"LOW"`, `"NORMAL"`, `"HIGH"`. Default: `"NORMAL"` |
  | `expected_value` | Float | Optional | Expected benchmark control value | |
  | `observed_value` | Float | Optional | Experimentally observed value | |
  | `acceptable_range_min`| Float | Optional | Minimum range constraint limit | |
  | `acceptable_range_max`| Float | Optional | Maximum range constraint limit | |
  | `sd` | Float | Optional | Standard deviation statistical value | |
  | `cv_percent` | Float | Optional | Coefficient of variation percentage | |
  | `result` | String | Optional | Outcome code (auto-set if values provided) | `"PASS"`, `"FAIL"`. Default: `"PENDING"` |
  | `corrective_action` | String | Optional | Notes on troubleshooting | |
  | `westgard_rule_violated`| String | Optional | Westgard statistical rule code triggered | |

* **Example Request:**
```json
{
  "item_id": "df86d618-0977-4579-a061-e75b1d14d4dd",
  "qc_level": "NORMAL",
  "expected_value": 5.0,
  "observed_value": 4.2,
  "acceptable_range_min": 4.5,
  "acceptable_range_max": 5.5,
  "equipment_name": "Sysmex XN-1000"
}
```
* **Example Response (201 Created):**
```json
{
  "success": true,
  "message": "QC record saved successfully",
  "data": {
    "id": "qc-uuid-882233",
    "item_id": "df86d618-0977-4579-a061-e75b1d14d4dd",
    "qc_level": "NORMAL",
    "expected_value": 5.0,
    "observed_value": 4.2,
    "acceptable_range_min": 4.5,
    "acceptable_range_max": 5.5,
    "result": "FAIL",
    "performed_by": "user-uuid-tech",
    "performed_at": "2026-06-29T14:00:00Z"
  }
}
```

---

## 5. Reagents Procurement Lifecycle

These endpoints manage Purchase Orders, Goods Received Notes (which automatically increment inventory stock using First-Expiry-First-Out batches), and Vendor Invoices.

### 5.1 Create Purchase Order (PO)
Initiates a requisition request for lab items.

* **Endpoint:** `POST /diagnostic-orders/lab/procurement/po`
* **Required Permission:** `diagnostics:test:manage`
* **Request Body (JSON):**
  | Field | Type | Required? | Description | Constraints / Default |
  | :--- | :--- | :--- | :--- | :--- |
  | `vendor_id` | UUID | Optional | Target vendor UUID | |
  | `priority` | String | Optional | Urgency level | `"ROUTINE"`, `"MEDIUM"`, `"HIGH"`, `"EMERGENCY"`. Default: `"MEDIUM"` |
  | `lab_section` | String | Optional | Requesting lab department | e.g. `"Haematology"` |
  | `store_location` | String | Optional | Target storage warehouse | |
  | `delivery_address`| String | Optional | Delivery destination address | |
  | `expected_delivery`| Date | Optional | Expected arrival date (`YYYY-MM-DD`) | |
  | `notes` | String | Optional | Requisition comments | |
  | `items` | Array | **Mandatory** | Purchase line items list | Must contain at least one item |

* **LabPOLineItemCreate structure:**
  | Field | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `item_id` | UUID | Optional | Inventory item ID | |
  | `item_name` | String | **Mandatory** | Name / description of item | |
  | `category` | String | Optional | Classification of line item | |
  | `suggested_qty` | Integer | Optional | Auto-calculated restock count | Default: `0` |
  | `final_qty` | Integer | **Mandatory** | Quantity ordered | Must be > 0 |
  | `uom` | String | Optional | Unit of measure | Default: `"PIECE"` |
  | `unit_price` | Float | **Mandatory** | Unit cost price | Must be >= 0.0 |
  | `gst_percent` | Float | Optional | Tax component percentage | Default: `0.0` |

* **Example Request:**
```json
{
  "vendor_id": "vendor-uuid-777",
  "priority": "HIGH",
  "lab_section": "Haematology",
  "items": [
    {
      "item_id": "df86d618-0977-4579-a061-e75b1d14d4dd",
      "item_name": "Hematology Reagent Diluent",
      "final_qty": 20,
      "unit_price": 150.00,
      "gst_percent": 18.0
    }
  ]
}
```
* **Example Response (201 Created):**
```json
{
  "success": true,
  "message": "Purchase Order created successfully",
  "data": {
    "id": "po-uuid-1111",
    "po_number": "PO-LAB-00001",
    "status": "DRAFT",
    "priority": "HIGH",
    "estimated_value": 3000.00,
    "gst_amount": 540.00,
    "grand_total": 3540.00,
    "created_at": "2026-06-29T14:30:00Z"
  }
}
```

---

### 5.2 List Purchase Orders
Retrieves all Purchase Orders registered for the branch.

* **Endpoint:** `GET /diagnostic-orders/lab/procurement/po`
* **Required Permission:** `diagnostics:order:view`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": "po-uuid-1111",
      "po_number": "PO-LAB-00001",
      "vendor_id": "vendor-uuid-777",
      "status": "DRAFT",
      "priority": "HIGH",
      "grand_total": 3540.00,
      "created_at": "2026-06-29T14:30:00Z"
    }
  ]
}
```

---

### 5.3 Get Purchase Order
Retrieves detailed PO information along with item lines.

* **Endpoint:** `GET /diagnostic-orders/lab/procurement/po/{id}`
* **Required Permission:** `diagnostics:order:view`
* **Path Parameters:**
  | Parameter | Type | Required? | Description |
  | :--- | :--- | :--- | :--- |
  | `id` | UUID | **Mandatory** | The Purchase Order identifier |

* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "po-uuid-1111",
    "tenant_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "po_number": "PO-LAB-00001",
    "vendor_id": "vendor-uuid-777",
    "status": "DRAFT",
    "priority": "HIGH",
    "lab_section": "Haematology",
    "requested_by": "user-uuid-coordinator",
    "approved_by": null,
    "estimated_value": 3000.00,
    "gst_amount": 540.00,
    "grand_total": 3540.00,
    "created_at": "2026-06-29T14:30:00Z",
    "items": [
      {
        "id": "po-line-uuid-1",
        "item_id": "df86d618-0977-4579-a061-e75b1d14d4dd",
        "item_name": "Hematology Reagent Diluent",
        "final_qty": 20,
        "received_qty": 0,
        "unit_price": 150.00,
        "gst_percent": 18.0,
        "total": 3540.00
      }
    ]
  }
}
```

---

### 5.4 Update Purchase Order
Updates PO status or modifies details. **Note: A PO in `CANCELLED` or `RECEIVED` state cannot transition status.**

* **Endpoint:** `PUT /diagnostic-orders/lab/procurement/po/{id}`
* **Required Permission:** `diagnostics:test:manage`
* **Path Parameters:**
  | Parameter | Type | Required? | Description |
  | :--- | :--- | :--- | :--- |
  | `id` | UUID | **Mandatory** | The Purchase Order identifier |

* **Request Body (JSON):**
  | Field | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `status` | String | Optional | New PO status | `"APPROVED"`, `"CANCELLED"`, `"RECEIVED"` |
  | `priority` | String | Optional | Urgency level | `"ROUTINE"`, `"MEDIUM"`, `"HIGH"`, `"EMERGENCY"` |
  | `expected_delivery`| Date | Optional | Date (`YYYY-MM-DD`) | |
  | `actual_delivery` | Date | Optional | Date (`YYYY-MM-DD`) | |
  | `notes` | String | Optional | Comments | |

* **Example Request:**
```json
{
  "status": "APPROVED",
  "notes": "Approved for shipment order"
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Purchase Order updated successfully",
  "data": {
    "id": "po-uuid-1111",
    "status": "APPROVED",
    "approved_by": "user-uuid-approver",
    "updated_at": "2026-06-29T15:00:00Z"
  }
}
```

---

### 5.5 Create Goods Received Note (GRN)
Processes the delivery of items against a PO. **Note: This action automatically creates inventory lots/batches, increments item stock, and marks the associated PO status as RECEIVED.**

* **Endpoint:** `POST /diagnostic-orders/lab/procurement/grn`
* **Required Permission:** `diagnostics:test:manage`
* **Request Body (JSON):**
  | Field | Type | Required? | Description | Constraints / Default |
  | :--- | :--- | :--- | :--- | :--- |
  | `po_id` | UUID | **Mandatory** | Associated Purchase Order ID | |
  | `vendor_id` | UUID | Optional | Vendor identifier | |
  | `notes` | String | Optional | Reception remarks | |
  | `items` | Array | **Mandatory** | List of received line items | |

* **LabGRNItemCreate structure:**
  | Field | Type | Required? | Description | Constraints / Default |
  | :--- | :--- | :--- | :--- | :--- |
  | `item_id` | UUID | Optional | Received inventory item ID | |
  | `item_name` | String | **Mandatory** | Product description label | |
  | `expected_qty` | Integer | Optional | Promised count | |
  | `received_qty` | Integer | **Mandatory** | Accepted package lot count | Must be > 0 |
  | `accepted_qty` | Integer | Optional | Approved count | |
  | `rejected_qty` | Integer | Optional | Denied count | Default: `0` |
  | `rejection_reason` | String | Optional | Reason for denial | |
  | `batch_number` | String | Optional | Vendor lot/batch barcode string | Auto-generated if not provided |
  | `lot_number` | String | Optional | Manufacturer lot tag | |
  | `expiry_date` | Date | Optional | Expiry date (`YYYY-MM-DD`) | Default: Today's date |
  | `manufacturing_date`| Date | Optional | Manufacture date (`YYYY-MM-DD`) | |
  | `unit_price` | Float | Optional | Invoice unit price | |
  | `coa_received` | Boolean | Optional | Certificate of Analysis checked | Default: `false` |
  | `qc_status` | String | Optional | Quality check tier status | Default: `"PENDING"` |

* **Example Request:**
```json
{
  "po_id": "po-uuid-1111",
  "items": [
    {
      "item_id": "df86d618-0977-4579-a061-e75b1d14d4dd",
      "item_name": "Hematology Reagent Diluent",
      "received_qty": 20,
      "batch_number": "BATCH-HEM-9988",
      "expiry_date": "2027-12-31",
      "manufacturing_date": "2026-06-01",
      "unit_price": 150.00,
      "coa_received": true
    }
  ]
}
```
* **Example Response (201 Created):**
```json
{
  "success": true,
  "message": "Goods Received Note processed successfully",
  "data": {
    "id": "grn-uuid-2222",
    "grn_number": "GRN-LAB-00001",
    "po_id": "po-uuid-1111",
    "status": "PENDING",
    "received_by": "user-uuid-receiver",
    "received_at": "2026-06-29T16:00:00Z",
    "items": [
      {
        "id": "grn-line-uuid-1",
        "item_id": "df86d618-0977-4579-a061-e75b1d14d4dd",
        "item_name": "Hematology Reagent Diluent",
        "received_qty": 20,
        "batch_number": "BATCH-HEM-9988",
        "expiry_date": "2027-12-31",
        "qc_status": "PENDING"
      }
    ]
  }
}
```

---

### 5.6 List Goods Received Notes
Retrieves all Goods Received Notes logged for the branch.

* **Endpoint:** `GET /diagnostic-orders/lab/procurement/grn`
* **Required Permission:** `diagnostics:order:view`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": "grn-uuid-2222",
      "grn_number": "GRN-LAB-00001",
      "po_id": "po-uuid-1111",
      "status": "PENDING",
      "received_at": "2026-06-29T16:00:00Z"
    }
  ]
}
```

---

### 5.7 Get Goods Received Note
Retrieves comprehensive GRN detail record.

* **Endpoint:** `GET /diagnostic-orders/lab/procurement/grn/{id}`
* **Required Permission:** `diagnostics:order:view`
* **Path Parameters:**
  | Parameter | Type | Required? | Description |
  | :--- | :--- | :--- | :--- |
  | `id` | UUID | **Mandatory** | The GRN identifier |

* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "grn-uuid-2222",
    "tenant_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "grn_number": "GRN-LAB-00001",
    "po_id": "po-uuid-1111",
    "vendor_id": "vendor-uuid-777",
    "status": "PENDING",
    "received_by": "user-uuid-receiver",
    "received_at": "2026-06-29T16:00:00Z",
    "inspected_at": null,
    "notes": null,
    "items": [
      {
        "id": "grn-line-uuid-1",
        "item_id": "df86d618-0977-4579-a061-e75b1d14d4dd",
        "item_name": "Hematology Reagent Diluent",
        "received_qty": 20,
        "accepted_qty": 20,
        "rejected_qty": 0,
        "batch_number": "BATCH-HEM-9988",
        "expiry_date": "2027-12-31",
        "qc_status": "PENDING"
      }
    ]
  }
}
```

---

### 5.8 Update Goods Received Note
Updates GRN status (e.g. from PENDING to ACCEPTED / REJECTED following quality control validation).

* **Endpoint:** `PUT /diagnostic-orders/lab/procurement/grn/{id}`
* **Required Permission:** `diagnostics:test:manage`
* **Path Parameters:**
  | Parameter | Type | Required? | Description |
  | :--- | :--- | :--- | :--- |
  | `id` | UUID | **Mandatory** | The GRN identifier |

* **Request Body (JSON):**
  | Field | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `status` | String | Optional | Updated status | e.g. `"ACCEPTED"`, `"REJECTED"` |
  | `inspected_by` | UUID | Optional | Inspector user UUID | |
  | `notes` | String | Optional | Comments | |

* **Example Request:**
```json
{
  "status": "ACCEPTED",
  "notes": "Verified batch details against packing sheet"
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Goods Received Note updated successfully",
  "data": {
    "id": "grn-uuid-2222",
    "status": "ACCEPTED",
    "inspected_at": "2026-06-29T16:30:00Z",
    "notes": "Verified batch details against packing sheet"
  }
}
```

---

### 5.9 Create Vendor Invoice
Registers vendor billing invoice details to clear supplier payment.

* **Endpoint:** `POST /diagnostic-orders/lab/procurement/invoice`
* **Required Permission:** `diagnostics:test:manage`
* **Request Body (JSON):**
  | Field | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `invoice_number` | String | **Mandatory** | Unique vendor invoice reference code | |
  | `invoice_amount` | Float | **Mandatory** | Subtotal value | Must be >= 0.0 |
  | `total_amount` | Float | **Mandatory** | Grand total value | Must be >= 0.0 |
  | `vendor_id` | UUID | Optional | Vendor identifier | |
  | `po_id` | UUID | Optional | Associated Purchase Order ID | |
  | `grn_id` | UUID | Optional | Associated Goods Received Note ID | |
  | `invoice_date` | Date | Optional | Statement date (`YYYY-MM-DD`) | |
  | `gst_amount` | Float | Optional | Total tax component amount | Default: `0.0` |

* **Example Request:**
```json
{
  "invoice_number": "INV-HEM-5544",
  "po_id": "po-uuid-1111",
  "grn_id": "grn-uuid-2222",
  "invoice_amount": 3000.00,
  "gst_amount": 540.00,
  "total_amount": 3540.00,
  "invoice_date": "2026-06-29"
}
```
* **Example Response (201 Created):**
```json
{
  "success": true,
  "message": "Vendor Invoice created successfully",
  "data": {
    "id": "invoice-uuid-3333",
    "invoice_number": "INV-HEM-5544",
    "po_id": "po-uuid-1111",
    "grn_id": "grn-uuid-2222",
    "invoice_amount": 3000.00,
    "total_amount": 3540.00,
    "match_status": "MATCHED",
    "payment_status": "UNPAID",
    "created_at": "2026-06-29T17:00:00Z"
  }
}
```

---

### 5.10 List Vendor Invoices
Retrieves registered supplier invoices.

* **Endpoint:** `GET /diagnostic-orders/lab/procurement/invoice`
* **Required Permission:** `diagnostics:order:view`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": "invoice-uuid-3333",
      "invoice_number": "INV-HEM-5544",
      "total_amount": 3540.00,
      "match_status": "MATCHED",
      "payment_status": "UNPAID",
      "created_at": "2026-06-29T17:00:00Z"
    }
  ]
}
```

---

### 5.11 Get Vendor Invoice
Retrieves detailed vendor invoice info.

* **Endpoint:** `GET /diagnostic-orders/lab/procurement/invoice/{id}`
* **Required Permission:** `diagnostics:order:view`
* **Path Parameters:**
  | Parameter | Type | Required? | Description |
  | :--- | :--- | :--- | :--- |
  | `id` | UUID | **Mandatory** | The Invoice identifier |

* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "invoice-uuid-3333",
    "tenant_id": "46fc39d8-7c4e-4704-9430-f82d6dcfa34c",
    "invoice_number": "INV-HEM-5544",
    "vendor_id": "vendor-uuid-777",
    "po_id": "po-uuid-1111",
    "grn_id": "grn-uuid-2222",
    "invoice_amount": 3000.00,
    "total_amount": 3540.00,
    "match_status": "MATCHED",
    "payment_status": "UNPAID",
    "created_at": "2026-06-29T17:00:00Z"
  }
}
```

---

### 5.12 Update Vendor Invoice
Modifies billing status or matches invoice values.

* **Endpoint:** `PUT /diagnostic-orders/lab/procurement/invoice/{id}`
* **Required Permission:** `diagnostics:test:manage`
* **Path Parameters:**
  | Parameter | Type | Required? | Description |
  | :--- | :--- | :--- | :--- |
  | `id` | UUID | **Mandatory** | The Invoice identifier |

* **Request Body (JSON):**
  | Field | Type | Required? | Description | Constraints |
  | :--- | :--- | :--- | :--- | :--- |
  | `match_status` | String | Optional | Match audit status | `"MATCHED"`, `"MISMATCHED"` |
  | `mismatch_reason`| String | Optional | Details of mismatch | |
  | `payment_status` | String | Optional | Invoice payout status | `"UNPAID"`, `"PAID"` |

* **Example Request:**
```json
{
  "payment_status": "PAID"
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Vendor Invoice updated successfully",
  "data": {
    "id": "invoice-uuid-3333",
    "payment_status": "PAID",
    "updated_at": "2026-06-29T17:30:00Z"
  }
}
```

---

## 6. Extended Dashboards & Pathologist Verification Workflow APIs

### 6.1 Sample Management Statistics
Retrieves statistics for the Sample Management dashboard.

* **Endpoint:** `GET /diagnostic-orders/lab/sample-management/statistics`
* **Required Permission:** `diagnostics:order:view`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "completion_rate": 88.5,
    "rejection_rate": 1.2,
    "in_transit": 4,
    "awaiting_processing": 12,
    "on_analyzer": 5,
    "avg_tat_minutes": 24.5
  }
}
```

---

### 6.2 List Dashboard Samples
Lists all samples currently in phlebotomy, transit, or lab receiving.

* **Endpoint:** `GET /diagnostic-orders/lab/samples`
* **Required Permission:** `diagnostics:order:view`
* **Query Parameters:** `status`, `priority`, `search`, `page`, `limit`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "total": 1,
    "page": 1,
    "limit": 10,
    "items": [
      {
        "sample_id": "df86d618-0977-4579-a061-e75b1d14d4dd",
        "barcode": "BAR-10029",
        "patient_name": "Karan Sharma",
        "patient_uhid": "UHID-2026-98721",
        "test_name": "Complete Blood Count",
        "machine_name": "Sysmex XN-1000",
        "technician_name": "Amit Sharma",
        "status": "IN_TRANSIT",
        "priority": "Routine",
        "created_at": "2026-06-23T10:40:00Z",
        "tat_minutes": 15.2
      }
    ]
  }
}
```

---

### 6.3 Confirm Sample Arrival
Technician registers container arrival at receiving bay, transitioning status from `IN_TRANSIT` to `AWAITING_PROCESSING`.

* **Endpoint:** `PATCH /diagnostic-orders/lab/samples/{id}/confirm-arrival`
* **Required Permission:** `diagnostics:lab:collect`
* **Path Parameters:**
  | Parameter | Type | Required? | Description |
  | :--- | :--- | :--- | :--- |
  | `id` | UUID | **Mandatory** | The lab order item identifier |
* **Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Sample arrival confirmed successfully",
  "data": {
    "id": "df86d618-0977-4579-a061-e75b1d14d4dd",
    "status": "SCHEDULED",
    "updated_at": "2026-07-13T11:00:00Z"
  }
}
```

---

### 6.4 Test Processing Statistics
Retrieves statistics for the Test Processing dashboard.

* **Endpoint:** `GET /diagnostic-orders/lab/test-processing/statistics`
* **Required Permission:** `diagnostics:order:view`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "assigned_bays": 3,
    "hl7_pending": 8,
    "manual_pending": 4,
    "total_tests": 12
  }
}
```

---

### 6.5 List Bench Assignments
Lists all laboratory workspaces, running instruments, and workloads.

* **Endpoint:** `GET /diagnostic-orders/lab/bench-assignments`
* **Required Permission:** `diagnostics:order:view`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "bench_name": "Hematology Bench A",
      "machine_name": "Sysmex XN-1000",
      "technician_name": "Amit Sharma",
      "active_test_count": 5
    }
  ]
}
```

---

### 6.6 Save Result Draft
Autosaves temporary parameter entries.

* **Endpoint:** `POST /diagnostic-orders/lab/results/{orderId}/draft`
* **Required Permission:** `diagnostics:lab:collect`
* **Path Parameters:**
  | Parameter | Type | Required? | Description |
  | :--- | :--- | :--- | :--- |
  | `orderId` | UUID | **Mandatory** | The lab order item identifier |
* **Request Body:**
```json
{
  "draft_values": {
    "wbc_param_id": "14.2",
    "hgb_param_id": "11.2"
  },
  "remarks": "Slight turbidity noted"
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Draft saved successfully"
}
```

---

### 6.7 Mark Critical Value & Pathologist Alert
Flags high-risk parameters and sends alerts to pathologists.

* **Endpoint:** `PATCH /diagnostic-orders/lab/results/{orderId}/critical`
* **Required Permission:** `diagnostics:lab:collect`
* **Request Body:**
```json
{
  "parameter_ids": ["wbc_param_id"],
  "reason": "WBC count highly elevated",
  "pathologist_id": "pathologist-uuid-1111"
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Results marked as critical and pathologist notified"
}
```

---

### 6.8 Acknowledge Critical Alert
Enables clinicians or technicians to sign-off critical alarms.

* **Endpoint:** `POST /diagnostic-orders/lab/result-submission/critical-logs/{id}/acknowledge`
* **Required Permission:** `diagnostics:lab:collect`
* **Request Body:**
```json
{
  "notes": "Notified Dr. Reyes via phone call at 08:32"
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Critical result log acknowledged successfully"
}
```

---

### 6.9 Result Submission Dashboard Statistics
Retrieves statistics for the Result Submission screen.

* **Endpoint:** `GET /diagnostic-orders/lab/result-submission/statistics`
* **Required Permission:** `diagnostics:order:view`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "ready_to_send": 4,
    "sent_to_pathologist": 1,
    "unacknowledged_criticals": 3,
    "criticals_triggered": 5
  }
}
```

---

### 6.10 Reports & Delivery Extended Statistics
Compiles verification KPIs for the pathologist.

* **Endpoint:** `GET /diagnostic-orders/lab/results/statistics`
* **Required Permission:** `diagnostics:order:view`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "critical_values": 3,
    "delta_flagged": 5,
    "abnormal_results": 3,
    "within_range": 2,
    "approved_today": 142,
    "generated_today": 186,
    "delivered_today": 180
  }
}
```

---

### 6.11 Update Result Status (Approve, Reprocess, Sign, Deliver)
Pathologist validation and report lifecycle transitions.

* **Endpoint:** `PATCH /diagnostic-orders/lab/results/{id}/status`
* **Required Permission:** `diagnostics:lab:result:validate`
* **Path Parameters:**
  | Parameter | Type | Required? | Description |
  | :--- | :--- | :--- | :--- |
  | `id` | UUID | **Mandatory** | The result identifier |

* **Request Examples:**
  * **Approve & Digitally Sign:**
    ```json
    {
      "status": "APPROVED",
      "remarks": "Verified",
      "digitalSignature": true
    }
    ```
  * **Reprocess / Re-run:**
    ```json
    {
      "status": "REPROCESSING",
      "remarks": "Analyzer calibration failed"
    }
    ```
  * **Deliver Report:**
    ```json
    {
      "status": "REPORT_DELIVERED"
    }
    ```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Result status updated successfully",
  "data": {
    "id": "result-uuid-1111",
    "status": "APPROVED",
    "digital_signature": true,
    "validated_by": "pathologist-uuid-1111"
  }
}
```

---

## 7. Clinical Addendum Resource APIs

Clinical addenda management for post-release updates.

### 7.1 Create Addendum
* **Endpoint:** `POST /diagnostic-orders/lab/addendums`
* **Required Permission:** `diagnostics:lab:result:validate`
* **Request Body:**
```json
{
  "lab_result_id": "result-uuid-1111",
  "patient_name": "Rajan Kumar",
  "patient_uhid": "UHID-2026-091245",
  "test_type": "Lipid Profile",
  "amendment_type": "Clinical Interpretation Update",
  "requested_by": "Dr. Priya Sharma",
  "updated_by": "Dr. Rajesh Mehta",
  "reason": "Incorporate post-release clinical notes",
  "clinical_notes": "Slight hyperlipidemia observed, follow up in 2 months",
  "status": "Pending"
}
```
* **Success Response (201 Created):**
```json
{
  "success": true,
  "message": "Addendum report created successfully",
  "data": {
    "id": "addendum-uuid-5555",
    "status": "Pending",
    "created_at": "2026-07-13T11:25:00Z"
  }
}
```

---

### 7.2 List Addendums
* **Endpoint:** `GET /diagnostic-orders/lab/addendums`
* **Required Permission:** `diagnostics:order:view`
* **Query Parameters:** `status`, `reportId`, `patient`, `doctor`, `page`, `limit`, `search`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "total": 1,
    "page": 1,
    "limit": 10,
    "items": [
      {
        "addendum_id": "addendum-uuid-5555",
        "lab_result_id": "result-uuid-1111",
  "success": true,
  "message": "Draft saved successfully"
}
```

---

### 6.7 Mark Critical Value & Pathologist Alert
Flags high-risk parameters and sends alerts to pathologists.

* **Endpoint:** `PATCH /diagnostic-orders/lab/results/{orderId}/critical`
* **Required Permission:** `diagnostics:lab:collect`
* **Request Body:**
```json
{
  "parameter_ids": ["wbc_param_id"],
  "reason": "WBC count highly elevated",
  "pathologist_id": "pathologist-uuid-1111"
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Results marked as critical and pathologist notified"
}
```

---

### 6.8 Acknowledge Critical Alert
Enables clinicians or technicians to sign-off critical alarms.

* **Endpoint:** `POST /diagnostic-orders/lab/result-submission/critical-logs/{id}/acknowledge`
* **Required Permission:** `diagnostics:lab:collect`
* **Request Body:**
```json
{
  "notes": "Notified Dr. Reyes via phone call at 08:32"
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Critical result log acknowledged successfully"
}
```

---

### 6.9 Result Submission Dashboard Statistics
Retrieves statistics for the Result Submission screen.

* **Endpoint:** `GET /diagnostic-orders/lab/result-submission/statistics`
* **Required Permission:** `diagnostics:order:view`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "ready_to_send": 4,
    "sent_to_pathologist": 1,
    "unacknowledged_criticals": 3,
    "criticals_triggered": 5
  }
}
```

---

### 6.10 Reports & Delivery Extended Statistics
Compiles verification KPIs for the pathologist.

* **Endpoint:** `GET /diagnostic-orders/lab/results/statistics`
* **Required Permission:** `diagnostics:order:view`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "critical_values": 3,
    "delta_flagged": 5,
    "abnormal_results": 3,
    "within_range": 2,
    "approved_today": 142,
    "generated_today": 186,
    "delivered_today": 180
  }
}
```

---

### 6.11 Update Result Status (Approve, Reprocess, Sign, Deliver)
Pathologist validation and report lifecycle transitions.

* **Endpoint:** `PATCH /diagnostic-orders/lab/results/{id}/status`
* **Required Permission:** `diagnostics:lab:result:validate`
* **Path Parameters:**
  | Parameter | Type | Required? | Description |
  | :--- | :--- | :--- | :--- |
  | `id` | UUID | **Mandatory** | The result identifier |

* **Request Examples:**
  * **Approve & Digitally Sign:**
    ```json
    {
      "status": "APPROVED",
      "remarks": "Verified",
      "digitalSignature": true
    }
    ```
  * **Reprocess / Re-run:**
    ```json
    {
      "status": "REPROCESSING",
      "remarks": "Analyzer calibration failed"
    }
    ```
  * **Deliver Report:**
    ```json
    {
      "status": "REPORT_DELIVERED"
    }
    ```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Result status updated successfully",
  "data": {
    "id": "result-uuid-1111",
    "status": "APPROVED",
    "digital_signature": true,
    "validated_by": "pathologist-uuid-1111"
  }
}
```

---

## 7. Clinical Addendum Resource APIs

Clinical addenda management for post-release updates.

### 7.1 Create Addendum
* **Endpoint:** `POST /diagnostic-orders/lab/addendums`
* **Required Permission:** `diagnostics:lab:result:validate`
* **Request Body:**
```json
{
  "lab_result_id": "result-uuid-1111",
  "patient_name": "Rajan Kumar",
  "patient_uhid": "UHID-2026-091245",
  "test_type": "Lipid Profile",
  "amendment_type": "Clinical Interpretation Update",
  "requested_by": "Dr. Priya Sharma",
  "updated_by": "Dr. Rajesh Mehta",
  "reason": "Incorporate post-release clinical notes",
  "clinical_notes": "Slight hyperlipidemia observed, follow up in 2 months",
  "status": "Pending"
}
```
* **Success Response (201 Created):**
```json
{
  "success": true,
  "message": "Addendum report created successfully",
  "data": {
    "id": "addendum-uuid-5555",
    "status": "Pending",
    "created_at": "2026-07-13T11:25:00Z"
  }
}
```

---

### 7.2 List Addendums
* **Endpoint:** `GET /diagnostic-orders/lab/addendums`
* **Required Permission:** `diagnostics:order:view`
* **Query Parameters:** `status`, `reportId`, `patient`, `doctor`, `page`, `limit`, `search`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "total": 1,
    "page": 1,
    "limit": 10,
    "items": [
      {
        "addendum_id": "addendum-uuid-5555",
        "lab_result_id": "result-uuid-1111",
        "patient_name": "Rajan Kumar",
        "amendment_type": "Clinical Interpretation Update",
        "status": "Pending"
      }
    ]
  }
}
```

---

### 7.3 View Addendum Detail
* **Endpoint:** `GET /diagnostic-orders/lab/addendums/{id}`
* **Required Permission:** `diagnostics:order:view`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "addendum-uuid-5555",
    "lab_result_id": "result-uuid-1111",
    "patient_name": "Rajan Kumar",
    "patient_uhid": "UHID-2026-091245",
    "test_type": "Lipid Profile",
    "amendment_type": "Clinical Interpretation Update",
    "requested_by": "Dr. Priya Sharma",
    "updated_by": "Dr. Rajesh Mehta",
    "reason": "Incorporate post-release clinical notes",
    "clinical_notes": "Slight hyperlipidemia observed, follow up in 2 months",
    "status": "Pending",
    "created_at": "2026-07-13T11:25:00Z"
  }
}
```

---

### 7.4 Update Addendum
Updates the status or clinical notes of a released addendum.

* **Endpoint:** `PATCH /diagnostic-orders/lab/addendums/{id}`
* **Required Permission:** `diagnostics:lab:result:validate`
* **Request Body:**
```json
{
  "status": "Approved",
  "clinical_notes": "Reviewed and signed off clinical interpretation notes."
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Addendum report updated successfully",
  "data": {
    "id": "addendum-uuid-5555",
    "status": "Approved",
    "updated_at": "2026-07-13T11:30:00Z"
  }
}
```

---

## 8. Dashboard Feeds & Workflow Pipelines

### 8.1 List HL7 Imported Results
Lists imported analyser findings from integrated laboratory instruments.

* **Endpoint:** `GET /diagnostic-orders/lab/results/hl7`
* **Required Permission:** `diagnostics:order:view`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": "hl7-result-uuid",
      "machine_name": "Sysmex XN-1000",
      "test_name": "Hemoglobin (HGB)",
      "result_value": "11.2",
      "unit": "g/dL",
      "flag": "Low",
      "timestamp": "2026-07-13T08:00:00Z"
    }
  ]
}
```

---

### 8.2 List Manual Entry Queue
Lists test orders awaiting manually typed parameter entries.

* **Endpoint:** `GET /diagnostic-orders/lab/results/manual-entry`
* **Required Permission:** `diagnostics:order:view`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "order_item_id": "item-uuid-1111",
      "patient_name": "Maria Santos",
      "test_name": "Complete Blood Count",
      "status": "AWAITING_PROCESSING"
    }
  ]
}
```

---

### 8.3 Get Result Entry Details
Loads dynamic parameters, units, saved drafts, and patient history for Result Entry.

* **Endpoint:** `GET /diagnostic-orders/lab/results/{orderId}/details`
* **Required Permission:** `diagnostics:order:view`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "patient": {
      "name": "Karan Sharma",
      "uhid": "UHID-2026-98721",
      "age": 42,
      "gender": "Male"
    },
    "doctor": "Dr. Ramesh Nair",
    "department": "General Medicine",
    "order_id": "order-uuid-2222",
    "items": [
      {
        "item_id": "item-uuid-1111",
        "test_name": "Complete Blood Count",
        "parameters": [
          {
            "id": "param-uuid-1111",
            "parameter_name": "Hemoglobin",
            "reference_range": "13.0 - 17.0",
            "unit": "g/dL"
          }
        ],
        "draft": {
          "values": {
            "param-uuid-1111": "11.2"
          },
          "remarks": "Slight low"
        }
      }
    ],
    "previous_results": []
  }
}
```

---

### 8.4 Submit Result Values
Submits completed parameters, validating that all mandatory fields are present.

* **Endpoint:** `POST /diagnostic-orders/lab/results/{orderId}/submit`
* **Required Permission:** `diagnostics:lab:collect`
* **Request Body:**
```json
{
  "result_values": {
    "param-uuid-1111": "11.2"
  },
  "remarks": "Results typing complete"
}
```
* **Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Results submitted successfully"
}
```

---

### 8.5 List Pathologist Queue (Result Review)
Lists submitted result records in the Pathologist Verification workflow pipeline.

* **Endpoint:** `GET /diagnostic-orders/lab/result-submission/pathologist-queue`
* **Required Permission:** `diagnostics:order:view`
* **Query Parameters:** `status` (Ready, Sent, Held), `search`, `page`, `limit`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "total": 1,
    "items": [
      {
        "order_item_id": "item-uuid-1111",
        "patient_name": "Maria Santos",
        "test_name": "Complete Blood Count",
        "doctor_name": "Dr. Reyes",
        "status": "Ready to Send"
      }
    ]
  }
}
```

---

### 8.6 List Critical Flags Queue
Lists critical alarms requiring clinical sign-off.

* **Endpoint:** `GET /diagnostic-orders/lab/result-submission/critical-logs`
* **Required Permission:** `diagnostics:order:view`
* **Query Parameters:** `status` (unacknowledged, acknowledged)
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "log_id": "log-uuid-1111",
      "patient_name": "Maria Santos",
      "priority": "Emergency",
      "doctor_name": "Dr. Reyes",
      "reason": "WBC count highly elevated",
      "parameter_details": "WBC: 12.4 10^3/uL",
      "timestamp": "2026-07-13T08:31:00Z",
      "acknowledged_at": null
    }
  ]
}
```

---

### 8.7 Send Result to Pathologist Queue
Promotes autosaved drafts into final review status (`READY_FOR_REVIEW`).

* **Endpoint:** `POST /diagnostic-orders/lab/result-submission/results/{id}/send`
* **Required Permission:** `diagnostics:lab:collect`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Result sent to pathologist successfully"
}
```

---

### 8.8 Get Detailed Result Details (Pathologist Report Review)
Loads detailed context for laboratory investigation reports.

* **Endpoint:** `GET /diagnostic-orders/lab/results/{id}`
* **Required Permission:** `diagnostics:order:view`
* **Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "patient": {
      "name": "Rajan Kumar",
      "uhid": "UHID-2026-091245",
      "age": 42,
      "gender": "Male",
      "phone": "+919999999999"
    },
    "doctor": "Dr. Priya Sharma",
    "department": "General Medicine",
    "barcode": "LX-2026-43482",
    "result_id": "result-uuid-1111",
    "status": "APPROVED",
    "digital_signature": true,
    "parameters": [
      {
        "parameter_name": "Hemoglobin",
        "actual_value": "11.2",
        "unit": "g/dL",
        "reference_range": "13.0 - 17.0"
      }
    ],
    "previous_reports": [],
    "status_history": []
  }
}
```

---

### 8.9 Download Signed Investigation Report PDF
Streams dynamic report PDF document downloads.

* **Endpoint:** `GET /diagnostic-orders/lab/results/{id}/pdf`
* **Required Permission:** `diagnostics:order:view`
* **Response:**
  * **Content-Type:** `application/pdf`
  * **Content-Disposition:** `attachment; filename=lab_report_{id}.pdf`

---

### 8.10 Export Filtered Result Rows
Streams worklist spreadsheet downloads matching current filter criteria.

* **Endpoint:** `GET /diagnostic-orders/lab/results/export`
* **Required Permission:** `diagnostics:order:view`
* **Query Parameters:** `status`, `search`, `format` (default: `csv`)
* **Response:**
  * **Content-Type:** `text/csv`
  * **Content-Disposition:** `attachment; filename=lab_samples_export.csv`
