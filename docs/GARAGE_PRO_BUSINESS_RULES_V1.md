# GARAGE PRO --- BUSINESS RULES & WORKFLOW ENGINE V1

**Document Type:** Business Rules Specification\
**Product:** GARAGE PRO --- Workshop Management System\
**Version:** V1.0\
**Status:** Development Ready\
**Date:** 14 September 2026

------------------------------------------------------------------------

## 1. Purpose

Dokumen ini menjadi sumber kebenaran business logic GARAGE PRO V1 untuk
Backend Service Layer, Work Order Engine, Inventory Engine, Invoice
Engine, Payment Engine, Permission Engine, dan implementasi Claude Code.

Prinsip:

> Frontend membantu pengguna. Backend menegakkan aturan bisnis. Database
> menjaga integritas data.

------------------------------------------------------------------------

## 2. Core Workshop Flow

``` text
CUSTOMER
  ↓
VEHICLE
  ↓
WORK ORDER
  ↓
INSPECTION
  ↓
DIAGNOSIS
  ↓
SERVICE + PART
  ↓
ESTIMATE
  ↓
CUSTOMER APPROVAL
  ↓
WORK
  ↓
QC
  ↓
READY
  ↓
INVOICE
  ↓
PAYMENT
  ↓
COMPLETED
```

------------------------------------------------------------------------

## 3. Core Entities

``` text
Customer
Vehicle
Mechanic
Work Order
Inspection
Service
Spare Part
Recommendation
Warehouse
Stock Movement
Supplier
Purchase
Invoice
Payment
User
Role
Permission
Audit Log
```

------------------------------------------------------------------------

## 4. Customer Rules

### BR-CUS-001 --- Customer ID

Setiap customer memiliki unique internal ID. Optional display code:
`CUS-000001`. Generate server-side.

### BR-CUS-002 --- Required Fields

Minimum: - name - phone

### BR-CUS-003 --- Duplicate Detection

Sistem harus memberi warning bila nomor HP sudah terdaftar atau nama
sangat mirip. Warning tidak otomatis berarti transaksi ditolak.

### BR-CUS-004 --- Deactivation

Customer yang memiliki transaksi tidak boleh hard-delete. Gunakan soft
delete/deactivation. Historical transaction tetap tersedia.

------------------------------------------------------------------------

## 5. Vehicle Rules

### BR-VEH-001 --- Ownership

Vehicle wajib memiliki customer.

### BR-VEH-002 --- Plate

Nomor polisi harus dinormalisasi untuk pencarian, tetapi dapat
ditampilkan dalam format yang mudah dibaca.

### BR-VEH-003 --- Service History

Service history berasal dari Work Order/transaksi valid, bukan tabel
history manual.

### BR-VEH-004 --- KM

KM saat WO dibuat harus `>= 0`. KM baru tidak boleh lebih kecil dari KM
terakhir tanpa koreksi berpermission khusus.

------------------------------------------------------------------------

## 6. Mechanic Rules

### BR-MEC-001

Mechanic harus aktif untuk menerima assignment baru.

### BR-MEC-002

Mechanic melihat WO yang ditugaskan kepadanya. Owner/Admin dapat melihat
seluruh WO.

### BR-MEC-003

Mechanic tidak boleh mengubah payment, invoice final, system settings,
role, atau permission kecuali permission eksplisit diberikan.

------------------------------------------------------------------------

## 7. Work Order Rules

### BR-WO-001 --- Unique Number

Format: `WO-YYYYMMDD-XXXX`

Contoh: `WO-20260914-0001`

Generate server-side dan concurrency-safe.

### BR-WO-002 --- Required Data

Minimum: - customer - vehicle - opened_at - status

### BR-WO-003 --- Vehicle/Customer Match

Vehicle harus dimiliki customer yang dipilih. Backend menolak mismatch.

### BR-WO-004 --- Mechanic

Mechanic dapat ditentukan saat create atau setelah WO dibuat.

