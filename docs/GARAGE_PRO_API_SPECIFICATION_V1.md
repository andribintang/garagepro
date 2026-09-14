# GARAGE PRO --- API SPECIFICATION V1

**Document Type:** REST API Specification\
**Product:** GARAGE PRO --- Workshop Management System\
**Version:** V1.0\
**Base Path:** `/api/v1`\
**Backend:** Node.js + Express + TypeScript\
**Database:** MySQL 8+\
**ORM:** Sequelize\
**Authentication:** JWT\
**Timezone:** Asia/Jakarta\
**Status:** Development Ready\
**Date:** 14 September 2026

------------------------------------------------------------------------

# 1. Purpose

Dokumen ini mendefinisikan kontrak REST API GARAGE PRO V1.

API menjadi penghubung resmi antara:

``` text
React Frontend
      |
      v
REST API /api/v1
      |
      v
Business Services
      |
      v
Sequelize ORM
      |
      v
MySQL
```

Dokumen ini harus konsisten dengan:

`GARAGE_PRO_MASTER_DEVELOPMENT_SPECIFICATION_V1.md`

dan

`GARAGE_PRO_DATABASE_DESIGN_V1.md`

------------------------------------------------------------------------

# 2. API Principles

1.  RESTful.
2.  JSON request/response.
3.  Semua endpoint menggunakan `/api/v1`.
4.  Authentication menggunakan Bearer JWT.
5.  Authorization dilakukan di backend.
6.  Validasi dilakukan sebelum business logic.
7.  Financial/inventory mutation menggunakan database transaction.
8.  Error response konsisten.
9.  Pagination konsisten.
10. Jangan expose database/internal error.
11. Jangan menerima total invoice dari frontend sebagai sumber
    kebenaran.
12. Jangan menerima balance stock dari frontend.
13. Transaction number dibuat server-side.
14. Semua mutation penting dicatat pada audit log.

------------------------------------------------------------------------

# 3. Base URL

Development:

``` text
http://localhost:5000/api/v1
```

Production:

``` text
https://<domain>/api/v1
```

Frontend membaca dari:

``` text
VITE_API_URL
```

------------------------------------------------------------------------

# 4. Authentication

## 4.1 Login

``` http
POST /auth/login
```

Auth required:

``` text
NO
```

Request:

``` json
{
  "username": "admin",
  "password": "********"
}
```

Validation:

``` text
username: required, string
password: required, string, min 6
```

Success:

``` json
{
  "success": true,
  "message": "Login successful",
  "data": {
    "user": {
      "id": 1,
      "username": "admin",
      "name": "Administrator",
      "role": {
        "code": "ADMIN",
        "name": "Admin"
      }
    },
    "accessToken": "JWT_TOKEN"
  }
}
```

Errors:

``` text
401 AUTH_INVALID_CREDENTIALS
403 AUTH_USER_INACTIVE
```

------------------------------------------------------------------------

# 5. Current User

``` http
GET /auth/me
```

Auth:

``` text
Bearer JWT
```

Response:

``` json
{
  "success": true,
  "data": {
    "id": 1,
    "username": "admin",
    "name": "Administrator",
    "role": "ADMIN",
    "permissions": [
      "customer.view",
      "customer.create",
      "work_order.create"
    ]
  }
}
```

------------------------------------------------------------------------

# 6. Logout

``` http
POST /auth/logout
```

Auth required.

If using stateless JWT, logout may invalidate refresh token/session
record.

Response:

``` json
{
  "success": true,
  "message": "Logout successful"
}
```

------------------------------------------------------------------------

# 7. Common Response Format

## Success

``` json
{
  "success": true,
  "message": "Operation successful",
  "data": {},
  "meta": null
}
```

## Error

``` json
{
  "success": false,
  "message": "Validation failed",
  "errors": [
    {
      "field": "phone",
      "code": "REQUIRED",
      "message": "Phone is required"
    }
  ],
  "meta": null
}
```

------------------------------------------------------------------------

# 8. HTTP Status Codes

  Status   Meaning
  -------- -----------------------
  200      Success
  201      Created
  204      Success without body
  400      Bad Request
  401      Unauthorized
  403      Forbidden
  404      Not Found
  409      Conflict
  422      Validation Error
  429      Too Many Requests
  500      Internal Server Error

------------------------------------------------------------------------

# 9. Standard Error Codes

``` text
AUTH_INVALID_CREDENTIALS
AUTH_TOKEN_INVALID
AUTH_TOKEN_EXPIRED
AUTH_USER_INACTIVE

VALIDATION_ERROR
RESOURCE_NOT_FOUND
DUPLICATE_RESOURCE
RESOURCE_CONFLICT

INSUFFICIENT_STOCK
INVALID_STATUS_TRANSITION
INVALID_PAYMENT
INVOICE_ALREADY_PAID
WORK_ORDER_LOCKED

DATABASE_ERROR
INTERNAL_ERROR
```

