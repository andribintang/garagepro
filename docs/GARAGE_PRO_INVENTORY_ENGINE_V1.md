# GARAGE PRO --- INVENTORY ENGINE V1

**Project:** GARAGE PRO --- Workshop Management System\
**Document:** Inventory & Stock Management Engine\
**Version:** V1.0\
**Status:** Implementation Ready\
**Backend:** Node.js + Express + TypeScript\
**Frontend:** React + Vite + TypeScript\
**Database:** MySQL 8+\
**ORM:** Sequelize\
**Validation:** Zod

------------------------------------------------------------------------

# 1. Tujuan

Inventory Engine adalah sumber kebenaran stok GARAGE PRO.

Arsitektur:

``` text
Purchase Receiving
       ↓
   STOCK IN
       ↓
Stock Ledger
       ↓
Stock Balance
       ↓
      ├── Work Order Issue
      ├── Work Order Return
      ├── Adjustment
      ├── Stock Opname
      └── Transfer
```

Prinsip utama:

> Tidak boleh ada perubahan stok tanpa stock movement.

Inventory harus mampu memberikan:

-   stok aktual;
-   stok per warehouse;
-   stok per location;
-   riwayat mutasi;
-   low stock;
-   stock opname;
-   adjustment;
-   receiving;
-   issue ke Work Order;
-   return dari Work Order;
-   stock valuation;
-   audit trail;
-   concurrency protection.

------------------------------------------------------------------------

# 2. Inventory Architecture

``` text
                   ┌─────────────────┐
                   │ Purchase Receipt│
                   └────────┬────────┘
                            ↓
                     ┌─────────────┐
                     │ Stock Ledger│
                     └──────┬──────┘
                            ↓
                  ┌──────────────────┐
                  │ Stock Projection │
                  └───────┬──────────┘
                          ↓
             ┌────────────┼────────────┐
             ↓            ↓            ↓
         Work Order   Adjustment   Stock Opname
           Issue/Return
```

Source of truth:

``` text
stock_movements
```

------------------------------------------------------------------------

# 3. Inventory Terminology

## On Hand

Jumlah stok fisik tercatat.

``` text
ON_HAND
```

## Reserved

V1 optional.

Jika belum menerapkan reservation:

``` text
reserved = 0
```

## Available

``` text
available = on_hand - reserved
```

Untuk V1:

``` text
available = on_hand
```

kecuali reservation feature sudah diaktifkan.

------------------------------------------------------------------------

# 4. Stock Movement Types

Canonical:

``` text
PURCHASE_RECEIPT
WORK_ORDER_ISSUE
WORK_ORDER_RETURN
ADJUSTMENT_IN
ADJUSTMENT_OUT
STOCK_OPNAME_IN
STOCK_OPNAME_OUT
TRANSFER_IN
TRANSFER_OUT
```

------------------------------------------------------------------------

# 5. Movement Direction

IN:

``` text
PURCHASE_RECEIPT
WORK_ORDER_RETURN
ADJUSTMENT_IN
STOCK_OPNAME_IN
TRANSFER_IN
```

OUT:

``` text
WORK_ORDER_ISSUE
ADJUSTMENT_OUT
STOCK_OPNAME_OUT
TRANSFER_OUT
```

Database `quantity` dapat menggunakan signed quantity:

``` text
IN  = positive
OUT = negative
```

Example:

``` text
+10
-2
+1
```

------------------------------------------------------------------------

# 6. Stock Movement Schema

``` text
id
spare_part_id
warehouse_id
warehouse_location_id
movement_type
reference_type
reference_id
quantity
unit_cost
balance_after
notes
created_by
created_at
```

Important:

``` text
NO updated_at
```

karena ledger immutable.

------------------------------------------------------------------------

# 7. Ledger Rules

Setelah movement dibuat:

``` text
cannot edit
cannot update quantity
cannot change reference
cannot delete
```

Jika terjadi kesalahan:

``` text
create reversing movement
```

Contoh:

Salah issue:

``` text
-5
```

Correction:

``` text
+5
```

Jangan mengubah movement lama.

------------------------------------------------------------------------

# 8. Stock Balance

Ada dua pendekatan.

## Option A --- Calculate from Ledger

``` sql
SUM(quantity)
```

Advantages:

``` text
simple
audit-friendly
always reconstructable
```

Disadvantage:

``` text
expensive for high transaction volume
```

