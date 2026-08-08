# HMS Notification Platform — API Specification & Multi-User Routing Guide

This document defines the complete API specification, event schemas, and multi-user recipient routing logic (branch isolation, role broadcasts, direct doctor targeting) for the HMS Notification System.

---

## 1. Multi-User Recipient Routing Logic ("Who Gets What & How Many?")

When a business event occurs, the system determines **which specific users receive the notification** based on two target resolution strategies:

### **Strategy A: Direct Personal Targeting (`EVENT_USER`)**
- **Target Key**: `doctor_id`, `patient_id`, `uploader_id`, etc.
- **Recipient Scope**: Exactly **1 specific user**.
- **Example**: When an appointment is booked for Dr. Rajesh, `doctor_id = "f1837d2a-..."`.
- **Result**: **Only Dr. Rajesh receives the real-time popup and notification drawer entry.** Other doctors will NOT receive it.

---

### **Strategy B: Branch-Scoped Role Broadcast (`ROLE` + `branch_id`)**
- **Target Key**: Role name (e.g., `RECEPTIONIST`, `LAB_TECHNICIAN`, `NURSE`, `CASHIER`, `OT_MANAGER`).
- **Recipient Scope**: **All active users with that role assigned to THAT specific hospital branch/facility.**

#### 🏢 Multi-User Branch Isolation Example:
Suppose a hospital system has **100 Receptionists** across 10 hospital branches:
- 5 Receptionists work at **Branch A (City Center)**.
- 10 Receptionists work at **Branch B (Westside)**.

When a patient completes consultation at **Branch A**:
1. Event is published with `branch_id = "branch-A-uuid"`.
2. Recipient resolver queries active users where `role = 'RECEPTIONIST'` AND `facility_id = 'branch-A-uuid'`.
3. **Result**: **Only the 5 Receptionists at Branch A receive the notification.** The 95 receptionists at other branches do NOT receive it.

#### 🧪 Laboratory Department Example (2-3 Lab Technicians):
When a doctor orders a blood test at Branch A:
1. Event `lab.order.created` is emitted for `branch_id = "branch-A-uuid"`.
2. All **3 active Lab Technicians at Branch A** receive the real-time WebSocket popup simultaneously.
3. Whichever technician is free can tap the notification and process the lab order.

---

## 2. HMS Event & Recipient Matrix (15 Workflow Events)

| # | HMS Workflow Event | Event Key (`event_type`) | Target Recipients & Scope | Priority | Entity Reference |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1** | OPD Appointment Confirmed | `appointment.created` | • **Doctor** (Direct `doctor_id`)<br>• **Branch Nurses** (Role `NURSE` at `branch_id`) | `HIGH` | `APPOINTMENT` |
| **2** | Patient Checked In | `opd.patient.checked_in` | • **Doctor** (Direct `doctor_id`)<br>• **Branch Nurses** (Role `NURSE` at `branch_id`) | `HIGH` | `OPD_VISIT` |
| **3** | Consultation Started | `opd.consultation.started` | • **Branch Receptionists** (Role `RECEPTIONIST` at `branch_id`)<br>• **Branch Nurses** (Role `NURSE`) | `NORMAL` | `OPD_VISIT` |
| **4** | Consultation Completed | `opd.consultation.completed` | • **Branch Receptionists** (Role `RECEPTIONIST` at `branch_id`) | `NORMAL` | `OPD_VISIT` |
| **5** | Lab Order Created | `lab.order.created` | • **Branch Lab Techs** (All Role `LAB_TECHNICIAN` at `branch_id`) | `HIGH` | `LAB_ORDER` |
| **6** | Lab Result Ready | `lab.result.ready` | • **Ordering Doctor** (Direct `doctor_id`)<br>• **Branch Nurses** (Role `NURSE`) | `HIGH` | `LAB_ORDER` |
| **7** | IPD Recommended | `ipd.admission.recommended` | • **Branch Receptionists** (Role `RECEPTIONIST` at `branch_id`) | `HIGH` | `ADMISSION_RECOMMENDATION` |
| **8** | IPD Admission Confirmed | `ipd.admission.created` | • **Admitting Doctor** (Direct `doctor_id`)<br>• **Ward Nurses** (Role `NURSE` at `branch_id`) | `HIGH` | `ADMISSION` |
| **9** | Bed Assigned | `bed.assigned` | • **Ward Nurses** (Role `NURSE` at `branch_id`)<br>• **Attending Doctor** (Direct `doctor_id`)<br>• **IPD Manager** (Role `IPD_MANAGER`) | `NORMAL` | `BED` |
| **10** | Bed Transferred | `bed.transferred` | • **Ward Nurses** (Role `NURSE` at `branch_id`)<br>• **Attending Doctor** (Direct `doctor_id`) | `NORMAL` | `BED` |
| **11** | OT Requested | `ot.request.created` | • **Branch Receptionists** (Role `RECEPTIONIST`)<br>• **Ward Nurses** (Role `NURSE`)<br>• **OT Team** (Role `OT_MANAGER`) | `HIGH` | `SURGERY_REQUEST` |
| **12** | OT Scheduled | `ot.scheduled` | • **Surgeon** (Direct `doctor_id`)<br>• **OT Nurses** (Role `NURSE`)<br>• **OT Team** (Role `OT_MANAGER`) | `HIGH` | `OT_SCHEDULE` |
| **13** | Discharge Initiated | `discharge.initiated` | • **Ward Nurses** (Role `NURSE`)<br>• **Billing Staff** (Role `CASHIER` / `RECEPTIONIST`) | `NORMAL` | `ADMISSION` |
| **14** | Billing Cleared | `discharge.billing_cleared` | • **Ward Nurses** (Role `NURSE`)<br>• **Attending Doctor** (Direct `doctor_id`) | `NORMAL` | `INVOICE` |
| **15** | Discharge Completed | `discharge.completed` | • **IPD Manager** (Role `IPD_MANAGER`)<br>• **Attending Doctor** (Direct `doctor_id`)<br>• **Ward Nurses** (Role `NURSE`) | `NORMAL` | `ADMISSION` |