------------------------------------------------------------------------

# 10. Pagination

List endpoints support:

``` text
?page=1
&limit=20
&search=
&sortBy=created_at
&sortOrder=desc
```

Response:

``` json
{
  "success": true,
  "data": [],
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 125,
    "totalPages": 7
  }
}
```

Maximum limit:

``` text
100
```

------------------------------------------------------------------------

# 11. Customer API

## 11.1 List

``` http
GET /customers
```

Permission:

``` text
customer.view
```

Query:

``` text
search
status
page
limit
sortBy
sortOrder
```

Response data:

``` json
[
  {
    "id": 1,
    "customer_code": "CUS-000001",
    "name": "Budi Santoso",
    "phone": "08123456789",
    "vehicle_count": 2,
    "last_service_at": "2026-09-14T09:00:00+07:00"
  }
]
```

------------------------------------------------------------------------

## 11.2 Create

``` http
POST /customers
```

Permission:

``` text
customer.create
```

Request:

``` json
{
  "name": "Budi Santoso",
  "phone": "08123456789",
  "email": "budi@example.com",
  "address": "Bogor",
  "notes": ""
}
```

Rules:

-   name required
-   phone required
-   phone normalized
-   customer_code generated server-side

Response:

``` text
201 Created
```

------------------------------------------------------------------------

## 11.3 Detail

``` http
GET /customers/:id
```

Permission:

``` text
customer.view
```

Response includes:

``` text
customer
vehicles
summary
service_count
last_service
```

------------------------------------------------------------------------

## 11.4 Update

``` http
PUT /customers/:id
```

Permission:

``` text
customer.update
```

Audit log required.

------------------------------------------------------------------------

## 11.5 Delete

``` http
DELETE /customers/:id
```

Permission:

``` text
customer.delete
```

Use soft delete.

If customer has historical transactions:

``` text
deactivate/soft-delete only
```

------------------------------------------------------------------------

# 12. Vehicle API

## 12.1 List

``` http
GET /vehicles
```

Query:

``` text
search
customer_id
brand
model
page
limit
```

Search: - plate number - customer name - vehicle code

------------------------------------------------------------------------

## 12.2 Create

``` http
POST /vehicles
```

Request:

``` json
{
  "customer_id": 1,
  "plate_number": "B 1234 XYZ",
  "brand": "Honda",
  "model": "Beat",
  "variant": "CBS",
  "year": 2022,
  "color": "Black",
  "current_km": 18250
}
```

Backend normalizes plate:

``` text
B1234XYZ
```

------------------------------------------------------------------------

## 12.3 Detail

``` http
GET /vehicles/:id
```

Response:

``` json
{
  "id": 3,
  "vehicle_code": "VH-000003",
  "plate_number": "B1234XYZ",
  "display_plate_number": "B 1234 XYZ",
  "customer": {},
  "current_km": 18250,
  "last_service": {}
}
```

------------------------------------------------------------------------

## 12.4 Update

``` http
PUT /vehicles/:id
```

Rules:

-   current_km cannot decrease below latest recorded KM unless
    privileged correction.
-   important correction must be audited.

------------------------------------------------------------------------

## 12.5 Service History

``` http
GET /vehicles/:id/history
```

Query:

``` text
page
limit
from
to
```

Response:

``` json
{
  "vehicle": {},
  "history": [
    {
      "work_order_number": "WO-20260914-0001",
      "date": "2026-09-14",
      "km": 18250,
      "services": [],
      "parts": [],
      "invoice": {}
    }
  ]
}
```

------------------------------------------------------------------------

# 13. Mechanic API

## List

``` http
GET /mechanics
```

Permission:

``` text
mechanic.view
```

Query:

``` text
status
search
```

## Create

``` http
POST /mechanics
```

## Detail

``` http
GET /mechanics/:id
```

## Update

``` http
PUT /mechanics/:id
```

## Assigned Work Orders

``` http
GET /mechanics/:id/work-orders
```

Query:

``` text
status
date
```

Mechanic role may only access own/assigned data unless elevated
permission exists.

------------------------------------------------------------------------

# 14. Service Category API

``` http
GET /service-categories
POST /service-categories
GET /service-categories/:id
PUT /service-categories/:id
DELETE /service-categories/:id
```

Permissions:

``` text
service_category.view
service_category.create
service_category.update
service_category.delete
```

------------------------------------------------------------------------

# 15. Service API

## List

``` http
GET /services
```

Query:

``` text
search
category_id
status
page
limit
```

## Create

``` http
POST /services
```

Request:

``` json
{
  "category_id": 1,
  "name": "Service CVT",
  "description": "Service dan pembersihan CVT",
  "price": 75000,
  "estimated_duration_minutes": 60
}
```

## Detail

``` http
GET /services/:id
```

## Update

``` http
PUT /services/:id
```

## Delete

``` http
DELETE /services/:id
```

Soft delete.

------------------------------------------------------------------------

# 16. Part Category API

``` http
GET /part-categories
POST /part-categories
GET /part-categories/:id
PUT /part-categories/:id
DELETE /part-categories/:id
```

------------------------------------------------------------------------

# 17. Spare Part API

## List

``` http
GET /parts
```

Query:

``` text
search
category_id
brand
status
low_stock
page
limit
```

Search: - SKU - barcode - name - brand

------------------------------------------------------------------------

## Create

``` http
POST /parts
```

Request:

``` json
{
  "sku": "BRK-BEAT-001",
  "barcode": "8991234567890",
  "name": "Kampas Rem Beat",
  "category_id": 2,
  "brand": "Example",
  "unit": "PCS",
  "purchase_price": 35000,
  "selling_price": 50000,
  "minimum_stock": 5
}
```

Rules:

-   SKU unique
-   barcode unique when present
-   prices \>= 0
-   minimum_stock \>= 0

------------------------------------------------------------------------

## Detail

``` http
GET /parts/:id
```

Response:

``` json
{
  "id": 1,
  "sku": "BRK-BEAT-001",
  "name": "Kampas Rem Beat",
  "pricing": {
    "purchase_price": 35000,
    "selling_price": 50000
  },
  "stock": {
    "total": 8
  }
}
```

------------------------------------------------------------------------

## Update

``` http
PUT /parts/:id
```

Master changes must not mutate historical transactions.

------------------------------------------------------------------------

## Delete

``` http
DELETE /parts/:id
```

Soft delete.

------------------------------------------------------------------------

## Part Movements

``` http
GET /parts/:id/movements
```

Query:

``` text
warehouse_id
movement_type
from
to
page
limit
```

------------------------------------------------------------------------

# 18. Warehouse API

``` http
GET /warehouses
POST /warehouses
GET /warehouses/:id
PUT /warehouses/:id
```

------------------------------------------------------------------------

# 19. Warehouse Location API

``` http
GET /warehouses/:warehouseId/locations
POST /warehouses/:warehouseId/locations
PUT /warehouse-locations/:id
DELETE /warehouse-locations/:id
```

------------------------------------------------------------------------

# 20. Work Order API

This is the core API.

## 20.1 List

``` http
GET /work-orders
```

Query:

``` text
search
status
mechanic_id
customer_id
vehicle_id
date_from
date_to
page
limit
sortBy
sortOrder
```

Search: - WO number - customer - phone - plate number

------------------------------------------------------------------------

## 20.2 Create

``` http
POST /work-orders
```

Permission:

``` text
work_order.create
```

Request:

``` json
{
  "customer_id": 1,
  "vehicle_id": 3,
  "mechanic_id": 5,
  "warehouse_id": 1,
  "current_km": 18250,
  "complaint": "Mesin terasa berisik",
  "customer_notes": ""
}
```

Backend:

1.  Validate customer.
2.  Validate vehicle belongs to customer.
3.  Validate mechanic.
4.  Validate warehouse.
5.  Generate WO number.
6.  Create WO.
7.  Write audit log.

Initial status:

``` text
NEW
```

------------------------------------------------------------------------

# 21. Work Order Detail

``` http
GET /work-orders/:id
```

Response should include:

``` text
work_order
customer
vehicle
mechanic
inspection
services
parts
recommendations
invoice
payments
timeline
```

------------------------------------------------------------------------

# 22. Update Work Order

``` http
PUT /work-orders/:id
```

Only editable fields based on current status.

Rules:

``` text
NEW/CHECKING/ESTIMATE:
editable

APPROVED:
limited

IN_PROGRESS:
limited

QC:
locked except QC fields

READY:
locked

INVOICED:
locked

PAID:
locked

COMPLETED:
locked
```

Attempt to edit locked fields returns:

``` text
409 WORK_ORDER_LOCKED
```

------------------------------------------------------------------------

# 23. Inspection API

## Save Inspection

``` http
POST /work-orders/:id/inspection
```

Request:

``` json
{
  "items": [
    {
      "inspection_item_id": 1,
      "result": "NORMAL",
      "measurement": "",
      "notes": ""
    },
    {
      "inspection_item_id": 2,
      "result": "REPLACE",
      "measurement": "",
      "notes": "Kampas sudah tipis",
      "recommendation": "Ganti kampas rem"
    }
  ]
}
```

