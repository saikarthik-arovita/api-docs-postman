# Finance, Billing & Insurance Claims — Recent Revenue Summary API Specification

## 1. Overview
The **Finance, Billing & Insurance Claims** dashboard provides high-level financial KPIs (Today's Revenue, Monthly Revenue & Target Completion, Highest Revenue Department, Top Earning Service), along with a filterable Recent Revenue Summary Table (with sub-service breakdowns and monthly revenue trend sparkline points for department drawer modals).

---

## 2. API Endpoint

### `GET /admin/finance/revenue-summary`
* **Aliases**:
  - `GET /admin/billing/revenue-summary`
  - `GET /admin/revenue/summary`
  - `GET /admin/finance/revenue-overview`
* **Authentication**: `Bearer <token>` (Requires `SYSTEM_ADMINISTRATOR`, `HOSPITAL_ADMIN`, or Finance/Billing role)
* **Query Parameters**:
  - `department` (String, optional): Filter by department name (e.g., `Radiology`, `IPD`, `OPD`, `Emergency`, `Pharmacy`, `Laboratory`, `All`).
  - `service` (String, optional): Filter by service name (e.g., `MRI Scan`, `Inpatient Admission`, `Consultation`, `Trauma Care`, `Prescription`, `Blood Panel`).
  - `min_revenue` (Float, optional): Minimum revenue filter (e.g., `10000`).
  - `max_revenue` (Float, optional): Maximum revenue filter (e.g., `100000`).
  - `from_date` (String, optional): Start date range filter (`YYYY-MM-DD` or `DD/MM/YYYY`).
  - `to_date` (String, optional): End date range filter (`YYYY-MM-DD` or `DD/MM/YYYY`).
  - `branch_id` / `location` (UUID/String, optional): Branch filter.
  - `search` (String, optional): Search across department and service names.
  - `page` (Integer, default `1`): Current page number.
  - `page_size` (Integer, default `10`): Items per page.

---

## 3. Response Structure (`200 OK`)

```json
{
  "success": true,
  "code": 200,
  "data": {
    "summary": {
      "todays_revenue": 425000.0,
      "formatted_todays_revenue": "₹4,25,000",
      "todays_revenue_trend": "+5.1% vs yesterday",
      "monthly_revenue": 84500.0,
      "formatted_monthly_revenue": "₹84,500",
      "monthly_target": 1000000.0,
      "formatted_monthly_target": "Target: ₹10,00,000",
      "monthly_target_completion_pct": 8.4,
      "highest_revenue_department": "Radiology",
      "highest_revenue_dept_amount": 1240000.0,
      "formatted_highest_revenue_dept_amount": "₹12,40,000",
      "highest_revenue_dept_share_pct": 34.2,
      "highest_revenue_dept_subtitle": "₹12,40,000 (34.2% of total)",
      "top_earning_service": "MRI Scan",
      "top_earning_service_revenue": 420000.0,
      "formatted_top_earning_service_revenue": "₹4,20,000",
      "top_earning_service_subtitle": "MRI Scan Revenue: ₹4,20,000"
    },
    "recent_revenue_summary_table": {
      "items": [
        {
          "id": "rev-rad-mri",
          "department_name": "Radiology",
          "service_name": "MRI Scan",
          "revenue": 48200.0,
          "formatted_revenue": "₹48,200",
          "transactions": 124,
          "avg_bill_value": 3888.7,
          "formatted_avg_bill_value": "₹3,888.70",
          "growth_pct": 18.2,
          "formatted_growth_pct": "+18.2%",
          "department_details": {
            "department_name": "Radiology Department",
            "total_revenue": 48200.0,
            "formatted_revenue": "₹48,200",
            "transactions": 124,
            "avg_bill_value": 3888.7,
            "formatted_avg_bill_value": "₹3,888.70",
            "growth_pct": 8.2,
            "formatted_growth_pct": "+8.2%",
            "service_breakdown": [
              {
                "service_name": "MRI Scan",
                "amount": 24000.0,
                "formatted_amount": "₹24,000",
                "count": 50
              },
              {
                "service_name": "CT Scan",
                "amount": 14200.0,
                "formatted_amount": "₹14,200",
                "count": 32
              },
              {
                "service_name": "X-Ray",
                "amount": 6000.0,
                "formatted_amount": "₹6,000",
                "count": 24
              },
              {
                "service_name": "Ultrasound",
                "amount": 4000.0,
                "formatted_amount": "₹4,000",
                "count": 18
              }
            ],
            "monthly_revenue_trend": {
              "trend_label": "+12.4% vs last month",
              "sparkline_points": [
                32000.0,
                35000.0,
                38000.0,
                42000.0,
                45000.0,
                48200.0
              ]
            }
          }
        }
      ],
      "meta": {
        "total": 124,
        "page": 1,
        "page_size": 10,
        "total_pages": 13
      }
    }
  }
}
```