## Option B --- Stock Projection

Table:

``` text
stock_balances
```

Recommended untuk production.

Fields:

``` text
id
spare_part_id
warehouse_id
warehouse_location_id
quantity
updated_at
```

Unique:

``` text
spare_part_id
warehouse_id
warehouse_location_id
```

Source of truth tetap:

``` text
stock_movements
```

Projection digunakan untuk performance.

------------------------------------------------------------------------

# 9. Recommended V1 Architecture

``` text
stock_movements
       ↓
stock_balances
```

Every stock mutation:

``` text
BEGIN
lock stock balance
validate
create movement
update balance
COMMIT
```

Jika projection rusak:

``` text
rebuild stock_balances from stock_movements
```

------------------------------------------------------------------------

# 10. Add stock_balances Migration

Recommended additional migration:

``` text
create-stock-balances
```

Fields:

``` text
id BIGINT UNSIGNED PK
spare_part_id BIGINT UNSIGNED NOT NULL
warehouse_id BIGINT UNSIGNED NOT NULL
warehouse_location_id BIGINT UNSIGNED NULL
quantity DECIMAL(15,3) NOT NULL DEFAULT 0
created_at DATETIME NOT NULL
updated_at DATETIME NOT NULL
```

Constraints:

``` text
quantity >= 0
```

Unique logical key:

``` text
spare_part_id
warehouse_id
warehouse_location_id
```

If MySQL NULL uniqueness behavior creates issues, use a canonical
default location or separate balance model.

------------------------------------------------------------------------

# 11. Stock Balance Key

Preferred:

``` text
part + warehouse
```

for V1 if location is informational.

Then:

``` text
stock_balances
UNIQUE(spare_part_id, warehouse_id)
```

Location remains movement metadata.

If exact bin-level inventory is required:

``` text
part + warehouse + location
```

must be used consistently.

Recommendation for GARAGE PRO V1:

> **Warehouse-level stock balance first; location is used for
> operational placement and movement traceability.**

------------------------------------------------------------------------

# 12. Stock Calculation

For warehouse:

``` text
stock(part, warehouse)
=
SUM(stock movements)
```

Projection:

``` text
stock_balances.quantity
```

Reconciliation:

``` text
ledger stock
==
projection stock
```

Any mismatch:

``` text
INVENTORY_BALANCE_MISMATCH
```

------------------------------------------------------------------------

# 13. Stock Service

Create:

``` text
inventory.service.ts
```

Core methods:

``` ts
getStock()
getAvailableStock()
issueStock()
returnStock()
receiveStock()
adjustStock()
transferStock()
performOpname()
rebuildBalance()
reconcileBalance()
```

------------------------------------------------------------------------

# 14. Inventory Repository

Methods:

``` ts
findBalance()
findBalanceForUpdate()
createBalance()
updateBalance()
createMovement()
findMovements()
findMovementByReference()
```

`findBalanceForUpdate()` harus menggunakan row locking dalam
transaction.

------------------------------------------------------------------------

# 15. Issue Stock

Endpoint:

``` http
POST /api/v1/inventory/issue
```

Alternative integrated endpoint:

``` http
POST /api/v1/work-orders/:id/parts/:itemId/issue
```

Recommended:

> Work Order endpoint memanggil Inventory Service, bukan membuat stock
> movement sendiri.

------------------------------------------------------------------------

# 16. Issue Stock Request

``` json
{
  "sparePartId": 10,
  "warehouseId": 1,
  "warehouseLocationId": 2,
  "quantity": 1,
  "referenceType": "WORK_ORDER",
  "referenceId": 123
}
```

------------------------------------------------------------------------

# 17. Issue Stock Flow

``` text
request
 ↓
validate part
 ↓
validate warehouse
 ↓
validate quantity
 ↓
BEGIN
 ↓
lock stock balance
 ↓
read current quantity
 ↓
check available
 ↓
create movement -quantity
 ↓
update balance
 ↓
audit
 ↓
COMMIT
```

------------------------------------------------------------------------

# 18. Insufficient Stock

Example:

``` text
stock = 2
request = 3
```

Reject:

``` text
INSUFFICIENT_STOCK
```

Response:

``` json
{
  "success": false,
  "error": {
    "code": "INSUFFICIENT_STOCK",
    "message": "Stok tidak mencukupi."
  }
}
```

------------------------------------------------------------------------

# 19. Negative Stock

Setting:

``` text
inventory.allow_negative_stock = false
```

Default.

If explicitly enabled in future:

``` text
allow negative
```

must be audited.

Do not silently allow negative stock.

------------------------------------------------------------------------

# 20. Return Stock

Endpoint:

``` http
POST /api/v1/inventory/return
```

Payload:

``` json
{
  "sparePartId": 10,
  "warehouseId": 1,
  "warehouseLocationId": 2,
  "quantity": 1,
  "referenceType": "WORK_ORDER",
  "referenceId": 123
}
```

Movement:

``` text
WORK_ORDER_RETURN
+quantity
```

------------------------------------------------------------------------

# 21. Return Validation

For Work Order return:

``` text
return_qty <= issued_qty - returned_qty
```

Reject:

``` text
INVALID_RETURN_QUANTITY
```

Inventory service should not trust frontend-issued quantity.

------------------------------------------------------------------------

# 22. Purchase Receiving

Receiving creates:

``` text
PURCHASE_RECEIPT
```

Movement:

``` text
+quantity
```

Flow:

``` text
Purchase
 ↓
Receiving
 ↓
Validate item
 ↓
BEGIN
 ↓
lock stock balance
 ↓
create receipt movement
 ↓
update balance
 ↓
update received_qty
 ↓
update purchase status
 ↓
COMMIT
```

------------------------------------------------------------------------

# 23. Purchase Receiving Status

Purchase:

``` text
DRAFT
ORDERED
PARTIAL_RECEIVED
RECEIVED
CANCELLED
```

Rules:

``` text
received_qty = 0
→ ORDERED

0 < received_qty < ordered_qty
→ PARTIAL_RECEIVED

received_qty = ordered_qty
→ RECEIVED
```

------------------------------------------------------------------------

# 24. Adjustment

Adjustment is controlled stock correction.

Types:

``` text
ADJUSTMENT_IN
ADJUSTMENT_OUT
```

Permission:

``` text
inventory.adjust
```

Required:

``` text
part
warehouse
quantity
reason
```

Example:

``` json
{
  "sparePartId": 10,
  "warehouseId": 1,
  "quantity": 2,
  "direction": "OUT",
  "reason": "Barang rusak"
}
```

------------------------------------------------------------------------

# 25. Adjustment Rules

Adjustment OUT:

``` text
stock must be sufficient
```

Adjustment IN:

``` text
always allowed if authorized
```

Reason required:

``` text
minimum 5 characters
```

Audit:

``` text
actor
reason
before
after
```

------------------------------------------------------------------------

# 26. Stock Opname

Purpose:

``` text
compare physical stock
vs
system stock
```

Flow:

``` text
Create Opname
 ↓
COUNTING
 ↓
Input Count
 ↓
REVIEW
 ↓
Approve
 ↓
Generate Adjustment Movements
 ↓
COMPLETED
```

------------------------------------------------------------------------

# 27. Opname Creation

Endpoint:

``` http
POST /api/v1/inventory/opnames
```

Payload:

``` json
{
  "warehouseId": 1,
  "notes": "Opname bulanan September 2026"
}
```

Creates:

``` text
stock_opnames
```

Status:

``` text
DRAFT
```

------------------------------------------------------------------------

# 28. Start Counting

``` http
POST /api/v1/inventory/opnames/:id/start
```

Status:

``` text
COUNTING
```

At this point system captures:

``` text
system_qty
```

Recommended snapshot:

``` text
system_qty = current stock at count start
```

------------------------------------------------------------------------

# 29. Opname Item

Fields:

``` text
stock_opname_id
spare_part_id
system_qty
counted_qty
difference_qty
unit_cost
notes
```

Formula:

``` text
difference_qty =
counted_qty - system_qty
```

Example:

``` text
system = 10
physical = 8
difference = -2
```

------------------------------------------------------------------------

# 30. Opname Completion

If:

``` text
difference = 0
```

No movement required.

If:

``` text
difference > 0
```

create:

``` text
STOCK_OPNAME_IN
```

If:

``` text
difference < 0
```

create:

``` text
STOCK_OPNAME_OUT
```

All adjustments must occur inside one transaction per controlled
completion.

------------------------------------------------------------------------

# 31. Opname Concurrency

Important problem:

``` text
Opname starts
↓
stock moves during counting
↓
physical count
```

Recommended V1 policy:

> `system_qty` is a snapshot at counting start, and completion must
> calculate/handle movements according to an explicit stock-opname
> policy.

Simpler V1 policy:

``` text
freeze warehouse movement during finalization
```

or:

``` text
re-read current balance
apply controlled difference
```

Recommended:

``` text
finalization locks relevant stock rows
reconciles current system quantity
requires recount if movement conflict detected
```

------------------------------------------------------------------------

# 32. Low Stock

Low stock condition:

``` text
current stock <= minimum_stock
```

Example:

``` text
stock = 2
minimum_stock = 3
→ LOW STOCK
```

Critical:

``` text
stock = 0
→ OUT OF STOCK
```

------------------------------------------------------------------------

# 33. Low Stock API

``` http
GET /api/v1/inventory/low-stock
```

Query:

``` text
warehouseId
categoryId
search
page
pageSize
```

Response:

``` json
{
  "success": true,
  "data": [
    {
      "sku": "OIL-001",
      "name": "Engine Oil",
      "stock": 2,
      "minimumStock": 3,
      "status": "LOW_STOCK"
    }
  ]
}
```

------------------------------------------------------------------------

# 34. Stock Overview API

``` http
GET /api/v1/inventory/stock
```

Filters:

``` text
warehouseId
partId
categoryId
search
lowStock
outOfStock
```

Columns:

``` text
SKU
Part
Warehouse
Stock
Minimum
Maximum
Status
```

------------------------------------------------------------------------

# 35. Stock Movement API

``` http
GET /api/v1/inventory/movements
```

Filters:

``` text
partId
warehouseId
movementType
referenceType
dateFrom
dateTo
```

Columns:

``` text
Date
Part
Warehouse
Type
Qty
Balance After
Reference
Actor
```

------------------------------------------------------------------------

# 36. Movement Detail

Detail:

``` text
Movement ID
Date
Movement Type
Part
Warehouse
Location
Quantity
Unit Cost
Balance After
Reference
Created By
Notes
```

Movement cannot be edited.

------------------------------------------------------------------------

# 37. Transfer Stock

V1 can support transfer.

Endpoint:

``` http
POST /api/v1/inventory/transfers
```

Payload:

``` json
{
  "sparePartId": 10,
  "fromWarehouseId": 1,
  "fromLocationId": 2,
  "toWarehouseId": 2,
  "toLocationId": 3,
  "quantity": 5,
  "notes": "Transfer ke cabang"
}
```

Flow:

``` text
lock source
 ↓
validate stock
 ↓
create TRANSFER_OUT
 ↓
lock destination
 ↓
create TRANSFER_IN
 ↓
commit
```

Both movements must share:

``` text
referenceType = TRANSFER
referenceId = transfer transaction ID
```

Recommended additional table:

``` text
stock_transfers
stock_transfer_items
```

if multi-part transfer is required.

------------------------------------------------------------------------

# 38. Transfer Atomicity

Never allow:

``` text
TRANSFER_OUT succeeds
TRANSFER_IN fails
```

Both must be in one DB transaction.

``` text
BEGIN
OUT
IN
COMMIT
```

On failure:

``` text
ROLLBACK
```

------------------------------------------------------------------------

# 39. Stock Valuation

V1 recommended method:

``` text
weighted average cost
```

For purchase:

``` text
old_value = old_qty × old_cost
new_value = received_qty × received_cost

new_average_cost =
(old_value + new_value)
/
(old_qty + received_qty)
```

Store current cost projection if required.

------------------------------------------------------------------------

# 40. Cost Snapshot

Work Order Part:

``` text
unit_cost
```

must be snapshot at issue/transaction time according to costing policy.

Do not recalculate historical cost from today's master part cost.

------------------------------------------------------------------------

# 41. Stock Valuation API

``` http
GET /api/v1/reports/stock-valuation
```

Response fields:

``` text
part
warehouse
quantity
averageCost
stockValue
```

Formula:

``` text
stockValue =
quantity × averageCost
```

------------------------------------------------------------------------

# 42. Inventory Audit

Audit events:

``` text
STOCK_RECEIVED
STOCK_ISSUED
STOCK_RETURNED
STOCK_ADJUSTED
STOCK_OPNAME_STARTED
STOCK_OPNAME_COMPLETED
STOCK_TRANSFERRED
STOCK_RECONCILED
```

------------------------------------------------------------------------

# 43. Reconciliation

Endpoint:

``` http
POST /api/v1/inventory/reconcile
```

OWNER/authorized admin only.

Process:

``` text
SUM ledger
vs
stock_balances
```

Output:

``` text
matched
mismatched
difference
```

Do not automatically modify stock just because mismatch exists.

------------------------------------------------------------------------

# 44. Rebuild Stock Balance

Admin operation:

``` http
POST /api/v1/inventory/rebuild-balance
```

Recommended:

``` text
maintenance mode
```

Process:

``` text
clear/recalculate projection
from immutable ledger
```

Audit:

``` text
STOCK_BALANCE_REBUILT
```

------------------------------------------------------------------------

# 45. Inventory Service Transaction Helper

Recommended abstraction:

``` ts
inventoryTransaction(
  callback
)
```

Example:

``` ts
await sequelize.transaction(async (transaction) => {
  const balance =
    await stockBalanceRepository.findForUpdate(
      {
        sparePartId,
        warehouseId,
      },
      { transaction }
    );

  // validate
  // create movement
  // update balance
});
```

------------------------------------------------------------------------

# 46. Row Locking

For stock mutation:

``` sql
SELECT ...
FROM stock_balances
WHERE spare_part_id = ?
AND warehouse_id = ?
FOR UPDATE
```

Sequelize equivalent:

``` ts
lock: transaction.LOCK.UPDATE
```

Use inside transaction.

------------------------------------------------------------------------

# 47. Race Condition Example

Initial:

``` text
Stock = 1
```

Request A:

``` text
issue 1
```

Request B:

``` text
issue 1
```

Without locking:

``` text
A sees 1
B sees 1
A succeeds
B succeeds
stock = -1
```

With row lock:

``` text
A locks row
B waits
A stock → 0
A commits
B reads 0
B rejected
```

------------------------------------------------------------------------

# 48. Inventory Idempotency

Critical endpoints:

``` text
issue
return
receive
adjust
transfer
opname completion
```

support:

``` text
Idempotency-Key
```

Store:

``` text
request key
operation
user
result/reference
created_at
```

Recommended table:

``` text
idempotency_keys
```

------------------------------------------------------------------------

# 49. Duplicate Issue Protection

Example:

User clicks:

``` text
Issue Part
```

twice.

Server should ensure:

``` text
same idempotency key
→ same result
→ no second movement
```

Do not rely on disabling the button alone.

------------------------------------------------------------------------

# 50. Inventory API Summary

``` text
GET  /inventory/stock
GET  /inventory/movements
GET  /inventory/low-stock

POST /inventory/issue
POST /inventory/return
POST /inventory/receive
POST /inventory/adjust
POST /inventory/transfers

GET  /inventory/opnames
POST /inventory/opnames
GET  /inventory/opnames/:id
POST /inventory/opnames/:id/start
POST /inventory/opnames/:id/review
POST /inventory/opnames/:id/complete

GET  /inventory/reconcile
POST /inventory/reconcile
POST /inventory/rebuild-balance
```

Some operations may remain integrated with Purchase/Work Order
controllers while delegating business logic to InventoryService.

------------------------------------------------------------------------

# 51. Inventory Permission Matrix

  Action            OWNER        ADMIN   MECHANIC   WAREHOUSE
  --------------- ------- ------------ ---------- -----------
  View Stock            ✓            ✓         \-           ✓
  View Movement         ✓            ✓         \-           ✓
  Issue                 ✓           \-         \-           ✓
  Return                ✓           \-         \-           ✓
  Receive               ✓           \-         \-           ✓
  Adjustment            ✓           \-         \-           ✓
  Opname                ✓           \-         \-           ✓
  Transfer              ✓           \-         \-           ✓
  Reconcile             ✓   controlled         \-          \-
  Rebuild               ✓   controlled         \-          \-

Actual access remains permission-based.

------------------------------------------------------------------------

# 52. Warehouse Scope

If user is WAREHOUSE:

``` text
warehouse scope
```

should be determined by assigned warehouse relationship, not simply
frontend selection.

Future field:

``` text
user_warehouses
```

If multi-warehouse user support is required.

V1 can use:

``` text
user.primary_warehouse_id
```

if business requires strict warehouse scoping.

------------------------------------------------------------------------

# 53. Inventory UI

## Stock Overview

``` text
Inventory

[Search Part]
[Warehouse]
[Category]
[Low Stock]

SKU
Part
Warehouse
Stock
Min
Status
Action
```