### BR-WO-005 --- Complaint

Keluhan customer adalah historical information dan tidak boleh ditimpa
oleh diagnosis mekanik.

------------------------------------------------------------------------

## 8. Official Work Order Status

``` text
NEW
CHECKING
ESTIMATE
WAITING_APPROVAL
APPROVED
IN_PROGRESS
QC
REWORK
READY
INVOICED
PAID
COMPLETED
CANCELLED
```

------------------------------------------------------------------------

## 9. Allowed Status Transitions

``` text
NEW
 ├── CHECKING
 └── CANCELLED

CHECKING
 ├── ESTIMATE
 └── CANCELLED

ESTIMATE
 ├── WAITING_APPROVAL
 ├── APPROVED
 └── CANCELLED

WAITING_APPROVAL
 ├── APPROVED
 └── CANCELLED

APPROVED
 ├── IN_PROGRESS
 └── CANCELLED

IN_PROGRESS
 └── QC

QC
 ├── READY
 └── REWORK

REWORK
 └── IN_PROGRESS

READY
 └── INVOICED

INVOICED
 └── PAID

PAID
 └── COMPLETED
```

Invalid transitions seperti `NEW → PAID`, `QC → PAID`, atau
`COMPLETED → IN_PROGRESS` harus ditolak.

------------------------------------------------------------------------

## 10. Work Order State Engine

Jangan mengizinkan:

``` text
PATCH status = "PAID"
```

Gunakan service:

``` text
transitionWorkOrder(
  workOrderId,
  targetStatus,
  actor,
  reason?,
  metadata?
)
```

Service wajib memvalidasi: 1. WO exists 2. actor authenticated 3.
permission 4. current status 5. target status 6. prerequisites 7.
transaction rules 8. audit log

------------------------------------------------------------------------

## 11. NEW

Meaning: WO baru dibuat.

Data yang dapat dilengkapi: - customer - vehicle - mechanic - complaint

Next: `CHECKING`

------------------------------------------------------------------------

## 12. CHECKING

Mekanik melakukan pemeriksaan.

Inspection dapat dicatat.

Next: `ESTIMATE`

------------------------------------------------------------------------

## 13. ESTIMATE

Jasa, spare part dan recommendation dihitung.

**Estimate tidak mengurangi stok.**

------------------------------------------------------------------------

## 14. WAITING_APPROVAL

Digunakan ketika pekerjaan tambahan membutuhkan persetujuan customer.

Contoh:

``` text
Service existing: Rp100.000
Recommendation: Ganti Kampas Rem Rp75.000
Status: WAITING_APPROVAL
```

------------------------------------------------------------------------

## 15. Customer Approval

Approval menyimpan:

``` text
approved_by
approved_at
approval_source
```

V1 approval source:

``` text
STAFF
CUSTOMER_VERBAL
OTHER
```

Tambahan digital/WhatsApp dapat dikembangkan di V2.

------------------------------------------------------------------------

## 16. APPROVED

Sebelum `APPROVED`: - estimate tersedia bila diperlukan - approval
selesai bila diperlukan - mechanic assigned

------------------------------------------------------------------------

## 17. IN_PROGRESS

Mekanik sedang mengerjakan kendaraan.

Dapat: - menambah service - menambah part - update diagnosis - menambah
recommendation - update notes

Pekerjaan chargeable tambahan harus mengikuti approval rule.

------------------------------------------------------------------------

## 18. Additional Work Rule

Alur resmi:

``` text
Diagnosis
  ↓
Recommendation
  ↓
Customer Approval
  ↓
Convert to Work
```

Jangan menambahkan pekerjaan chargeable tanpa approval yang diwajibkan.

------------------------------------------------------------------------

## 19. QC Rule

Sebelum `READY` wajib ada: - QC result - QC actor - QC timestamp

Outcome:

``` text
PASS
REWORK
```

Jika REWORK:

``` text
QC → REWORK → IN_PROGRESS → QC
```

