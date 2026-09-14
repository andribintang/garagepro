# GARAGE PRO --- WORK ORDER ENGINE V1

**Project:** GARAGE PRO --- Workshop Management System\
**Document:** Work Order Core Transaction Engine\
**Version:** V1.0\
**Status:** Implementation Ready\
**Backend:** Node.js + Express + TypeScript\
**Frontend:** React + Vite + TypeScript\
**Database:** MySQL 8+ / Sequelize\
**Validation:** Zod

------------------------------------------------------------------------

# 1. Tujuan

Work Order (WO) adalah **core transaction engine** GARAGE PRO.

Work Order menghubungkan:

``` text
Customer
   ↓
Vehicle
   ↓
Complaint
   ↓
Inspection
   ↓
Diagnosis
   ↓
Service
   ↓
Spare Part
   ↓
Recommendation
   ↓
Estimate
   ↓
Customer Approval
   ↓
Work Execution
   ↓
QC
   ↓
Ready
   ↓
Invoice
```

Semua transaksi utama bengkel bermuara pada Work Order.

Prinsip:

> Satu kendaraan dapat memiliki banyak Work Order. Satu Work Order
> merepresentasikan satu kunjungan/perbaikan utama dan memiliki histori
> yang immutable setelah transaksi masuk tahap final.

------------------------------------------------------------------------

# 2. Work Order Lifecycle

Canonical state machine:

``` text
NEW
 ↓
CHECKING
 ↓
ESTIMATE
 ↓
WAITING_APPROVAL
 ↓
APPROVED
 ↓
IN_PROGRESS
 ↓
QC
 ├── REWORK ──→ IN_PROGRESS
 ↓
READY
 ↓
INVOICED
 ↓
PAID
 ↓
COMPLETED
```

Alternative:

``` text
NEW → CANCELLED
CHECKING → REJECTED
ESTIMATE → CANCELLED
WAITING_APPROVAL → REJECTED
APPROVED → CANCELLED
IN_PROGRESS → CANCELLED
```

Final states:

``` text
COMPLETED
CANCELLED
REJECTED
```

------------------------------------------------------------------------

# 3. State Transition Matrix

  From               To                 Allowed
  ------------------ ------------------ ------------
  NEW                CHECKING           YES
  NEW                CANCELLED          YES
  CHECKING           ESTIMATE           YES
  CHECKING           REJECTED           YES
  CHECKING           CANCELLED          YES
  ESTIMATE           WAITING_APPROVAL   YES
  ESTIMATE           CANCELLED          YES
  WAITING_APPROVAL   APPROVED           YES
  WAITING_APPROVAL   REJECTED           YES
  APPROVED           IN_PROGRESS        YES
  APPROVED           CANCELLED          YES
  IN_PROGRESS        QC                 YES
  IN_PROGRESS        CANCELLED          controlled
  QC                 READY              YES
  QC                 REWORK             YES
  REWORK             IN_PROGRESS        YES
  READY              INVOICED           YES
  INVOICED           PAID               YES
  PAID               COMPLETED          YES

Semua transition harus melewati:

``` text
transitionWorkOrder()
```

Jangan mengubah status langsung dari controller.

------------------------------------------------------------------------

# 4. Work Order Number

Format:

``` text
WO-YYYY-XXXXXX
```

Contoh:

``` text
WO-2026-000001
```

Generated server-side.

Rules:

``` text
unique
immutable
never reused
```

Jika WO dibatalkan:

``` text
WO number tetap reserved
```

------------------------------------------------------------------------

# 5. Work Order Core Fields

``` text
id
wo_number
customer_id
vehicle_id
mechanic_id
status
complaint
diagnosis
estimated_total
approved_total
final_total
discount
tax
notes
opened_at
approved_at
started_at
qc_at
ready_at
completed_at
cancelled_at
created_by
updated_by
created_at
updated_at
```

------------------------------------------------------------------------

# 6. Required Fields

Saat create:

``` text
customer_id
vehicle_id
complaint
```

Mechanic:

``` text
optional saat create
required sebelum execution
```

Rules:

``` text
customer must exist
customer must be active
vehicle must exist
vehicle must belong to customer
```

------------------------------------------------------------------------

# 7. Vehicle Ownership Validation

Critical rule:

``` text
WO.customer_id == vehicle.customer_id
```

Jika tidak:

``` text
VEHICLE_CUSTOMER_MISMATCH
```

Frontend tidak boleh menjadi satu-satunya validator.

------------------------------------------------------------------------

# 8. Create Work Order Flow

``` text
POST /work-orders
        ↓
validate customer
        ↓
validate vehicle
        ↓
validate ownership
        ↓
generate WO number
        ↓
create WO
        ↓
status = NEW
        ↓
audit
```

Response:

``` json
{
  "success": true,
  "data": {
    "id": 1,
    "woNumber": "WO-2026-000001",
    "status": "NEW"
  }
}
```

------------------------------------------------------------------------

# 9. Work Order Create API

``` http
POST /api/v1/work-orders
```

Permission:

``` text
work_orders.create
```

Body:

``` json
{
  "customerId": 1,
  "vehicleId": 10,
  "mechanicId": 3,
  "complaint": "Mesin terasa bergetar saat langsam",
  "notes": "Pelanggan meminta pengecekan menyeluruh"
}
```

------------------------------------------------------------------------

# 10. WO Detail Structure

Detail harus memiliki tabs/sections:

``` text
Overview
Inspection
Services
Spare Parts
Recommendations
Estimate
Timeline
Notes
```

Header:

``` text
WO Number
Status
Customer
Vehicle
Mechanic
Created Date
```

Primary actions berubah berdasarkan status.

------------------------------------------------------------------------