Backend must upsert inspection items.

Status may move:

``` text
NEW -> CHECKING
```

------------------------------------------------------------------------

# 24. Add Service to WO

``` http
POST /work-orders/:id/services
```

Request:

``` json
{
  "service_id": 3,
  "qty": 1,
  "unit_price": 75000,
  "discount": 0
}
```

Important:

Frontend may send price, but backend must validate/retrieve current
master price according to business policy.

Recommended:

``` text
unit_price omitted
→ use master price

unit_price supplied
→ require price override permission if different
```

Response:

``` json
{
  "id": 10,
  "service_id": 3,
  "qty": 1,
  "unit_price": 75000,
  "discount": 0,
  "subtotal": 75000
}
```

------------------------------------------------------------------------

# 25. Update WO Service

``` http
PUT /work-orders/:id/services/:itemId
```

Rules: - item must belong to WO - locked WO cannot be edited - price
override permission required if applicable

------------------------------------------------------------------------

# 26. Remove WO Service

``` http
DELETE /work-orders/:id/services/:itemId
```

Do not allow when WO is locked.

------------------------------------------------------------------------

# 27. Add Spare Part to WO

``` http
POST /work-orders/:id/parts
```

Request:

``` json
{
  "part_id": 1,
  "warehouse_id": 1,
  "location_id": 2,
  "qty": 1
}
```

Important:

Adding an estimated part does NOT automatically reduce stock.

It creates a requested item:

``` text
stock_issued = false
```

------------------------------------------------------------------------

# 28. Issue Spare Part

``` http
POST /work-orders/:id/parts/:itemId/issue
```

Permission:

``` text
work_order.issue_part
```

Backend transaction:

``` text
BEGIN

1. Lock inventory state.
2. Check available stock.
3. If insufficient -> rollback.
4. Get current cost.
5. Create stock movement OUT.
6. Set stock_issued=true.
7. Save issued_by and issued_at.

COMMIT
```

Error:

``` text
409 INSUFFICIENT_STOCK
```

------------------------------------------------------------------------

# 29. Return Spare Part

``` http
POST /work-orders/:id/parts/:itemId/return
```

Only issued quantity may be returned.

Creates:

``` text
RETURN
```

stock movement.

Never delete original OUT movement.

------------------------------------------------------------------------

# 30. Recommendations API

## List

``` http
GET /work-orders/:id/recommendations
```

## Create

``` http
POST /work-orders/:id/recommendations
```

Request:

``` json
{
  "service_id": null,
  "part_id": 5,
  "description": "Ganti ban depan",
  "estimated_price": 250000,
  "priority": "MEDIUM",
  "notes": "Ban mulai aus"
}
```

## Approve

``` http
POST /work-orders/:id/recommendations/:recommendationId/approve
```

## Decline

``` http
POST /work-orders/:id/recommendations/:recommendationId/decline
```

## Convert

``` http
POST /work-orders/:id/recommendations/:recommendationId/convert
```

Conversion must happen inside a transaction.

------------------------------------------------------------------------

# 31. Work Order Approval

``` http
POST /work-orders/:id/approve
```

Permission:

``` text
work_order.approve
```

Request:

``` json
{
  "approved": true,
  "notes": ""
}
```

Success:

``` text
status = APPROVED
approval_status = APPROVED
approval_at = now
approved_by = current user
```

Rejected:

``` text
status = REJECTED
approval_status = REJECTED
```

------------------------------------------------------------------------

# 32. Start Work

``` http
POST /work-orders/:id/start
```

Permission:

``` text
work_order.start
```

Precondition:

``` text
status == APPROVED
```

Result:

``` text
status = IN_PROGRESS
started_at = now
```

------------------------------------------------------------------------

# 33. QC

``` http
POST /work-orders/:id/qc
```

Request:

``` json
{
  "passed": true,
  "final_km": 18260,
  "notes": "Unit normal"
}
```

If passed:

``` text
IN_PROGRESS -> QC -> READY
```

If failed:

``` text
QC -> REWORK
```

------------------------------------------------------------------------

# 34. Rework

``` http
POST /work-orders/:id/rework
```

Request:

``` json
{
  "reason": "Suara masih muncul setelah test"
}
```

Result:

``` text
status = REWORK
```

Then:

``` text
REWORK -> IN_PROGRESS
```

------------------------------------------------------------------------

# 35. Mark Ready

``` http
POST /work-orders/:id/ready
```

Permission:

``` text
work_order.ready
```

Precondition:

``` text
QC passed
```

Result:

``` text
status = READY
```

------------------------------------------------------------------------