QC history sebelumnya tetap dipertahankan.

------------------------------------------------------------------------

## 20. READY Rule

WO dapat menjadi READY jika: - pekerjaan selesai - QC lulus - tidak ada
blocking issue

Default V1:

``` text
QC PASS → READY
```

------------------------------------------------------------------------

## 21. Invoice Rules

Invoice dibuat dari WO yang READY.

Invoice menyimpan snapshot:

``` text
item name
SKU
qty
unit price
discount
subtotal
tax
```

Perubahan master price tidak mengubah invoice lama.

------------------------------------------------------------------------

## 22. Payment Rules

Payment berlaku terhadap invoice.

Status:

``` text
UNPAID
PARTIAL
PAID
VOID
```

Formula:

``` text
Outstanding =
Invoice Total - Valid Payments
```

------------------------------------------------------------------------

## 23. Partial Payment

Jika policy mengizinkan:

``` text
Invoice = Rp200.000
Payment = Rp100.000
Outstanding = Rp100.000
Status = PARTIAL
```

------------------------------------------------------------------------

## 24. Overpayment

V1 default:

``` text
payment > outstanding
```

ditolak.

V2 dapat mendukung customer credit/refund.

------------------------------------------------------------------------

## 25. Completed Rule

Default:

``` text
PAID → COMPLETED
```

Unpaid vehicle release hanya boleh jika ada setting dan permission
eksplisit.

------------------------------------------------------------------------

## 26. Cancel Rule

Cancellation: - membutuhkan permission - membutuhkan reason -
menghasilkan audit log

Jika stock sudah dikeluarkan, gunakan `WO_RETURN`. Jangan menghapus
stock movement asli.

------------------------------------------------------------------------

## 27. Rework Rule

Rework: - menyimpan alasan - menyimpan actor - mengembalikan WO ke
IN_PROGRESS - mempertahankan history QC

------------------------------------------------------------------------

## 28. Service Rules

Service master memiliki current default price.

WO service menyimpan snapshot:

``` text
service_id
service_name_snapshot
unit_price
qty
discount
subtotal
```

Subtotal:

``` text
qty × unit_price - discount
```

Perhitungan final dilakukan backend.

------------------------------------------------------------------------

## 29. Spare Part Rules

Part memiliki: - SKU - name - category - unit - selling price - stock
configuration

SKU wajib unique.

WO part menyimpan snapshot:

``` text
part_id
sku_snapshot
name_snapshot
unit_price
qty
discount
subtotal
```

------------------------------------------------------------------------

## 30. Inventory Rule

Inventory adalah ledger-based.

Source of truth:

``` text
stock_movements
```

Setiap perubahan fisik stok harus menghasilkan movement.

------------------------------------------------------------------------

## 31. Stock Movement Types

``` text
PURCHASE_RECEIVE
WO_ISSUE
WO_RETURN
ADJUSTMENT_IN
ADJUSTMENT_OUT
OPNAME_IN
OPNAME_OUT
TRANSFER_IN
TRANSFER_OUT
```

------------------------------------------------------------------------

## 32. Stock Formula

``` text
Current Stock =
SUM(IN movements)
-
SUM(OUT movements)
```

Jika memakai cached balance untuk performance, cache harus konsisten
dengan ledger.

------------------------------------------------------------------------

## 33. Issue Part Rule

Sebelum issue:

``` text
WO exists
Part exists
Warehouse exists
WO part exists
qty > 0
WO status permits issue
user has permission
available stock >= qty
```

Jika gagal:

``` text
ROLLBACK
```

------------------------------------------------------------------------

## 34. Issue Transaction

``` text
BEGIN
 ↓
Lock stock
 ↓
Check available stock
 ↓
Create stock movement OUT
 ↓
Update WO consumed quantity
 ↓
Create audit
 ↓
COMMIT
```

Error:

``` text
ROLLBACK
```

------------------------------------------------------------------------

## 35. Return Part Rule

Return harus memiliki: - WO - original WO part item - quantity - reason