# 11. Status-Based Actions

## NEW

``` text
Mulai Pemeriksaan
Edit
Batalkan
```

## CHECKING

``` text
Simpan Inspection
Lanjut Estimasi
Reject
```

## ESTIMATE

``` text
Tambah Service
Tambah Part
Tambah Recommendation
Kirim Approval
```

## WAITING_APPROVAL

``` text
Edit Estimate
Catat Approval
Reject
```

## APPROVED

``` text
Mulai Pengerjaan
```

## IN_PROGRESS

``` text
Tambah pekerjaan tambahan
QC
```

## QC

``` text
Lulus QC
Rework
```

## READY

``` text
Buat Invoice
```

## INVOICED

``` text
Lihat Invoice
Catat Payment
```

## PAID

``` text
Selesaikan WO
```

------------------------------------------------------------------------

# 12. Inspection

Inspection digunakan untuk mencatat kondisi kendaraan.

Data:

``` text
inspection_item
result
severity
note
```

Result:

``` text
GOOD
CHECK
REPLACE
NOT_APPLICABLE
```

Severity:

``` text
INFO
LOW
MEDIUM
HIGH
CRITICAL
```

------------------------------------------------------------------------

# 13. Inspection API

``` http
POST /api/v1/work-orders/:id/inspection
```

Permission:

``` text
work_orders.update
```

Body:

``` json
{
  "items": [
    {
      "inspectionItemId": 1,
      "result": "GOOD",
      "severity": "INFO",
      "note": "Normal"
    },
    {
      "inspectionItemId": 2,
      "result": "REPLACE",
      "severity": "HIGH",
      "note": "Kampas rem tipis"
    }
  ]
}
```

------------------------------------------------------------------------

# 14. Inspection Business Rules

Inspection hanya boleh diubah pada:

``` text
NEW
CHECKING
ESTIMATE
```

Setelah:

``` text
APPROVED
```

inspection normalnya locked.

Jika perlu koreksi setelah approval:

``` text
controlled amendment
```

dan harus diaudit.

------------------------------------------------------------------------

# 15. Diagnosis

Diagnosis disimpan di:

``` text
work_orders.diagnosis
```

Diagnosis dapat diperbarui selama:

``` text
CHECKING
ESTIMATE
IN_PROGRESS
```

Setelah QC:

``` text
locked
```

kecuali controlled correction.

------------------------------------------------------------------------

# 16. Service Line

Service ditambahkan ke WO:

``` text
service_id
service_code_snapshot
service_name_snapshot
qty
unit_price
discount
subtotal
mechanic_id
notes
```

Calculation:

``` text
subtotal =
(qty × unit_price) - discount
```

Server calculated.

------------------------------------------------------------------------

# 17. Add Service API

``` http
POST /api/v1/work-orders/:id/services
```

Permission:

``` text
work_orders.update
```

Body:

``` json
{
  "serviceId": 10,
  "qty": 1,
  "discount": 0,
  "mechanicId": 3,
  "notes": "Service rutin"
}
```

Backend:

``` text
load service
↓
validate active
↓
snapshot code/name/price
↓
calculate subtotal
↓
create WO service
↓
recalculate estimate
```

------------------------------------------------------------------------

# 18. Service Price Snapshot

Ketika service ditambahkan:

``` text
services.price
       ↓
work_order_services.unit_price
```

Setelah itu perubahan master price:

``` text
DOES NOT
↓
change existing WO
```

------------------------------------------------------------------------

# 19. Spare Part Line

Fields:

``` text
spare_part_id
warehouse_id
sku_snapshot
part_number_snapshot
part_name_snapshot
qty
issued_qty
returned_qty
unit_cost
unit_price
discount
subtotal
issue_status
```

Issue status:

``` text
PENDING
PARTIAL
ISSUED
RETURNED
```

------------------------------------------------------------------------

# 20. Add Part API

``` http
POST /api/v1/work-orders/:id/parts
```

Permission:

``` text
work_orders.update
```

Body:

``` json
{
  "sparePartId": 25,
  "warehouseId": 1,
  "qty": 1,
  "discount": 0
}
```

Important:

> Menambahkan part ke estimate **tidak mengurangi stock**.

------------------------------------------------------------------------

# 21. Part Price Snapshot

Saat part ditambahkan:

``` text
master selling_price
        ↓
WO unit_price
```

Cost snapshot:

``` text
purchase/current cost
        ↓
WO unit_cost
```

Historis tidak berubah karena master data berubah.

------------------------------------------------------------------------

# 22. Part Issue

Stock hanya berkurang saat:

``` text
PART ISSUE
```

Endpoint:

``` http
POST /api/v1/work-orders/:id/parts/:itemId/issue
```

Permission:

``` text
inventory.issue
```

Flow:

``` text
validate WO
↓
validate status
↓
validate part
↓
validate warehouse
↓
check available stock
↓
BEGIN TRANSACTION
↓
lock relevant stock
↓
create WORK_ORDER_ISSUE movement
↓
update issued_qty
↓
commit
```

------------------------------------------------------------------------

# 23. No Negative Stock

Default:

``` text
inventory.allow_negative_stock = false
```

Jika stock:

``` text
available = 0
request = 1
```

reject:

``` text
INSUFFICIENT_STOCK
```

Jangan membiarkan frontend menentukan stock availability.

------------------------------------------------------------------------

# 24. Part Return

Endpoint:

``` http
POST /api/v1/work-orders/:id/parts/:itemId/return
```

Rules:

``` text
return_qty <= issued_qty - returned_qty
```

Flow:

``` text
validate
↓
BEGIN
↓
create WORK_ORDER_RETURN
↓
update returned_qty
↓
commit
```