# 36. Create Invoice

``` http
POST /work-orders/:id/invoice
```

Permission:

``` text
invoice.create
```

Preconditions:

``` text
status in [READY]
```

Backend must calculate:

``` text
service_subtotal
part_subtotal
discount
tax
grand_total
```

Frontend-provided total must NOT be trusted.

Request:

``` json
{
  "discount": 5000,
  "tax": 0
}
```

Response:

``` json
{
  "invoice_number": "INV-20260914-0001",
  "subtotal_service": 95000,
  "subtotal_part": 100000,
  "discount": 5000,
  "tax": 0,
  "grand_total": 190000,
  "status": "ISSUED"
}
```

------------------------------------------------------------------------

# 37. Invoice API

## List

``` http
GET /invoices
```

Query:

``` text
search
status
date_from
date_to
page
limit
```

## Detail

``` http
GET /invoices/:id
```

## Void

``` http
POST /invoices/:id/void
```

Permission:

``` text
invoice.void
```

Cannot void a paid invoice without elevated reversal process.

------------------------------------------------------------------------

# 38. Payment API

## Create Payment

``` http
POST /payments
```

Request:

``` json
{
  "invoice_id": 1,
  "payment_method": "CASH",
  "amount": 190000,
  "reference_number": null,
  "notes": ""
}
```

Backend transaction:

``` text
BEGIN

1. Lock invoice.
2. Calculate current paid amount.
3. Validate amount.
4. Create payment.
5. Update invoice paid amount.
6. Update outstanding.
7. Update invoice status.
8. If fully paid, update WO status.

COMMIT
```

Cannot exceed outstanding.

------------------------------------------------------------------------

# 39. Payment Detail

``` http
GET /payments/:id
```

## Payment List

``` http
GET /payments
```

Query:

``` text
invoice_id
payment_method
date_from
date_to
page
limit
```

------------------------------------------------------------------------

# 40. Complete Work Order

``` http
POST /work-orders/:id/complete
```

Permission:

``` text
work_order.complete
```

Precondition:

``` text
invoice exists
invoice.status == PAID
```

Result:

``` text
status = COMPLETED
completed_at = now
```

Also update vehicle:

``` text
vehicles.current_km = work_orders.final_km
```

when applicable.

------------------------------------------------------------------------

# 41. Cancel Work Order

``` http
POST /work-orders/:id/cancel
```

Request:

``` json
{
  "reason": "Customer membatalkan pekerjaan"
}
```

Rules: - cannot cancel completed WO - issued parts must be returned or
handled by approved reversal process - audit required

------------------------------------------------------------------------

# 42. Workshop Board API

``` http
GET /work-orders/board
```

Response:

``` json
{
  "NEW": [],
  "CHECKING": [],
  "WAITING_APPROVAL": [],
  "APPROVED": [],
  "IN_PROGRESS": [],
  "QC": [],
  "READY": []
}
```

Optional filters:

``` text
date
mechanic_id
```

------------------------------------------------------------------------

# 43. Inventory API

## Stock Overview

``` http
GET /inventory
```

Query:

``` text
search
warehouse_id
category_id
low_stock
page
limit
```

Response:

``` json
{
  "part_id": 1,
  "sku": "BRK-BEAT-001",
  "name": "Kampas Rem Beat",
  "stock": 8,
  "minimum_stock": 5,
  "stock_status": "NORMAL"
}
```

------------------------------------------------------------------------

# 44. Low Stock

``` http
GET /inventory/low-stock
```

Response:

``` json
[
  {
    "part_id": 5,
    "sku": "AKI-001",
    "name": "Aki MF",
    "stock": 1,
    "minimum_stock": 3,
    "shortage": 2
  }
]
```

------------------------------------------------------------------------

# 45. Stock Adjustment

``` http
POST /inventory/adjustment
```

Request:

``` json
{
  "part_id": 1,
  "warehouse_id": 1,
  "location_id": 2,
  "type": "ADJUSTMENT_OUT",
  "quantity": 1,
  "reason": "Barang rusak"
}
```

Transaction:

``` text
BEGIN
validate
create movement
commit
```

------------------------------------------------------------------------

# 46. Stock Movement List

``` http
GET /inventory/movements
```

Query:

``` text
part_id
warehouse_id
movement_type
reference_type
date_from
date_to
page
limit
```

------------------------------------------------------------------------

# 47. Stock Opname API

## Create

``` http
POST /inventory/stock-opnames
```

## Detail

``` http
GET /inventory/stock-opnames/:id
```

## Add Item

``` http
POST /inventory/stock-opnames/:id/items
```

## Update Count

``` http
PUT /inventory/stock-opnames/:id/items/:itemId
```

## Post