------------------------------------------------------------------------

# 54. Low Stock Screen

Cards:

``` text
LOW STOCK
24 Items

OUT OF STOCK
7 Items
```

List:

``` text
Part
Current
Minimum
Gap
Warehouse
```

CTA:

``` text
Buat Purchase Order
```

future integration.

------------------------------------------------------------------------

# 55. Movement Screen

Filters:

``` text
Date
Movement Type
Warehouse
Part
Reference
```

Timeline/table.

Movement colors should use semantic design tokens, not arbitrary colors.

------------------------------------------------------------------------

# 56. Issue Part Screen

Mobile-first:

``` text
Part
Warehouse
Available Stock
Quantity

Reference
Notes

[ISSUE STOCK]
```

Show:

``` text
Available: 12
Request: 3
After Issue: 9
```

Server remains authoritative.

------------------------------------------------------------------------

# 57. Receiving Screen

``` text
Purchase Order
Supplier
Warehouse

Items
----------------
Part A
Ordered 10
Received 5
Remaining 5

Receive Qty [5]

[CONFIRM RECEIVING]
```

Cannot receive:

``` text
> remaining
```

unless over-receiving is explicitly enabled.

------------------------------------------------------------------------

# 58. Adjustment Screen

``` text
Part
Warehouse
Direction
Quantity
Reason

Current Stock
After Adjustment

[CONFIRM]
```

Confirmation:

``` text
Anda akan mengubah stok.
Perubahan ini akan tercatat dalam audit.
```

------------------------------------------------------------------------

# 59. Stock Opname Screen

Step 1:

``` text
Warehouse
Notes
[Start Opname]
```

Step 2:

``` text
Part
System Qty
Physical Qty
Difference
```

Step 3:

``` text
Review Differences
```

Step 4:

``` text
[Complete Opname]
```

------------------------------------------------------------------------

# 60. Inventory Dashboard

KPIs:

``` text
Total SKU
Total Stock Value
Low Stock
Out of Stock
Today's Stock In
Today's Stock Out
```

Charts:

``` text
Stock movement trend
Top issued parts
Low stock by category
```

------------------------------------------------------------------------

# 61. Inventory Calculation Example

Initial:

``` text
Stock = 10
```

Purchase:

``` text
+20
```

WO Issue:

``` text
-5
```

WO Return:

``` text
+1
```

Adjustment:

``` text
-2
```

Final:

``` text
10 + 20 - 5 + 1 - 2 = 24
```

Ledger:

``` text
PURCHASE_RECEIPT +20
WORK_ORDER_ISSUE -5
WORK_ORDER_RETURN +1
ADJUSTMENT_OUT -2
```

Balance:

``` text
24
```

------------------------------------------------------------------------

# 62. Inventory Acceptance Test

Scenario:

``` text
Create Part
Create Warehouse
Initial Stock = 0
```

Receive:

``` text
10
```

Expected:

``` text
stock = 10
```

Issue:

``` text
3
```

Expected:

``` text
stock = 7
```

Return:

``` text
1
```

Expected:

``` text
stock = 8
```

Adjustment OUT:

``` text
2
```

Expected:

``` text
stock = 6
```

Opname physical:

``` text
5
```

Expected adjustment:

``` text
-1
```

Final:

``` text
5
```

Ledger sum:

``` text
5
```

Projection:

``` text
5
```

------------------------------------------------------------------------

# 63. Inventory Test Matrix

## Basic

-   [ ] receive
-   [ ] issue
-   [ ] return
-   [ ] adjustment in
-   [ ] adjustment out
-   [ ] transfer
-   [ ] opname

## Validation

-   [ ] invalid part
-   [ ] invalid warehouse
-   [ ] zero quantity
-   [ ] negative quantity
-   [ ] insufficient stock
-   [ ] excessive return
-   [ ] excessive receiving

## Integrity

-   [ ] movement immutable
-   [ ] balance equals ledger
-   [ ] no duplicate movement
-   [ ] audit exists
-   [ ] reference exists

## Concurrency

-   [ ] simultaneous issue
-   [ ] simultaneous adjustment
-   [ ] simultaneous transfer
-   [ ] simultaneous receiving

------------------------------------------------------------------------

# 64. Inventory Error Codes