------------------------------------------------------------------------

# 25. Recommendation Engine

Recommendation adalah pekerjaan tambahan yang ditemukan saat
inspection/repair.

Types:

``` text
SERVICE
PART
GENERAL
```

Status:

``` text
PENDING
APPROVED
DECLINED
CONVERTED
```

------------------------------------------------------------------------

# 26. Recommendation API

Create:

``` http
POST /api/v1/work-orders/:id/recommendations
```

Approve:

``` http
POST /api/v1/work-orders/:id/recommendations/:recommendationId/approve
```

Decline:

``` http
POST /api/v1/work-orders/:id/recommendations/:recommendationId/decline
```

Convert:

``` http
POST /api/v1/work-orders/:id/recommendations/:recommendationId/convert
```

------------------------------------------------------------------------

# 27. Additional Work Approval

Flow:

``` text
Mechanic finds additional work
        ↓
Recommendation
        ↓
Customer approval required
        ↓
APPROVED
        ↓
Convert to service/part
        ↓
Estimate updated
```

Tidak boleh langsung menambahkan pekerjaan berbayar tambahan tanpa
approval jika business policy mewajibkan approval.

------------------------------------------------------------------------

# 28. Estimate Calculation

Formula:

``` text
service subtotal
+
part subtotal
=
subtotal

subtotal
-
discount
+
tax
=
estimated total
```

Namun hindari double-discount.

Recommended canonical:

``` text
services subtotal
+
parts subtotal
=
gross subtotal

gross subtotal
-
document discount
+
tax
=
total
```

Line discount tetap dihitung di masing-masing line.

------------------------------------------------------------------------

# 29. Total Calculation Service

Buat service:

``` text
workOrderCalculationService
```

Responsibilities:

``` text
calculateServiceSubtotal()
calculatePartSubtotal()
calculateGrossSubtotal()
calculateDiscount()
calculateTax()
calculateTotal()
```

Semua total backend.

------------------------------------------------------------------------

# 30. Estimate Update

Estimate berubah ketika:

``` text
service added
service updated
service removed
part added
part updated
part removed
recommendation converted
discount changed
tax configuration changed
```

Setelah perubahan:

``` text
recalculateWorkOrderTotals()
```

------------------------------------------------------------------------

# 31. Approval

Approval endpoint:

``` http
POST /api/v1/work-orders/:id/approve
```

Permission:

``` text
work_orders.approve
```

Preconditions:

``` text
status = WAITING_APPROVAL
estimate has at least one chargeable item
```

Result:

``` text
status = APPROVED
approved_total = current calculated total
approved_at = now
```

------------------------------------------------------------------------

# 32. Approved Total

`approved_total` adalah snapshot nilai yang disetujui customer.

Jika additional work muncul:

``` text
approved_total tetap
```

sampai customer menyetujui additional work.

Jangan overwrite historical approval tanpa audit.

------------------------------------------------------------------------

# 33. Start Work

Endpoint:

``` http
POST /api/v1/work-orders/:id/start
```

Permission:

``` text
work_orders.start
```

Preconditions:

``` text
status = APPROVED
mechanic assigned
```

Result:

``` text
IN_PROGRESS
started_at = now
```

------------------------------------------------------------------------

# 34. Work Execution

Pada `IN_PROGRESS`:

mechanic dapat:

``` text
update diagnosis
add/update permitted service
request additional work
issue parts
record notes
```

Stock issue tetap membutuhkan:

``` text
inventory.issue
```

------------------------------------------------------------------------

# 35. Remove Service/Part Rules

Before approval:

``` text
free edit
```

After approval:

``` text
controlled amendment
```

After invoice:

``` text
LOCKED
```

After payment:

``` text
FULLY LOCKED
```

------------------------------------------------------------------------

# 36. QC

QC endpoint:

``` http
POST /api/v1/work-orders/:id/qc
```

Permission:

``` text
work_orders.qc
```

Payload:

``` json
{
  "result": "PASS",
  "notes": "Test ride normal, tidak ada suara abnormal"
}
```

Result:

``` text
PASS
FAIL
```

------------------------------------------------------------------------

# 37. QC Pass

Preconditions:

``` text
status = QC
```

QC pass:

``` text
status = READY
qc_at = now
ready_at = now
```

------------------------------------------------------------------------

# 38. QC Fail

QC fail:

``` text
status = REWORK
```

Required:

``` text
qc notes
```

Then:

``` text
REWORK
 ↓
IN_PROGRESS
 ↓
QC
```

Rework must be auditable.

------------------------------------------------------------------------

# 39. Rework API

``` http
POST /api/v1/work-orders/:id/rework
```

Permission:

``` text
work_orders.rework
```

Body:

``` json
{
  "reason": "Rem depan masih berbunyi"
}
```

------------------------------------------------------------------------

# 40. Ready State

Ready berarti:

``` text
work completed
QC passed
vehicle ready for customer
```

Ready bukan berarti invoice sudah paid.

Possible flow:

``` text
READY
 ↓
INVOICED
 ↓
PARTIAL
 ↓
PAID
 ↓
COMPLETED
```

Invoice/payment lifecycle tetap terpisah.

------------------------------------------------------------------------

# 41. Invoice Handoff

Endpoint:

``` http
POST /api/v1/work-orders/:id/invoice
```

Permission:

``` text
work_orders.invoice
```

Precondition:

``` text
status = READY
```

Creates:

``` text
invoice
```

Then:

``` text
WO status = INVOICED
```

Invoice generation harus idempotent.

Tidak boleh membuat dua invoice utama untuk satu WO.

------------------------------------------------------------------------

# 42. Completion

WO dapat completed setelah:

``` text
invoice fully paid
```

