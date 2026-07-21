# Laboratory Module Sequence Workflows
Version: 1.0.0
Author: Systems Architect

---

## 1. Laboratory Order Registration Flow
This diagram illustrates order registration completed by the registration clerk.

```mermaid
sequenceDiagram
    autonumber
    actor Staff as Registration Staff
    participant GW as API Gateway (main.py)
    participant C as Controller (registration.py)
    participant S as Service (registration_service.py)
    participant DB as Postgres Database

    Staff ->> GW: POST /lab/registrations (JWT, payload)
    GW ->> GW: Validate Cognito JWT Bearer Scopes
    GW ->> C: dispatch("create_registration", req)
    C ->> C: Pydantic payload verification
    C ->> S: create_registration(tenant_id, body)
    S ->> DB: Insert into diagnostic.diagnostic_orders
    S ->> DB: Insert into diagnostic.order_items
    S ->> DB: Insert into revenue.bills
    DB -->> S: Return UUID identifiers
    S -->> C: Registration result dict
    C -->> Staff: HTTP 201 Success Response (Created)
```

---

## 2. Bio-Sample Collection and Tracking Flow
Illustrates sample scanning at the receiving bench.

```mermaid
sequenceDiagram
    autonumber
    actor Tech as Laboratory Technician
    participant GW as API Gateway
    participant C as Controller (technician.py)
    participant S as Service (lab_inventory_service.py)
    participant DB as Postgres Database

    Tech ->> GW: PATCH /lab/samples/{id}/confirm-arrival (barcode)
    GW ->> C: dispatch("confirm_sample_arrival", req)
    C ->> S: confirm_sample_arrival(sample_id, barcode)
    S ->> DB: Update diagnostic.samples status = 'RECEIVED'
    DB -->> S: Return updated sample data
    S -->> C: Confirmation payload
    C -->> Tech: HTTP 200 OK (Sample Received)
```

---

## 3. Results Verification & Sign-off Flow
Enforces Pathologist verification and PDF report building.

```mermaid
sequenceDiagram
    autonumber
    actor Path as Pathologist
    participant GW as API Gateway
    participant C as Controller (pathologist.py)
    participant S as Service (pathologist_service.py)
    participant DB as Postgres Database

    Path ->> GW: PATCH /lab/results/{id}/status (VALIDATED)
    GW ->> C: dispatch("update_result_status", req)
    C ->> S: validate_result(result_id, status, digital_signature)
    S ->> DB: Update diagnostic.test_results status = 'VALIDATED'
    S ->> DB: Update diagnostic.order_items status = 'COMPLETED'
    DB -->> S: Status Updated
    S -->> C: Sign-off payload
    C -->> Path: HTTP 200 OK (Report Validated)
```