Total return tidak boleh melebihi quantity yang sudah di-issue dikurangi
return sebelumnya.

------------------------------------------------------------------------

## 36. Stock Adjustment

Membutuhkan: - permission - part - warehouse - quantity - reason

Jangan mengubah movement lama. Buat movement baru.

------------------------------------------------------------------------

## 37. Stock Opname

Flow:

``` text
Create Opname
 ↓
Select Warehouse
 ↓
System Quantity Snapshot
 ↓
Physical Count
 ↓
Variance
 ↓
Review
 ↓
Post
```

Formula:

``` text
variance = physical_qty - system_qty
```

Posting membuat `OPNAME_IN` atau `OPNAME_OUT`.

------------------------------------------------------------------------

## 38. Low Stock

Default condition:

``` text
available_stock <= minimum_stock
```

Tampilkan:

``` text
LOW STOCK
```

V1 tidak otomatis membuat PO.

------------------------------------------------------------------------

## 39. Purchase Rules

Status:

``` text
DRAFT
SUBMITTED
ORDERED
PARTIALLY_RECEIVED
RECEIVED
CANCELLED
```

Purchase tidak menambah stok.

Receiving menambah stok.

------------------------------------------------------------------------

## 40. Receiving Rule

Validasi: - purchase valid - supplier valid - warehouse valid - qty
received \> 0 - qty received \<= remaining ordered qty

Atomic flow:

``` text
Create receiving
 ↓
Create stock IN movement
 ↓
Update received quantity
 ↓
Update purchase status
 ↓
COMMIT
```

------------------------------------------------------------------------

## 41. Purchase Cost

Received quantity menyimpan historical unit cost.

Perubahan harga supplier berikutnya tidak mengubah receiving lama.

------------------------------------------------------------------------

## 42. Invoice Calculation

``` text
Service Subtotal
+
Part Subtotal
-
Discount
+
Tax
=
Grand Total
```

Server menghitung final total.

------------------------------------------------------------------------

## 43. Discount Rule

Discount dapat berupa: - item-level - invoice-level

Default authority: - OWNER - ADMIN

Mechanic tidak boleh menetapkan final financial discount tanpa
permission.

------------------------------------------------------------------------

## 44. Tax Rule

Jika tax disabled:

``` text
Tax = 0
```

Jika enabled:

``` text
Tax = taxable_base × tax_rate
```

Invoice menyimpan applied tax rate dan amount.

------------------------------------------------------------------------

## 45. Payment Methods

``` text
CASH
TRANSFER
QRIS
DEBIT
CREDIT_CARD
OTHER
```

Reference dapat diwajibkan berdasarkan policy.

------------------------------------------------------------------------

## 46. Payment Atomicity

``` text
BEGIN
 ↓
Lock invoice
 ↓
Calculate outstanding
 ↓
Validate payment
 ↓
Create payment
 ↓
Update invoice payment status
 ↓
Update WO if applicable
 ↓
Audit
 ↓
COMMIT
```

------------------------------------------------------------------------

## 47. Duplicate Payment Protection

Frontend: - disable submit - loading state

Backend: - Idempotency-Key - simpan hasil request - request yang sama
mengembalikan hasil asli

------------------------------------------------------------------------

## 48. Invoice Void

Void membutuhkan: - permission - reason - audit

Invoice paid tidak boleh hard-delete.

Payment yang terkait harus menggunakan reversal/void workflow.

------------------------------------------------------------------------

## 49. Price Freeze

Prinsip:

> Master data adalah current state. Transaction data adalah historical
> state.

Contoh:

``` text
Service master = Rp75.000
WO snapshot = Rp75.000

Master kemudian = Rp85.000

WO lama tetap = Rp75.000
```

------------------------------------------------------------------------

## 50. Service History

History kendaraan menampilkan:

``` text
Date
KM
WO
Services
Parts
Total
Mechanic
```

Cancelled WO tidak dihitung sebagai completed service history.