``` text
INVENTORY_PART_NOT_FOUND
INVENTORY_WAREHOUSE_NOT_FOUND
INVENTORY_LOCATION_NOT_FOUND

INVALID_STOCK_QUANTITY
INSUFFICIENT_STOCK
NEGATIVE_STOCK_NOT_ALLOWED

INVALID_RETURN_QUANTITY
INVALID_RECEIVING_QUANTITY

STOCK_MOVEMENT_NOT_FOUND
STOCK_MOVEMENT_IMMUTABLE

STOCK_OPNAME_NOT_FOUND
STOCK_OPNAME_INVALID_STATUS
STOCK_OPNAME_ALREADY_COMPLETED

STOCK_BALANCE_NOT_FOUND
INVENTORY_BALANCE_MISMATCH

TRANSFER_SOURCE_REQUIRED
TRANSFER_DESTINATION_REQUIRED
TRANSFER_SAME_WAREHOUSE
TRANSFER_INSUFFICIENT_STOCK

INVENTORY_IDEMPOTENCY_CONFLICT
```

------------------------------------------------------------------------

# 65. Performance

Inventory queries harus:

``` text
paginated
indexed
warehouse-filtered
```

Critical indexes:

``` text
stock_balances(spare_part_id, warehouse_id)

stock_movements(spare_part_id, warehouse_id, created_at)

stock_movements(reference_type, reference_id)

stock_movements(movement_type, created_at)
```

Untuk laporan periode:

``` text
created_at
```

harus indexed.

------------------------------------------------------------------------

# 66. Reconciliation Job

Recommended scheduled check:

``` text
daily
```

Process:

``` text
ledger aggregate
vs
stock balance
```

Jika mismatch:

``` text
create alert
```

Do not automatically adjust.

------------------------------------------------------------------------

# 67. Inventory Auditability

Untuk setiap stock change dapat dijawab:

``` text
What?
Part apa?

Where?
Warehouse mana?

How much?
Berapa?

Why?
Movement apa?

Reference?
WO/PO/Opname apa?

Who?
User siapa?

When?
Kapan?
```

Jika salah satu tidak dapat dijawab, inventory traceability belum cukup.

------------------------------------------------------------------------

# 68. Database Addition Summary

Recommended:

``` text
stock_balances
```

Optional but recommended for production:

``` text
stock_transfers
stock_transfer_items
idempotency_keys
```

Jika V1 ingin minimal:

``` text
stock_movements
stock_balances
```

sudah cukup untuk core inventory.

------------------------------------------------------------------------

# 69. Definition of Done

Inventory Engine selesai jika:

``` text
MASTER PART
   ↓
WAREHOUSE
   ↓
STOCK BALANCE
   ↓
RECEIVING
   ↓
STOCK IN
   ↓
WORK ORDER ISSUE
   ↓
STOCK OUT
   ↓
WORK ORDER RETURN
   ↓
STOCK IN
   ↓
ADJUSTMENT
   ↓
STOCK OPNAME
   ↓
RECONCILIATION
```

Dan seluruh perubahan:

``` text
transaction-safe
auditable
traceable
reconstructable
```

------------------------------------------------------------------------

# 70. Claude Code Master Prompt --- INVENTORY ENGINE