``` http
POST /inventory/stock-opnames/:id/post
```

Posting creates movement records.

------------------------------------------------------------------------

# 48. Supplier API

``` http
GET /suppliers
POST /suppliers
GET /suppliers/:id
PUT /suppliers/:id
DELETE /suppliers/:id
```

------------------------------------------------------------------------

# 49. Purchase API

## List

``` http
GET /purchases
```

## Create

``` http
POST /purchases
```

Request:

``` json
{
  "supplier_id": 1,
  "warehouse_id": 1,
  "items": [
    {
      "part_id": 1,
      "qty": 10,
      "unit_cost": 35000
    }
  ],
  "discount": 0,
  "notes": ""
}
```

Backend calculates subtotals.

------------------------------------------------------------------------

# 50. Purchase Detail

``` http
GET /purchases/:id
```

## Update

``` http
PUT /purchases/:id
```

Only allowed before receiving.

------------------------------------------------------------------------

# 51. Receive Purchase

``` http
POST /purchases/:id/receive
```

Request:

``` json
{
  "items": [
    {
      "purchase_item_id": 1,
      "received_qty": 10
    }
  ]
}
```

Backend transaction:

``` text
purchase_items.received_qty += received_qty
create STOCK IN movement
update purchase status
```

------------------------------------------------------------------------

# 52. Reports API

## Revenue

``` http
GET /reports/revenue
```

Query:

``` text
date_from
date_to
payment_method
```

Response:

``` json
{
  "gross_revenue": 5000000,
  "discount": 100000,
  "net_revenue": 4900000,
  "service_revenue": 2500000,
  "part_revenue": 2400000,
  "transaction_count": 25
}
```

------------------------------------------------------------------------

# 53. Work Order Report

``` http
GET /reports/work-orders
```

Metrics:

``` text
total
completed
cancelled
in_progress
average_ticket
```

------------------------------------------------------------------------

# 54. Service Report

``` http
GET /reports/services
```

Metrics: - service count - quantity - revenue - top services

------------------------------------------------------------------------

# 55. Part Report

``` http
GET /reports/parts
```

Metrics: - consumed quantity - sales revenue - cost - gross margin - top
parts

------------------------------------------------------------------------

# 56. Mechanic Report

``` http
GET /reports/mechanics
```

Metrics: - assigned WO - completed WO - service revenue - part revenue -
average ticket

------------------------------------------------------------------------

# 57. Stock Report

``` http
GET /reports/stock
```

Metrics: - total SKU - total units - stock value - low stock - movement
summary

------------------------------------------------------------------------

# 58. Dashboard API

``` http
GET /dashboard/summary
```

Response:

``` json
{
  "date": "2026-09-14",
  "kpis": {
    "work_orders": 18,
    "revenue": 4850000,
    "in_progress": 5,
    "ready": 4
  },
  "work_order_status": {},
  "low_stock": [],
  "recent_work_orders": [],
  "recent_payments": []
}
```

------------------------------------------------------------------------

# 59. User API

Owner only.

``` http
GET /users
POST /users
GET /users/:id
PUT /users/:id
POST /users/:id/reset-password
POST /users/:id/activate
POST /users/:id/deactivate
```

Passwords are always hashed.

------------------------------------------------------------------------

# 60. Role & Permission API

Owner only.

``` http
GET /roles
GET /permissions
GET /roles/:id/permissions
PUT /roles/:id/permissions
```

Do not allow removing the final OWNER access accidentally.

------------------------------------------------------------------------

# 61. Settings API

``` http
GET /settings
PUT /settings
```

Owner only.

Sensitive settings must be protected.

------------------------------------------------------------------------

# 62. Audit Log API

Owner:

``` http
GET /audit-logs
```

Admin may receive limited access if explicitly permitted.

Query:

``` text
user_id
action
entity_type
entity_id
date_from
date_to
page
limit
```

Audit log records should be immutable.

------------------------------------------------------------------------

# 63. State Transition Matrix

  Current            Allowed Next
  ------------------ --------------------------------------------------
  NEW                CHECKING, CANCELLED
  CHECKING           ESTIMATE, CANCELLED
  ESTIMATE           WAITING_APPROVAL, CANCELLED
  WAITING_APPROVAL   APPROVED, REJECTED
  APPROVED           IN_PROGRESS, CANCELLED
  REJECTED           COMPLETED/CANCELLED according to business policy
  IN_PROGRESS        QC
  QC                 READY, REWORK
  REWORK             IN_PROGRESS
  READY              INVOICED
  INVOICED           PAID
  PAID               COMPLETED
  COMPLETED          none
  CANCELLED          none

Every transition must be validated server-side.

------------------------------------------------------------------------