------------------------------------------------------------------------

## 51. Audit

Minimal audit action:

``` text
LOGIN
CREATE
UPDATE
DEACTIVATE
STATUS_CHANGE
APPROVAL
STOCK_ISSUE
STOCK_RETURN
STOCK_ADJUSTMENT
OPNAME_POST
PURCHASE_RECEIVE
INVOICE_CREATE
INVOICE_VOID
PAYMENT_CREATE
PAYMENT_VOID
PERMISSION_CHANGE
SETTINGS_CHANGE
```

Audit tidak boleh diedit oleh ordinary users dan tidak memiliki delete
button di V1.

------------------------------------------------------------------------

## 52. Permission Model

Format:

``` text
resource.action
```

Contoh:

``` text
customers.view
customers.create
customers.update

work_orders.view
work_orders.create
work_orders.approve
work_orders.issue_part
work_orders.cancel
work_orders.qc
work_orders.complete

inventory.view
inventory.issue
inventory.return
inventory.adjust
inventory.opname

payments.view
payments.create
payments.void

reports.view
```

Backend adalah security boundary.

------------------------------------------------------------------------

## 53. Role Rules

### OWNER

Full access, termasuk financial, inventory, reports, users, roles dan
settings.

### ADMIN

Operational management, customer, vehicle, WO, invoice, payment,
inventory dan reports. Tidak otomatis memiliki role/settings
administration.

### MECHANIC

Assigned WO, inspection, diagnosis, service, parts, recommendation dan
QC jika diizinkan.

### WAREHOUSE

Parts, inventory, receiving, issue, return, adjustment dan opname.

------------------------------------------------------------------------

## 54. Data Scope

Mechanic default:

``` text
mechanic_id = currentUser.mechanic_id
```

Warehouse user dapat dibatasi pada warehouse tertentu.

Owner/Admin default dapat melihat seluruh workshop data.

------------------------------------------------------------------------

## 55. Work Order Edit Rules

Normal editing diperbolehkan saat:

``` text
NEW
CHECKING
ESTIMATE
WAITING_APPROVAL
APPROVED
IN_PROGRESS
REWORK
```

Setelah invoice dibuat, financial data dikunci dari normal editing.

PAID/COMPLETED bersifat historical.

------------------------------------------------------------------------

## 56. Concurrency

Critical resources: - stock - invoice - payment - document number

Gunakan: - database transactions - row locks - unique constraints -
idempotency

------------------------------------------------------------------------

## 57. Money

Gunakan:

``` text
DECIMAL
```

Jangan gunakan FLOAT untuk uang.

------------------------------------------------------------------------

## 58. Quantity

Default motorcycle parts:

``` text
PCS
```

Future:

``` text
LITER
SET
PAIR
BOX
```

Issue/receive quantity harus valid dan positif.

------------------------------------------------------------------------

## 59. Time

Gunakan timezone konsisten.

Recommended:

``` text
Storage: UTC
Display: Asia/Jakarta
```

Jika database menggunakan local timezone, semua service harus konsisten.

------------------------------------------------------------------------

## 60. Delete Rules

Jangan hard-delete:

``` text
Work Order
Invoice
Payment
Stock Movement
Audit Log
Purchase Receipt
```

Gunakan: - cancel - void - reverse - deactivate

sesuai domain.

------------------------------------------------------------------------

## 61. Search Rules

Customer:

``` text
name
phone
```

Vehicle:

``` text
plate
brand
model
```

WO:

``` text
WO number
customer
plate
```

Part:

``` text
SKU
barcode
name
```

Invoice:

``` text
invoice number
customer
plate
```

------------------------------------------------------------------------

## 62. Dashboard Rules

Owner/Admin: - WO hari ini - revenue - in-progress - ready - low stock -
recent activity

Mechanic: - assigned work - active WO - QC tasks

Warehouse: - low stock - receiving - stock movement

------------------------------------------------------------------------

## 63. Report Rules

Date filter presets:

``` text
Today
Yesterday
This Week
This Month
Last Month
Custom
```

Report harus jelas menggunakan tanggal apa: - WO opened date - completed
date - invoice date - payment date

------------------------------------------------------------------------

## 64. Revenue Definition

Default revenue report:

> Valid recorded payments pada payment date yang dipilih.

Invoice sales value adalah metric berbeda dan harus diberi label
berbeda.

------------------------------------------------------------------------

## 65. Stock Valuation

Jika stock value diaktifkan, metode harus eksplisit.

Recommended V1:

``` text
Weighted Average Cost
```

Jangan mencampur metode valuation.

------------------------------------------------------------------------

## 66. Traceability

Stock movement harus dapat ditelusuri:

``` text
Movement
 ↓
Reference
 ↓
Source Transaction
 ↓
User
 ↓
Timestamp
```

Recommendation approval juga harus traceable:

``` text
Recommendation
 ↓
Approval
 ↓
Converted Work
```

------------------------------------------------------------------------

## 67. Business Rule Priority

Jika terdapat konflik:

``` text
Security
 ↓
Database Integrity
 ↓
Financial Integrity
 ↓
Inventory Integrity
 ↓
Workflow Rules
 ↓
UX Convenience
```

UX convenience tidak boleh mengalahkan financial atau inventory
integrity.

------------------------------------------------------------------------

## 68. Transaction Boundary Matrix

  Operation                      Transaction
  ---------------------------- -------------
  Create Customer                   Optional
  Create Vehicle                    Optional
  Create Service                    Optional
  Create Part                       Optional
  Create WO                      Recommended
  Add Service                    Recommended
  Add Recommendation             Recommended
  Issue Part                        REQUIRED
  Return Part                       REQUIRED
  Receive Purchase                  REQUIRED
  Stock Adjustment                  REQUIRED
  Stock Opname Post                 REQUIRED
  Generate Invoice                  REQUIRED
  Payment                           REQUIRED
  Cancel WO + stock reversal        REQUIRED

------------------------------------------------------------------------

## 69. Idempotency

Support `Idempotency-Key` untuk:

``` text
payment
receiving
stock issue
stock adjustment
invoice generation
```

Repeated request dengan key sama tidak boleh membuat transaksi kedua.

------------------------------------------------------------------------

## 70. Business Error Codes

``` text
CUSTOMER_NOT_FOUND
VEHICLE_NOT_FOUND
VEHICLE_CUSTOMER_MISMATCH
INVALID_WO_STATUS
INVALID_WO_TRANSITION
WO_NOT_ASSIGNED
APPROVAL_REQUIRED
APPROVAL_ALREADY_EXISTS
PART_NOT_FOUND
INSUFFICIENT_STOCK
INVALID_PART_QUANTITY
PURCHASE_NOT_FOUND
RECEIVE_QTY_EXCEEDED
INVOICE_NOT_FOUND
INVOICE_ALREADY_VOID
INVOICE_ALREADY_PAID
PAYMENT_EXCEEDS_OUTSTANDING
PAYMENT_ALREADY_PROCESSED
PERMISSION_DENIED
DUPLICATE_DOCUMENT_NUMBER
DUPLICATE_IDEMPOTENCY_KEY
```

------------------------------------------------------------------------