---

## 3. Real-Time WebSocket API Specification

### **Connection Request**
```http
wss://<api-id>.execute-api.<region>.amazonaws.com/<stage>?token=<JWT_ACCESS_TOKEN>&device_type=DESKTOP
```

- **Authentication**: Validated during `$connect` handshake using Cognito JWT Access Token.
- **Session Registry**: Connection ID is stored in `notification.websocket_connections` mapped to `user_id`.

---

### **Server-to-Client WebSocket Events**

#### **1. Real-Time Notification Push (`NOTIFICATION_POPUP`)**
```json
{
  "type": "NOTIFICATION_POPUP",
  "notification": {
    "id": "ab106800-6e63-4e24-aca9-fcc75c992e70",
    "event_type": "opd.patient.checked_in",
    "title": "Patient Checked In: Anita Verma",
    "message": "Patient Anita Verma (Token #12) has checked in for OPD consultation with Dr. Rajesh Kumar.",
    "priority": "HIGH",
    "entity_type": "OPD_VISIT",
    "entity_id": "26ec66ff-cf1c-40da-8850-c5f549a33fe2",
    "status": "UNREAD",
    "read_at": null,
    "created_at": "2026-08-05T10:45:00Z"
  }
}
```

#### **2. Client Heartbeat Ping (`$default`)**
Client sends:
```json
{ "action": "$default" }
```
Server responds:
```json
{ "type": "PONG", "status": "ALIVE" }
```

---

## 4. REST Notification Center APIs

Base Headers for all REST requests:
```http
Authorization: Bearer <JWT_ACCESS_TOKEN>
Content-Type: application/json
```

---

### **Endpoint 1: Fetch Notification Drawer List**
- **Method**: `GET`
- **Path**: `/notifications`
- **Query Params**:
  - `status`: Optional filter (`UNREAD`, `READ`, `ARCHIVED`).
  - `page`: Page number (default: `1`).
  - `limit`: Items per page (default: `20`).

#### **Response (`200 OK`)**:
```json
{
  "success": true,
  "data": {
    "notifications": [
      {
        "id": "ab106800-6e63-4e24-aca9-fcc75c992e70",
        "event_id": "2226ef52-94eb-4136-acc5-b582c34e7006",
        "title": "Patient Checked In: Anita Verma",
        "message": "Patient Anita Verma (Token #12) has checked in for OPD consultation.",
        "priority": "HIGH",
        "entity_type": "OPD_VISIT",
        "entity_id": "26ec66ff-cf1c-40da-8850-c5f549a33fe2",
        "status": "UNREAD",
        "read_at": null,
        "created_at": "2026-08-05T10:45:00Z"
      }
    ],
    "total": 1,
    "unread_count": 1,
    "page": 1,
    "limit": 20
  }
}
```

---

### **Endpoint 2: Fetch Unread Badge Count (Top Bell Icon)**
- **Method**: `GET`
- **Path**: `/notifications/unread-count`

#### **Response (`200 OK`)**:
```json
{
  "success": true,
  "data": {
    "unread_count": 3
  }
}
```

---

### **Endpoint 3: Mark Single Notification as Read**
- **Method**: `PATCH`
- **Path**: `/notifications/{id}/read`

#### **Response (`200 OK`)**:
```json
{
  "success": true,
  "message": "Notification marked as read"
}
```

---

### **Endpoint 4: Mark All Notifications as Read**
- **Method**: `PATCH`
- **Path**: `/notifications/read-all`

#### **Response (`200 OK`)**:
```json
{
  "success": true,
  "data": {
    "marked_read_count": 3
  }
}
```

---

### **Endpoint 5: Delete / Dismiss Notification**
- **Method**: `DELETE`
- **Path**: `/notifications/{id}`

#### **Response (`200 OK`)**:
```json
{
  "success": true,
  "message": "Notification deleted successfully"
}
```

---

### **Endpoint 6: Get Notification Preferences**
- **Method**: `GET`
- **Path**: `/notifications/preferences`

#### **Response (`200 OK`)**:
```json
{
  "success": true,
  "data": {
    "user_id": "f1837d2a-70e1-7077-6ad7-814aa786ca9d",
    "push_enabled": true,
    "email_enabled": true,
    "sms_enabled": true,
    "whatsapp_enabled": true,
    "quiet_hours_start": "22:00",
    "quiet_hours_end": "07:00"
  }
}
```

---

### **Endpoint 7: Update Notification Preferences**
- **Method**: `PUT`
- **Path**: `/notifications/preferences`

#### **Request Body**:
```json
{
  "push_enabled": true,
  "email_enabled": false,
  "sms_enabled": true,
  "whatsapp_enabled": true,
  "quiet_hours_start": "23:00",
  "quiet_hours_end": "06:00"
}
```