# 64. Authorization Rules

Examples:

``` text
GET /customers
→ customer.view

POST /customers
→ customer.create

POST /work-orders/:id/approve
→ work_order.approve

POST /work-orders/:id/parts/:itemId/issue
→ work_order.issue_part

POST /invoices/:id/void
→ invoice.void

POST /payments
→ payment.create
```

Frontend visibility is not authorization.

Backend must enforce permission.

------------------------------------------------------------------------

# 65. Query Filtering Standards

Date filter:

``` text
date_from=2026-09-01
date_to=2026-09-14
```

Status:

``` text
status=IN_PROGRESS
```

Multiple status:

``` text
status=READY,QC
```

Backend converts to safe parameterized query.

------------------------------------------------------------------------

# 66. Search Standards

Search should support partial matching where appropriate.

Example:

``` text
GET /work-orders?search=1234
```

May match:

``` text
WO number
plate number
customer phone
customer name
```

Avoid unindexed wildcard searches on huge tables without a search
strategy.

------------------------------------------------------------------------

# 67. Idempotency

Financial and inventory mutation endpoints should support an idempotency
key.

Header:

``` text
Idempotency-Key: <unique-client-key>
```

Especially:

``` text
POST /payments
POST /work-orders/:id/parts/:itemId/issue
POST /purchases/:id/receive
POST /inventory/adjustment
```

Purpose:

Prevent duplicate transactions when user double-clicks or network
retries.

------------------------------------------------------------------------

# 68. Concurrency Rules

Critical endpoints must lock the relevant records.

Example payment:

``` text
BEGIN
SELECT invoice FOR UPDATE
calculate outstanding
validate
insert payment
update invoice
COMMIT
```

Example inventory:

``` text
BEGIN
lock inventory state
validate available quantity
create movement
commit
```

------------------------------------------------------------------------

# 69. API Validation Architecture

Recommended:

``` text
Route
  ↓
Auth Middleware
  ↓
Permission Middleware
  ↓
Validation Middleware
  ↓
Controller
  ↓
Service
  ↓
Repository/Sequelize
```

Controllers should not contain complex business rules.

------------------------------------------------------------------------

# 70. Service Layer

Recommended services:

``` text
AuthService
CustomerService
VehicleService
MechanicService
ServiceMasterService
PartService
WorkOrderService
InspectionService
InventoryService
PurchaseService
InvoiceService
PaymentService
ReportService
DashboardService
AuditService
```

------------------------------------------------------------------------

# 71. Inventory Service

Centralize all stock changes in:

``` text
InventoryService
```

No controller should directly modify stock.

Methods:

``` text
getStock()
issueStock()
returnStock()
adjustStock()
receiveStock()
postStockOpname()
getMovements()
```

------------------------------------------------------------------------

# 72. Work Order Service

Centralize:

``` text
createWO()
updateWO()
changeStatus()
addService()
updateService()
removeService()
addPart()
issuePart()
returnPart()
addRecommendation()
approveRecommendation()
convertRecommendation()
approveWO()
startWO()
performQC()
markReady()
cancelWO()
completeWO()
```

------------------------------------------------------------------------

# 73. Invoice Service

Methods:

``` text
calculateInvoice()
createInvoice()
getInvoice()
voidInvoice()
recalculateOutstanding()
```

Never trust frontend financial totals.

------------------------------------------------------------------------

# 74. Payment Service

Methods:

``` text
createPayment()
validatePayment()
updateInvoicePaymentState()
getPayment()
```

All payment operations must be transactional.

------------------------------------------------------------------------

# 75. Number Generator

Central service:

``` text
NumberingService
```

Methods:

``` text
generateCustomerCode()
generateVehicleCode()
generateWorkOrderNumber()
generateInvoiceNumber()
generatePaymentNumber()
generatePurchaseNumber()
generateAdjustmentNumber()
generateStockOpnameNumber()
```

Numbers must be generated server-side and safely under concurrency.

------------------------------------------------------------------------

# 76. Audit Service

Central:

``` text
AuditService.log()
```

Example:

``` typescript
await auditService.log({
  userId,
  action: 'PART_ISSUED',
  entityType: 'work_order_part',
  entityId: item.id,
  oldValues,
  newValues
});
```

------------------------------------------------------------------------

# 77. API Security Middleware

Recommended middleware:

``` text
helmet
cors
rateLimit
authentication
authorization
validation
requestId
errorHandler
```

Request ID:

``` text
X-Request-ID
```

Useful for debugging production errors.

------------------------------------------------------------------------

# 78. API Logging

Production log should include:

``` text
timestamp
request_id
method
path
user_id
status
duration_ms
```

Never log: - password - JWT token - sensitive credentials - payment
secrets