## 71. Work Order Action Matrix

  ------------------------------------------------------------------------------------------------------------------
  Action              NEW   CHECK    EST   WAIT   APPROVED   PROGRESS        QC   READY   INVOICED   PAID   COMPLETE
  ---------------- ------ ------- ------ ------ ---------- ---------- --------- ------- ---------- ------ ----------
  Edit basic            ✓       ✓      ✓      ✓          ✓    Limited        \-      \-         \-     \-         \-

  Inspection           \-       ✓      ✓     \-         \-          ✓        \-      \-         \-     \-         \-

  Add service          \-       ✓      ✓      ✓          ✓          ✓        \-      \-         \-     \-         \-

  Add part             \-       ✓      ✓      ✓          ✓          ✓        \-      \-         \-     \-         \-

  Issue part           \-      \-     \-     \-         \-          ✓   Limited      \-         \-     \-         \-

  Recommendation       \-       ✓      ✓      ✓          ✓          ✓        \-      \-         \-     \-         \-

  Approval             \-      \-      ✓      ✓         \-         \-        \-      \-         \-     \-         \-

  Start work           \-      \-     \-     \-          ✓         \-        \-      \-         \-     \-         \-

  QC                   \-      \-     \-     \-         \-          ✓         ✓      \-         \-     \-         \-

  Invoice              \-      \-     \-     \-         \-         \-        \-       ✓         \-     \-         \-

  Payment              \-      \-     \-     \-         \-         \-        \-      \-          ✓     \-         \-

  Complete             \-      \-     \-     \-         \-         \-        \-      \-         \-      ✓         \-
  ------------------------------------------------------------------------------------------------------------------

------------------------------------------------------------------------

## 72. Default Workshop Policy

V1 default:

``` text
Additional chargeable work requires customer approval
Stock deducted when part is issued/consumed
Invoice generated after READY
Payment required before COMPLETED
Overpayment rejected
Paid WO locked
Cancelled WO excluded from completed service history
```

Do not over-configure these policies in V1.

------------------------------------------------------------------------

## 73. Workflow Engine Pseudocode

``` text
transitionWO(wo, target, actor):

  assertAuthenticated(actor)
  assertPermission(actor, target)
  assertCurrentStatusAllows(wo.status, target)
  assertPrerequisites(wo, target)

  beginTransaction

    update wo.status
    create audit log

  commit

  return updated WO
```

------------------------------------------------------------------------

## 74. Issue Part Pseudocode

``` text
issuePart(woPart, actor):

  assertPermission(actor, "work_orders.issue_part")
  assertWOAllowsIssue(wo.status)
  assertQtyValid(woPart.qty)

  beginTransaction

    lockStock(part, warehouse)
    assertAvailableStock >= qty

    createStockMovement(type=WO_ISSUE, qty=qty)
    updateWOConsumedQty()
    createAudit()

  commit
```

------------------------------------------------------------------------

## 75. Payment Pseudocode

``` text
createPayment(invoice, amount, method, actor):

  assertPermission(actor, "payments.create")

  beginTransaction

    lockInvoice(invoice)
    outstanding = calculateOutstanding(invoice)

    assert amount > 0
    assert amount <= outstanding

    createPayment()
    updatePaymentStatus()
    createAudit()

  commit
```

------------------------------------------------------------------------

## 76. Business Rules Definition of Done

``` text
[ ] Customer rules
[ ] Vehicle rules
[ ] Mechanic rules
[ ] WO state machine
[ ] Approval
[ ] Inspection
[ ] Services
[ ] Parts
[ ] Inventory
[ ] Purchase
[ ] Receiving
[ ] Stock Opname
[ ] Invoice
[ ] Payment
[ ] QC
[ ] Rework
[ ] Cancellation
[ ] Permissions
[ ] Audit
[ ] Idempotency
[ ] Concurrency
[ ] Financial integrity
[ ] Historical price freeze
[ ] Error codes
```

------------------------------------------------------------------------

## 77. Next Implementation Phase

Setelah seluruh blueprint V1 selesai, masuk ke implementasi:

``` text
01 PROJECT SCAFFOLD
02 DATABASE MIGRATION
03 SEED DATA
04 AUTHENTICATION + RBAC
05 MASTER DATA
06 WORK ORDER ENGINE
07 INVENTORY ENGINE
08 INVOICE + PAYMENT
09 FRONTEND INTEGRATION
10 TESTING
11 DEPLOYMENT
```

------------------------------------------------------------------------

# END OF DOCUMENT

**GARAGE PRO --- BUSINESS RULES & WORKFLOW ENGINE V1**\
**Status: Development Ready**
