# HMS Diagnostics & Lab Dashboard — API Reference

This document details the REST API endpoints in the `services/lab` module designed to serve the real-time patient queue, metrics counters, inventory tracking, and alert actions displayed on the Diagnostics Reception & Technician Dashboard.

---

## 1. Grouped Lab Queue

### GET `/diagnostics/lab/queue/grouped`
* **Description**: Returns a paginated list of lab orders grouped at the patient/order level (instead of individual items). This allows the frontend to show a cohesive queue item with all tests ordered for a patient.
* **Required Permission**: `diagnostics:order:view`
* **Authorized Roles**: Lab Technician (`LAB-001`), Lab Admin (`LAB-002`), Doctor (`MED-001`), Hospital Admin (`ADM-001`)
* **Query Parameters**:
  * `status` (optional): Filter by computed order status (`Collecting`, `Processing`, `Awaiting verify`, `Verified`, `Delivered`).
  * `page` (optional): Page number (default: `1`).
  * `page_size` (optional): Number of records per page (default: `50`).
* **Request Headers**:
  ```http
  Authorization: Bearer <access_token>
  ```
* **Response Body (`200 OK`)**:
  ```json
  {
    "success": true,
    "status": 200,
    "data": {
      "items": [
        {
          "token": "LAB-2418",
          "patient_name": "Karan Sharma",
          "patient_mrn": "UH-83821",
          "tests": ["Lipid Profile", "HbA1c"],
          "department_name": "Cardiology",
          "status": "Collecting",
          "order_id": "c8542c47-f3e5-4ec0-8f24-b9f7e58c32f2",
          "created_at": "2026-06-03T10:40:00Z"
        },
        {
          "token": "LAB-2417",
          "patient_name": "Sarita Singh",
          "patient_mrn": "UH-48320",
          "tests": ["CBC", "ESR"],
          "department_name": "General mucosa",
          "status": "Processing",
          "order_id": "e40badbc-c78d-4184-a438-a717f7428be8",
          "created_at": "2026-06-03T10:35:00Z"
        }
      ],
      "total": 2,
      "page": 1,
      "page_size": 50
    }
  }
  ```

---

## 2. Dashboard Metrics Today

### GET `/diagnostics/dashboard/stats`
* **Description**: Gathers all dashboard widgets' metrics for today, including hourly sparkline charts points.
* **Required Permission**: `diagnostics:dashboard`
* **Authorized Roles**: Lab Admin (`LAB-002`), Hospital Admin (`ADM-001`), Doctor (`MED-001`)
* **Response Body (`200 OK`)**:
  ```json
  {
    "success": true,
    "status": 200,
    "data": {
      "walkins_today": 142,
      "walkins_change_pct": 18.0,
      "pending_samples": 37,
      "pending_priority_count": 4,
      "tat_breaches": 4,
      "revenue_today": 84200.0,
      "revenue_change_pct": 12.4,
      "reports_ready": 58,
      "reports_awaiting_count": 12,
      "avg_tat_hours": 2.4,
      "avg_tat_change_mins": -18,
      "sparklines": {
        "walkins": [120, 125, 130, 128, 135, 142],
        "pending_samples": [42, 45, 39, 41, 35, 37],
        "tat_breaches": [2, 3, 5, 4, 3, 4],
        "revenue": [72000, 75000, 78000, 81000, 83000, 84200],
        "reports_ready": [45, 48, 52, 50, 55, 58],
        "avg_tat": [2.6, 2.5, 2.7, 2.5, 2.4, 2.4]
      }
    }
  }
  ```

---

## 3. Patient Critical & Emergency Alerts

### GET `/diagnostics/alerts/critical`
* **Description**: Returns all unacknowledged patient critical alert records (e.g. from laboratory tests with `critical_flag = TRUE`).
* **Required Permission**: `diagnostics:order:view`
* **Response Body (`200 OK`)**:
  ```json
  {
    "success": true,
    "status": 200,
    "data": {
      "items": [
        {
          "id": "c0b4356e-8215-44eb-813f-f3d1c33caaaa",
          "token": "TKN-0142",
          "details": "Critical potassium (6.8 mmol/L)",
          "duration_label": "Just now",
          "patient_id": "d1d96f70-b397-457c-8954-c8728e03ca0c",
          "patient_name": "Karan Sharma"
        }
      ]
    }
  }
  ```

### POST `/diagnostics/alerts/{alert_id}/dispatch`
* **Description**: Dispatches / acknowledges a critical emergency alert, logging the receptionist/technician audit action and dismissing the notification.
* **Required Permission**: `diagnostics:lab:result:validate`
* **Response Body (`200 OK`)**:
  ```json
  {
    "success": true,
    "status": 200,
    "message": "Critical alert c0b4356e-8215-44eb-813f-f3d1c33caaaa dispatched successfully"
  }
  ```

---

## 4. Inventory Alerts

### GET `/diagnostics/inventory/alerts`
* **Description**: Checks for low stock alerts on lab reagents, tubes, and disposable items. Calculates an overall stock health percentage indicator.
* **Required Permission**: `diagnostics:dashboard`
* **Response Body (`200 OK`)**:
  ```json
  {
    "success": true,
    "status": 200,
    "data": {
      "stock_health_percentage": 75,
      "items": [
        {
          "item_name": "Cobas Lipid Panel kit",
          "status": "CRITICAL",
          "current_stock": 12,
          "min_stock": 50,
          "unit": "kits"
        },
        {
          "item_name": "EDTA tubes 3mL",
          "status": "WARNING",
          "current_stock": 120,
          "min_stock": 1000,
          "unit": "tubes"
        }
      ]
    }
  }
  ```

---

## 5. Patient Analysis

### GET `/diagnostics/analysis/patient-summary`
* **Description**: Gathers monthly aggregated counts comparing hospital (in-house IPD/OPD) vs external (walk-in/referred) patients for the dynamic bar chart visualization.
* **Required Permission**: `diagnostics:dashboard`
* **Response Body (`200 OK`)**:
  ```json
  {
    "success": true,
    "status": 200,
    "data": {
      "items": [
        {
          "month": "Feb",
          "hospital_count": 650,
          "external_count": 180
        },
        {
          "month": "Mar",
          "hospital_count": 820,
          "external_count": 320
        }
      ]
    }
  }
  ```