Endpoint:

``` http
POST /api/v1/work-orders/:id/complete
```

Permission:

``` text
work_orders.complete
```

Precondition:

``` text
status = PAID
```

Result:

``` text
COMPLETED
completed_at = now
```

------------------------------------------------------------------------

# 43. Cancellation

Cancellation harus controlled.

Endpoint:

``` http
POST /api/v1/work-orders/:id/cancel
```

Permission:

``` text
work_orders.cancel
```

Body:

``` json
{
  "reason": "Customer membatalkan pekerjaan"
}
```

Rules:

-   cannot cancel completed;
-   cannot silently remove invoice;
-   issued parts harus direview;
-   payment harus direview;
-   audit wajib.

------------------------------------------------------------------------

# 44. Reject

Reject terutama digunakan sebelum execution.

Example:

``` text
vehicle issue not feasible
customer declines repair
duplicate WO
```

Endpoint dapat:

``` http
POST /api/v1/work-orders/:id/reject
```

Jika belum ada pada API baseline, implementasikan sebagai controlled
transition endpoint.

------------------------------------------------------------------------

# 45. Work Order Timeline

Timeline events:

``` text
WO_CREATED
INSPECTION_UPDATED
SERVICE_ADDED
SERVICE_UPDATED
PART_ADDED
PART_ISSUED
PART_RETURNED
RECOMMENDATION_CREATED
RECOMMENDATION_APPROVED
RECOMMENDATION_DECLINED
ESTIMATE_UPDATED
CUSTOMER_APPROVED
WORK_STARTED
QC_PASSED
QC_FAILED
REWORK
READY
INVOICE_CREATED
PAYMENT_RECEIVED
COMPLETED
CANCELLED
REJECTED
```

Timeline dapat dibangun dari audit/domain events.

------------------------------------------------------------------------

# 46. Audit Requirements

Audit untuk:

``` text
status changes
price changes
discount changes
approval
additional work
part issue
part return
QC
invoice creation
cancellation
completion
```

Audit actor:

``` text
user_id
```

------------------------------------------------------------------------

# 47. Work Order Permission Matrix

  Action          OWNER   ADMIN   MECHANIC   WAREHOUSE
  ------------- ------- ------- ---------- -----------
  View                ✓       ✓          ✓           ✓
  Create              ✓       ✓          ✓          \-
  Update              ✓       ✓          ✓          \-
  Approve             ✓       ✓         \-          \-
  Start               ✓       ✓          ✓          \-
  QC                  ✓       ✓          ✓          \-
  Rework              ✓       ✓          ✓          \-
  Ready               ✓       ✓          ✓          \-
  Invoice             ✓       ✓         \-          \-
  Complete            ✓       ✓         \-          \-
  Cancel              ✓       ✓         \-          \-
  Issue Part          ✓      \-         \-           ✓
  Return Part         ✓      \-         \-           ✓

Actual access harus mengikuti permission, bukan hard-coded role checks.

------------------------------------------------------------------------

# 48. Mechanic Scope

Jika mechanic login:

``` text
mechanic hanya melihat WO yang ditugaskan kepadanya
```

Default query:

``` text
WHERE mechanic_id = currentUser.mechanicId
```

OWNER/ADMIN:

``` text
full workshop scope
```

WAREHOUSE:

``` text
WO scope sesuai kebutuhan inventory
```

------------------------------------------------------------------------

# 49. Work Order Search

Support:

``` text
wo_number
customer name
phone
plate number
mechanic
status
date
```

Example:

``` text
GET /api/v1/work-orders?search=B1234XYZ
```

Filters:

``` text
status
mechanicId
customerId
vehicleId
dateFrom
dateTo
```

------------------------------------------------------------------------

# 50. Work Order List

Desktop columns:

``` text
WO
Date
Customer
Vehicle
Mechanic
Status
Total
Last Updated
Action
```

Mobile:

``` text
WO Number
Vehicle
Customer
Status
Total
Last Update
```

Status badge harus jelas.

------------------------------------------------------------------------

# 51. Workshop Board

Kanban:

``` text
CHECKING
ESTIMATE
WAITING_APPROVAL
APPROVED
IN_PROGRESS
QC
REWORK
READY
```

Card:

``` text
WO Number
Plate
Vehicle
Customer
Mechanic
Elapsed Time
Status
```

Actions berdasarkan permission.

------------------------------------------------------------------------

# 52. Mechanic Mobile Home

Mobile screen:

``` text
My Jobs Today

[ WO-000123 ]
B 1234 XYZ
Honda Beat
Keluhan: Mesin bergetar
Status: IN PROGRESS

[ Buka WO ]
```

Summary:

``` text
Assigned
In Progress
QC
Completed Today
```

------------------------------------------------------------------------

# 53. Work Order Mobile UX

Bottom actions:

``` text
[Inspection]
[Service]
[Parts]
[Notes]
```

Primary action sticky:

``` text
Mulai Pengerjaan
```

atau:

``` text
Submit QC
```

sesuai state.

------------------------------------------------------------------------

# 54. Estimate Screen

Layout:

``` text
Customer
Vehicle

Complaint
Diagnosis

Services
----------------
Service A     Rp xxx
Service B     Rp xxx

Spare Parts
----------------
Part A        Rp xxx
Part B        Rp xxx

Subtotal
Discount
Tax
Grand Total

[ Kirim Approval ]
```

------------------------------------------------------------------------

# 55. Customer Approval UX

Status:

``` text
MENUNGGU PERSETUJUAN
```

Show:

``` text
Total Estimasi
Service
Parts
Recommendations
```

Internal staff dapat mencatat:

``` text
APPROVED
REJECTED
```

Future:

``` text
customer digital approval
OTP / signature / link
```

------------------------------------------------------------------------

# 56. Additional Work UX

Saat mechanic menemukan pekerjaan tambahan:

``` text
+ Rekomendasi Tambahan
```

Form:

``` text
Type
Service/Part
Description
Estimated Price
Reason
```

Status:

``` text
Menunggu Persetujuan
```

Setelah approved:

``` text
Convert to WO line
```

------------------------------------------------------------------------

# 57. Work Order Calculation Example

Services:

``` text
Periodic Service     100,000
Brake Service         50,000
```

Parts:

``` text
Engine Oil            70,000
Brake Pad             80,000
```

Gross:

``` text
300,000
```

Discount:

``` text
10,000
```

Tax:

``` text
0
```

Total:

``` text
290,000
```

Server must produce this result.

------------------------------------------------------------------------

# 58. Work Order Transaction Boundaries

## Create WO

``` text
BEGIN
generate number
create WO
audit
COMMIT
```

## Add Service

``` text
BEGIN
validate
snapshot
insert line
recalculate
audit
COMMIT
```

## Add Part

``` text
BEGIN
validate
snapshot
insert line
recalculate
audit
COMMIT
```

## Issue Part

``` text
BEGIN
lock stock
validate availability
create movement
update issue qty
audit
COMMIT
```

## Approve

``` text
BEGIN
lock WO
recalculate
snapshot approved total
transition
audit
COMMIT
```

------------------------------------------------------------------------

# 59. Concurrency

Critical resources:

``` text
WO
Inventory
Invoice
Payment
```

Use:

``` text
transaction
+
row locking
+
unique constraints
+
idempotency
```

Example:

Two warehouse users issue same last part.

Only one should succeed if:

``` text
stock = 1
request A = 1
request B = 1
```

Second:

``` text
INSUFFICIENT_STOCK
```

------------------------------------------------------------------------

# 60. Idempotency

Critical POST endpoints should support:

``` text
Idempotency-Key
```

Especially:

``` text
create WO
issue part
create invoice
record payment
```

At minimum payment and inventory mutation must prevent accidental
duplicate submission.

------------------------------------------------------------------------

# 61. API Endpoint Summary

``` text
GET    /work-orders
POST   /work-orders
GET    /work-orders/:id
PUT    /work-orders/:id
```

Workflow:

``` text
POST /work-orders/:id/inspection
POST /work-orders/:id/services
PUT  /work-orders/:id/services/:itemId
DELETE /work-orders/:id/services/:itemId

POST /work-orders/:id/parts
PUT  /work-orders/:id/parts/:itemId
DELETE /work-orders/:id/parts/:itemId

POST /work-orders/:id/parts/:itemId/issue
POST /work-orders/:id/parts/:itemId/return

POST /work-orders/:id/recommendations
POST /work-orders/:id/recommendations/:recommendationId/approve
POST /work-orders/:id/recommendations/:recommendationId/decline
POST /work-orders/:id/recommendations/:recommendationId/convert

POST /work-orders/:id/approve
POST /work-orders/:id/start
POST /work-orders/:id/qc
POST /work-orders/:id/rework
POST /work-orders/:id/ready
POST /work-orders/:id/invoice
POST /work-orders/:id/complete
POST /work-orders/:id/cancel
POST /work-orders/:id/reject
```

------------------------------------------------------------------------

# 62. Work Order Service Architecture

Recommended:

``` text
work-order.service.ts
work-order-transition.service.ts
work-order-calculation.service.ts
work-order-inspection.service.ts
work-order-service-line.service.ts
work-order-part.service.ts
work-order-recommendation.service.ts
work-order-qc.service.ts
```

Supporting:

``` text
document-number.service.ts
inventory.service.ts
audit.service.ts
invoice.service.ts
```

------------------------------------------------------------------------

# 63. Work Order Repository

Methods:

``` ts
findById()
findByNumber()
findList()
create()
update()
lockById()
```

Line repositories:

``` text
findServices()
findParts()
findRecommendations()
findInspections()
```

Repository tidak menjalankan workflow.

------------------------------------------------------------------------

# 64. Transition Service

Canonical:

``` ts
transitionWorkOrder(
  workOrderId,
  targetStatus,
  actor,
  metadata
)
```

Responsibilities:

``` text
load current status
validate allowed transition
validate preconditions
update timestamps
update status
audit
```

Example:

``` ts
transitionWorkOrder(
  123,
  'IN_PROGRESS',
  actor
);
```

Jika current status:

``` text
WAITING_APPROVAL
```

throw:

``` text
INVALID_WORK_ORDER_TRANSITION
```

------------------------------------------------------------------------

# 65. Transition Preconditions

### CHECKING

``` text
current = NEW
```

### ESTIMATE

``` text
current = CHECKING
```

### WAITING_APPROVAL

``` text
current = ESTIMATE
at least one chargeable line
```

### APPROVED

``` text
current = WAITING_APPROVAL
```

### IN_PROGRESS

``` text
current = APPROVED
mechanic assigned
```

### QC

``` text
current = IN_PROGRESS
```

### READY

``` text
current = QC
QC PASS
```

### INVOICED

``` text
current = READY
invoice created
```

### COMPLETED

``` text
current = PAID
```

------------------------------------------------------------------------

# 66. Business Error Codes