``` text
You are the senior inventory-engine engineer implementing Phase 13 of GARAGE PRO.

PROJECT:
GARAGE PRO — Workshop Management System V1.

READ:
- GARAGE_PRO_MASTER_DEVELOPMENT_SPECIFICATION_V1.md
- GARAGE_PRO_DATABASE_DESIGN_V1.md
- GARAGE_PRO_API_SPECIFICATION_V1.md
- GARAGE_PRO_PROJECT_ARCHITECTURE_V1.md
- GARAGE_PRO_BUSINESS_RULES_V1.md
- GARAGE_PRO_AUTH_RBAC_V1.md
- GARAGE_PRO_MASTER_DATA_V1.md
- GARAGE_PRO_WORK_ORDER_ENGINE_V1.md
- GARAGE_PRO_INVENTORY_ENGINE_V1.md

OBJECTIVE:
Implement production-quality inventory management.

CORE PRINCIPLE:
stock_movements is the immutable inventory ledger.

No stock change without a stock movement.

RECOMMENDED:
Implement stock_balances as a projection for performance.
Ledger remains source of truth.

STACK:
- Node.js
- Express
- TypeScript
- Sequelize
- MySQL 8+
- React
- Vite
- TanStack Query
- Zod
- Tailwind

IMPLEMENT:

1. Stock Balance
2. Stock Movement
3. Issue
4. Return
5. Purchase Receiving
6. Adjustment
7. Stock Opname
8. Transfer
9. Low Stock
10. Stock Valuation
11. Reconciliation
12. Rebuild Balance
13. Inventory Audit
14. Idempotency
15. Concurrency protection

MOVEMENT TYPES:

PURCHASE_RECEIPT
WORK_ORDER_ISSUE
WORK_ORDER_RETURN
ADJUSTMENT_IN
ADJUSTMENT_OUT
STOCK_OPNAME_IN
STOCK_OPNAME_OUT
TRANSFER_IN
TRANSFER_OUT

LEDGER:
Immutable.
Never update/delete.
Corrections use reversing movement.

STOCK BALANCE:
Use:
part + warehouse
as primary V1 balance key.

If location-level stock is explicitly required, use:
part + warehouse + location
consistently.

ISSUE:
- transaction
- row lock
- check stock
- create negative movement
- update balance
- audit
- idempotency

RETURN:
- validate previously issued quantity
- transaction
- create positive movement
- update balance
- audit

RECEIVING:
- validate ordered vs received
- transaction
- create PURCHASE_RECEIPT
- update balance
- update purchase item
- update purchase status

ADJUSTMENT:
- authorized permission
- reason required
- sufficient stock for OUT
- create movement
- update balance
- audit

OPNAME:
DRAFT
→ COUNTING
→ REVIEW
→ COMPLETED

Capture system quantity.
Capture physical quantity.
Calculate difference.
Generate STOCK_OPNAME_IN/OUT.
Use transaction and appropriate locking.

TRANSFER:
Must be atomic:
TRANSFER_OUT + TRANSFER_IN
inside one transaction.

NO NEGATIVE STOCK:
Default false.
Use setting:
inventory.allow_negative_stock

CONCURRENCY:
Use:
transaction
row lock
unique constraints
idempotency

Example:
two users issue last unit.
Only one succeeds.

RECONCILIATION:
Compare:
SUM(stock_movements)
vs
stock_balances.

Never silently fix mismatch.
Create explicit reconciliation operation.

VALUATION:
Implement weighted average cost or integrate with the costing policy already present.
Do not modify historical transaction costs.

API:
Implement inventory endpoints under:
`/api/v1/inventory`

FRONTEND:
Implement:
- stock overview
- low stock
- movement history
- issue
- receiving
- adjustment
- stock opname
- transfer
- reconciliation/admin view

MOBILE:
Warehouse workflows must be mobile-friendly.
Touch targets >= 44px.

PERMISSIONS:
Use:
inventory.view
inventory.issue
inventory.return
inventory.adjust
inventory.opname
and existing purchasing permissions.

SECURITY:
authenticate
authorize
validate

Never trust:
client stock
client balance
client permission
client calculation

TESTS:
Test:
- receive
- issue
- return
- adjustment
- transfer
- opname
- low stock
- valuation
- reconciliation
- idempotency
- race conditions
- permission
- immutable ledger

PROCESS:
1. Inspect repository.
2. Inspect Work Order integration.
3. Inspect Purchase integration.
4. Inspect database.
5. Add migrations.
6. Implement models.
7. Implement repositories.
8. Implement InventoryService.
9. Integrate Work Order issue/return.
10. Integrate Purchase receiving.
11. Implement APIs.
12. Implement frontend.
13. Add tests.
14. Run migrations.
15. Run typecheck.
16. Run backend tests.
17. Run frontend build.
18. Run integration/concurrency tests.

Do not stop at planning.
Actually modify the repository.

FINAL REPORT:
- files created
- files modified
- migrations
- endpoints
- movement types
- concurrency strategy
- idempotency strategy
- tests
- test results
- known limitations
- next phase
```

------------------------------------------------------------------------

# 71. Next Phase

Setelah Inventory Engine stabil:

**GARAGE_PRO_INVOICE_PAYMENT_V1.md**

Flow:

``` text
READY WO
   ↓
Invoice
   ↓
Invoice Calculation
   ↓
Discount / Tax
   ↓
Payment
   ↓
Partial Payment
   ↓
Fully Paid
   ↓
WO PAID
   ↓
WO COMPLETED
```

Phase berikutnya akan menyelesaikan sisi finansial transaksi dan menjadi
penghubung antara Work Order dengan Revenue/Reporting.