------------------------------------------------------------------------

# 79. File Uploads

V1 may support workshop logo.

Recommended architecture:

``` text
POST /settings/logo
```

If cloud storage is introduced later, use a storage service abstraction.

Do not hard-code storage provider throughout application code.

------------------------------------------------------------------------

# 80. API Documentation

Generate OpenAPI/Swagger from this specification.

Recommended:

``` text
/docs
```

Development only or protected in production.

OpenAPI should document: - schemas - endpoints - authentication -
responses - errors - examples

------------------------------------------------------------------------

# 81. API Acceptance Tests

Minimum E2E sequence:

``` text
LOGIN
  ↓
CREATE CUSTOMER
  ↓
CREATE VEHICLE
  ↓
CREATE WO
  ↓
INSPECTION
  ↓
ADD SERVICE
  ↓
ADD PART
  ↓
APPROVE WO
  ↓
ISSUE PART
  ↓
START WO
  ↓
QC
  ↓
READY
  ↓
CREATE INVOICE
  ↓
PAYMENT
  ↓
COMPLETE
  ↓
SERVICE HISTORY
  ↓
STOCK MOVEMENT
```

------------------------------------------------------------------------

# 82. Critical Negative Tests

Test these cases:

``` text
Invalid login
Inactive user login
Unauthorized endpoint
Forbidden permission
Duplicate customer code
Duplicate SKU
Duplicate barcode
Vehicle belongs to another customer
Invalid WO transition
Issue stock > available
Double issue same item
Double payment
Payment > outstanding
Edit paid WO
Void paid invoice
Receive purchase twice
Post stock opname twice
Negative quantity
Negative price
Invalid ID
```

------------------------------------------------------------------------

# 83. API Performance Targets

Normal environment:

``` text
GET list endpoint < 500ms
GET detail < 500ms
Simple mutation < 800ms
Dashboard < 1500ms
```

Targets are indicative and must be measured under realistic load.

------------------------------------------------------------------------

# 84. API Versioning

Current:

``` text
/api/v1
```

Breaking changes require:

``` text
/api/v2
```

Do not silently break V1 clients.

------------------------------------------------------------------------

# 85. Frontend API Client

Recommended Axios structure:

``` text
apiClient
  ├── authApi
  ├── customerApi
  ├── vehicleApi
  ├── mechanicApi
  ├── serviceApi
  ├── partApi
  ├── workOrderApi
  ├── inventoryApi
  ├── purchaseApi
  ├── invoiceApi
  ├── paymentApi
  ├── reportApi
  └── dashboardApi
```

Interceptors: - attach token - handle 401 - attach request ID -
normalize API errors

------------------------------------------------------------------------

# 86. API Contract Rule

Frontend must not invent API behavior.

If API contract changes:

1.  Update this document.
2.  Update OpenAPI.
3.  Update backend.
4.  Update frontend.
5.  Update tests.

------------------------------------------------------------------------

# 87. Development Sequence

Implement API in this order:

``` text
PHASE 1
Auth
Users
Roles
Permissions

PHASE 2
Customers
Vehicles
Mechanics
Service Master
Part Master

PHASE 3
Warehouse
Inventory
Stock Movement

PHASE 4
Work Order
Inspection
Services
Parts
Recommendations

PHASE 5
Purchase
Receiving

PHASE 6
Invoice
Payment

PHASE 7
Dashboard
Reports

PHASE 8
Audit
Hardening
Testing
```

------------------------------------------------------------------------

# 88. API Definition of Done

An endpoint is DONE when:

-   Route exists.
-   Auth is enforced.
-   Permission is enforced.
-   Validation exists.
-   Controller exists.
-   Service logic exists.
-   Database operation works.
-   Transaction is used where required.
-   Error handling exists.
-   Audit log exists where required.
-   Unit/integration test exists.
-   OpenAPI documentation is updated.
-   Frontend contract matches.
-   No sensitive data is exposed.

------------------------------------------------------------------------

# 89. Next Specification

After this API document, the next required document is:

``` text
GARAGE_PRO_UI_UX_SCREEN_BIBLE_V1.md
```

It will define every screen in detail:

``` text
Layout
Navigation
Components
Fields
Actions
States
Loading
Empty
Error
Permissions
Responsive behavior
Mobile mechanic UX
Desktop admin UX
Table columns
Modal behavior
Form validation
Toast messages
Confirmation dialogs
```

Then:

``` text
GARAGE_PRO_DESIGN_SYSTEM_V1.md
```

followed by actual implementation.

------------------------------------------------------------------------

# END OF DOCUMENT

**GARAGE PRO --- API SPECIFICATION V1**\
**Status: Development Ready**