``` text
WORK_ORDER_NOT_FOUND
WORK_ORDER_INVALID_STATUS
INVALID_WORK_ORDER_TRANSITION

VEHICLE_CUSTOMER_MISMATCH
CUSTOMER_INACTIVE
VEHICLE_INACTIVE
MECHANIC_INACTIVE

WORK_ORDER_EMPTY_ESTIMATE
WORK_ORDER_APPROVAL_REQUIRED
WORK_ORDER_NOT_APPROVED
WORK_ORDER_MECHANIC_REQUIRED

SERVICE_NOT_FOUND
SERVICE_INACTIVE

PART_NOT_FOUND
PART_INACTIVE
WAREHOUSE_NOT_FOUND
INSUFFICIENT_STOCK

RECOMMENDATION_NOT_FOUND
RECOMMENDATION_ALREADY_PROCESSED

QC_NOT_READY
QC_FAILED
REWORK_REASON_REQUIRED

INVOICE_ALREADY_EXISTS
WORK_ORDER_ALREADY_COMPLETED

WORK_ORDER_CANCELLATION_NOT_ALLOWED
WORK_ORDER_LOCKED
```

------------------------------------------------------------------------

# 67. Frontend Feature Structure

``` text
frontend/src/features/work-orders/
├── api/
│   └── workOrdersApi.ts
├── components/
│   ├── WorkOrderTable.tsx
│   ├── WorkOrderCard.tsx
│   ├── WorkOrderHeader.tsx
│   ├── WorkOrderStatusBadge.tsx
│   ├── WorkOrderTimeline.tsx
│   ├── InspectionPanel.tsx
│   ├── ServiceLines.tsx
│   ├── PartLines.tsx
│   ├── RecommendationPanel.tsx
│   ├── EstimateSummary.tsx
│   ├── QCPanel.tsx
│   └── TransitionAction.tsx
├── hooks/
│   ├── useWorkOrders.ts
│   └── useWorkOrder.ts
├── pages/
│   ├── WorkOrderListPage.tsx
│   ├── WorkOrderCreatePage.tsx
│   ├── WorkOrderDetailPage.tsx
│   └── WorkshopBoardPage.tsx
├── schemas/
│   └── workOrder.schema.ts
└── types/
    └── workOrder.types.ts
```

------------------------------------------------------------------------

# 68. Work Order Create Screen

Sequence:

``` text
Customer
↓
Vehicle
↓
Mechanic
↓
Complaint
↓
Notes
↓
Create WO
```

After creation:

``` text
redirect /work-orders/:id
```

Status:

``` text
NEW
```

------------------------------------------------------------------------

# 69. Work Order Detail Header

``` text
WO-2026-000001
[IN PROGRESS]

B 1234 XYZ
Honda Beat
Budi Santoso

Mechanic:
Andi

Opened:
14 Sep 2026 09:30

[Action]
```

------------------------------------------------------------------------

# 70. Status Timeline

Visual:

``` text
NEW ✓
  ↓
CHECKING ✓
  ↓
ESTIMATE ✓
  ↓
APPROVED ✓
  ↓
IN_PROGRESS ●
  ↓
QC ○
  ↓
READY ○
  ↓
INVOICED ○
  ↓
PAID ○
  ↓
COMPLETED ○
```

For REWORK:

``` text
QC
 ↓
REWORK
 ↓
IN_PROGRESS
```

------------------------------------------------------------------------

# 71. Loading and Error UX

Every mutation:

``` text
disable submit
show progress
prevent duplicate click
```

Error:

``` text
toast + contextual message
```

For transition conflict:

``` text
"Status WO sudah berubah. Muat ulang data dan coba lagi."
```

For stock:

``` text
"Stok tidak mencukupi untuk jumlah yang diminta."
```

------------------------------------------------------------------------

# 72. Optimistic Update Policy

Do NOT optimistically update:

``` text
inventory issue
invoice
payment
status transitions
```

Prefer:

``` text
server mutation
↓
success
↓
invalidate/refetch
```

Because these are transactional.

------------------------------------------------------------------------

# 73. Service Line Editing

Before approval:

``` text
edit qty
edit discount
remove
```

After approval:

``` text
controlled change
```

After invoice:

``` text
locked
```

------------------------------------------------------------------------

# 74. Part Line Editing

Before issue:

``` text
edit qty
edit price if authorized
remove
```

After issue:

``` text
issued quantity cannot be silently changed
```

Correction:

``` text
return
+
new issue
```

This maintains inventory traceability.

------------------------------------------------------------------------

# 75. Odometer Update

When WO is completed:

``` text
vehicle.odometer
```

may be updated from final service odometer.

Recommended future field:

``` text
work_orders.odometer_in
work_orders.odometer_out
```

If not included in current schema, add before production if odometer
tracking is a core business requirement.

Rule:

``` text
odometer_out >= odometer_in
```

------------------------------------------------------------------------

# 76. Service History

Vehicle service history is derived from:

``` text
work_orders
+
work_order_services
+
work_order_parts
+
invoices
+
payments
```

Do not create a manually maintained duplicate service-history table.

Query:

``` text
vehicle_id
AND work_order.status = COMPLETED
ORDER BY completed_at DESC
```

------------------------------------------------------------------------

# 77. Customer Communication Hook

Future events:

``` text
WO_CREATED
ESTIMATE_READY
APPROVAL_REQUIRED
WORK_STARTED
READY_FOR_PICKUP
INVOICE_CREATED
PAYMENT_RECEIVED
```

V1 can expose event hooks without implementing WhatsApp integration.

------------------------------------------------------------------------

# 78. Work Order Reporting

WO engine must support:

``` text
daily WO
monthly WO
status distribution
mechanic workload
average completion time
average ticket
repeat customer
repeat vehicle
```

Do not calculate reports by loading every WO into frontend.

Use backend aggregate queries.

------------------------------------------------------------------------

# 79. Performance Rules

List endpoint:

``` text
pagination mandatory
```

Detail:

``` text
load related lines efficiently
```

Avoid:

``` text
N+1 queries
```

Use Sequelize:

``` text
include
attributes
separate where useful
```

Do not return unnecessary fields.

------------------------------------------------------------------------

# 80. Security

Every endpoint:

``` text
authenticate
+
authorize
+
validate
```

Critical mutations:

``` text
transaction
+
audit
```

Never trust:

``` text
client-calculated total
client status
client stock
client permission
```

Server recalculates everything critical.

------------------------------------------------------------------------

# 81. Testing Matrix

## Create

``` text
valid create
missing customer
missing vehicle
vehicle belongs to another customer
inactive customer
inactive vehicle
```

## Inspection

``` text
valid inspection
duplicate inspection item
invalid result
locked inspection
```

## Service

``` text
add service
inactive service
price snapshot
total calculation
```

## Part

``` text
add part
inactive part
add without stock issue
issue sufficient stock
issue insufficient stock
return valid quantity
return excessive quantity
```

## Workflow

``` text
valid transition
invalid transition
approval without line
start without mechanic
QC pass
QC fail
rework
invoice handoff
completion
```

## Concurrency

``` text
two simultaneous stock issues
two invoice creation requests
two status transition requests
```

------------------------------------------------------------------------

# 82. Acceptance Criteria

Phase 12 dianggap DONE jika:

-   [ ] WO dapat dibuat.
-   [ ] Customer/vehicle ownership tervalidasi.
-   [ ] WO number unique.
-   [ ] State machine bekerja.
-   [ ] Inspection bekerja.
-   [ ] Diagnosis bekerja.
-   [ ] Service line bekerja.
-   [ ] Part line bekerja.
-   [ ] Price snapshot bekerja.
-   [ ] Estimate calculation server-side.
-   [ ] Recommendation bekerja.
-   [ ] Approval bekerja.
-   [ ] Additional work approval bekerja.
-   [ ] Part issue terintegrasi inventory.
-   [ ] Part return terintegrasi inventory.
-   [ ] QC PASS → READY.
-   [ ] QC FAIL → REWORK.
-   [ ] REWORK → IN_PROGRESS.
-   [ ] READY → Invoice.
-   [ ] Payment dapat menyelesaikan WO melalui invoice lifecycle.
-   [ ] Audit status transitions.
-   [ ] Permission enforcement.
-   [ ] Mechanic scope bekerja.
-   [ ] Mobile WO UX bekerja.
-   [ ] Desktop admin UX bekerja.
-   [ ] Unit tests PASS.
-   [ ] Integration tests PASS.
-   [ ] Concurrency tests critical PASS.

------------------------------------------------------------------------

# 83. Definition of Done

``` text
MASTER DATA
     ↓
AUTH/RBAC
     ↓
CREATE WO
     ↓
INSPECTION
     ↓
ESTIMATE
     ↓
APPROVAL
     ↓
EXECUTION
     ↓
PART ISSUE
     ↓
QC
     ↓
READY
     ↓
INVOICE HANDOFF
     ↓
PAYMENT HANDOFF
     ↓
COMPLETION
     ↓
SERVICE HISTORY
```

Work Order Engine harus terintegrasi dengan:

``` text
Customer
Vehicle
Mechanic
Service
Part
Warehouse
Inventory
Invoice
Payment
Audit
```

------------------------------------------------------------------------

# 84. Claude Code Master Prompt --- WORK ORDER ENGINE

``` text
You are the senior full-stack transaction-engine engineer implementing Phase 12 of GARAGE PRO.

PROJECT:
GARAGE PRO — Workshop Management System V1.

READ FIRST:
- GARAGE_PRO_MASTER_DEVELOPMENT_SPECIFICATION_V1.md
- GARAGE_PRO_DATABASE_DESIGN_V1.md
- GARAGE_PRO_API_SPECIFICATION_V1.md
- GARAGE_PRO_UI_UX_SCREEN_BIBLE_V1.md
- GARAGE_PRO_DESIGN_SYSTEM_V1.md
- GARAGE_PRO_PROJECT_ARCHITECTURE_V1.md
- GARAGE_PRO_BUSINESS_RULES_V1.md
- GARAGE_PRO_PROJECT_SCAFFOLD_V1.md
- GARAGE_PRO_DATABASE_MIGRATION_V1.md
- GARAGE_PRO_AUTH_RBAC_V1.md
- GARAGE_PRO_MASTER_DATA_V1.md
- GARAGE_PRO_WORK_ORDER_ENGINE_V1.md

OBJECTIVE:
Implement the production-quality Work Order core transaction engine.

DO NOT redesign the existing architecture.
DO NOT invent unrelated modules.
DO NOT bypass service-layer business rules.

STACK:
Backend:
- Node.js
- Express
- TypeScript
- Sequelize
- MySQL 8+
- Zod

Frontend:
- React
- Vite
- TypeScript
- Tailwind
- React Router
- TanStack Query
- React Hook Form
- Zod
- Axios
- Lucide

CORE WORKFLOW:

NEW
→ CHECKING
→ ESTIMATE
→ WAITING_APPROVAL
→ APPROVED
→ IN_PROGRESS
→ QC
→ READY
→ INVOICED
→ PAID
→ COMPLETED

Alternative:
NEW/CHECKING/ESTIMATE/WAITING_APPROVAL/APPROVED/IN_PROGRESS
→ CANCELLED

CHECKING/WAITING_APPROVAL
→ REJECTED

QC
→ REWORK
→ IN_PROGRESS

IMPLEMENT:

1. WORK ORDER CRUD
2. WORK ORDER STATE MACHINE
3. INSPECTION
4. DIAGNOSIS
5. SERVICE LINES
6. PART LINES
7. PRICE SNAPSHOTS
8. ESTIMATE CALCULATION
9. CUSTOMER APPROVAL
10. ADDITIONAL WORK RECOMMENDATIONS
11. PART ISSUE
12. PART RETURN
13. QC
14. REWORK
15. READY
16. INVOICE HANDOFF
17. COMPLETION
18. CANCELLATION
19. AUDIT
20. WORK ORDER TIMELINE

CRITICAL RULE:

ALL STATUS CHANGES MUST GO THROUGH:

transitionWorkOrder()

Never directly assign:
workOrder.status = ...

TRANSITION PRECONDITIONS:

CHECKING:
current NEW

ESTIMATE:
current CHECKING

WAITING_APPROVAL:
current ESTIMATE
at least one chargeable line

APPROVED:
current WAITING_APPROVAL

IN_PROGRESS:
current APPROVED
mechanic assigned

QC:
current IN_PROGRESS

READY:
current QC
QC PASS

INVOICED:
current READY
invoice created

COMPLETED:
current PAID

SERVICE:

When adding service:
- load master
- validate active
- snapshot code/name/price
- calculate subtotal
- save line
- recalculate totals

PART:

When adding part:
- validate active
- validate warehouse
- snapshot SKU/name/part number
- snapshot price/cost
- calculate subtotal
- DO NOT reduce stock

STOCK:

Only issue/return changes inventory.

Issue must:
- run in DB transaction
- lock/check stock
- reject insufficient stock
- create WORK_ORDER_ISSUE movement
- update issued quantity
- audit

Return must:
- validate returned quantity
- create WORK_ORDER_RETURN
- update returned quantity
- audit

No stock mutation without ledger movement.

APPROVAL:

On approve:
- recalculate total
- snapshot approved_total
- transition to APPROVED
- audit

ADDITIONAL WORK:

Mechanic creates recommendation.
Customer approval required.
Approved recommendation may be converted to service/part line.
Do not silently add additional billable work without approval.

QC:

PASS:
QC → READY

FAIL:
QC → REWORK

REWORK:
REWORK → IN_PROGRESS

INVOICE:
READY → INVOICED

Do not create duplicate invoice if invoice already exists.

COMPLETION:
Only PAID → COMPLETED.

SECURITY:

Every endpoint:
authenticate
authorize(permission)
validate input

Never trust:
- client status
- client total
- client stock
- client permission

Calculate critical values server-side.

CONCURRENCY:

Use:
- transactions
- row locks
- unique constraints
- idempotency where appropriate

Critical operations:
- stock issue
- invoice creation
- payment handoff
- status transition

MECHANIC SCOPE:

If logged-in user is a mechanic:
only show/modify assigned work orders unless explicit permission policy says otherwise.

FRONTEND:

Implement:
- WO list
- create WO
- detail
- inspection
- services
- parts
- recommendations
- estimate
- QC
- timeline
- workshop board
- mechanic mobile view

Status-based actions must respect:
- current state
- permission
- loading
- server response

Use TanStack Query invalidation after mutations.

Do not optimistically update financial/inventory/status mutations.

RESPONSIVE:

Desktop:
efficient tables and multi-column detail.

Mobile:
cards
large touch targets
sticky primary action
simple workflow

TESTS:

Implement tests for:
- valid/invalid transitions
- ownership mismatch
- inspection
- service snapshot
- part snapshot
- total calculation
- approval
- additional work
- stock issue
- stock return
- insufficient stock
- QC
- rework
- invoice handoff
- completion
- cancellation
- authorization
- concurrency
- idempotency

PROCESS:

1. Inspect repository.
2. Inspect existing master-data implementation.
3. Inspect auth/RBAC.
4. Inspect inventory contracts.
5. Inspect invoice contracts.
6. Implement backend.
7. Implement tests.
8. Implement frontend.
9. Integrate API.
10. Run migrations if required.
11. Run typecheck.
12. Run backend tests.
13. Run frontend build.
14. Run integration tests.

Do not stop at planning.
Actually modify the repository.

FINAL REPORT:

Return:
- files created
- files modified
- APIs implemented
- state transitions implemented
- permissions
- inventory integration
- invoice integration
- frontend screens
- tests
- test results
- known limitations
- next phase
```

------------------------------------------------------------------------

# 85. Recommended Implementation Sequence

Untuk mengurangi risiko, implementasikan Work Order secara bertahap:

``` text
STEP 1
WO CRUD
       ↓
STEP 2
State Machine
       ↓
STEP 3
Inspection
       ↓
STEP 4
Service Lines
       ↓
STEP 5
Part Lines
       ↓
STEP 6
Calculation
       ↓
STEP 7
Approval
       ↓
STEP 8
Additional Work
       ↓
STEP 9
Inventory Issue/Return
       ↓
STEP 10
QC/Rework
       ↓
STEP 11
Invoice Handoff
       ↓
STEP 12
Completion
       ↓
STEP 13
Audit + Timeline
       ↓
STEP 14
Frontend Integration
       ↓
STEP 15
Testing
```

Jangan langsung membangun semua layar sebelum backend state machine
stabil.

------------------------------------------------------------------------

# 86. Next Phase

Setelah Work Order Engine stabil, phase berikutnya:

**GARAGE_PRO_INVENTORY_ENGINE_V1.md**

Cakupan:

``` text
Stock Ledger
↓
Stock Balance
↓
Multi Warehouse
↓
Stock Issue
↓
Stock Return
↓
Purchase Receiving
↓
Stock Adjustment
↓
Stock Opname
↓
Low Stock
↓
Stock Valuation
↓
Inventory Audit
↓
Concurrency
```

Inventory Engine akan menjadi sumber kebenaran stok dan terintegrasi
langsung dengan Work Order serta Purchasing.
