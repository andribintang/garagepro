# GARAGE PRO --- INVOICE + PAYMENT ENGINE V1

## Financial Transaction Specification & Claude Code Implementation Guide

**Project:** GARAGE PRO --- Workshop Management System\
**Phase:** 14 --- Invoice + Payment\
**Version:** V1.0\
**Status:** Implementation Ready\
**Primary stack:** React + Vite + TypeScript / Node.js + Express +
Sequelize + MySQL 8+

------------------------------------------------------------------------

# 1. PURPOSE

Invoice + Payment adalah financial transaction engine GARAGE PRO.

Engine ini bertanggung jawab mengubah Work Order yang sudah READY
menjadi tagihan resmi, menerima pembayaran, menghitung saldo secara
akurat, serta menjadi sumber data untuk revenue report.

Prinsip utama:

``` text
READY WO
   ↓
Create Invoice
   ↓
Invoice Calculation
   ↓
Issue Invoice
   ↓
Payment
   ↓
Partial / Full Payment
   ↓
Invoice PAID
   ↓
WO PAID
   ↓
WO COMPLETED
```

Invoice dan payment harus bersifat **transaction-safe, auditable,
immutable pada data historis, dan server-calculated**.

------------------------------------------------------------------------

# 2. OBJECTIVES

## 2.1 Functional Objectives

System harus mampu:

1.  Generate invoice dari WO READY.
2.  Menyalin service dan spare part menjadi invoice lines.
3.  Membekukan harga transaksi.
4.  Menghitung subtotal, discount, tax, grand total.
5.  Menerbitkan invoice.
6.  Menerima pembayaran penuh.
7.  Menerima partial payment.
8.  Menghitung outstanding balance.
9.  Menolak overpayment secara default.
10. Mencegah double payment akibat concurrent request.
11. Mendukung void invoice sesuai aturan.
12. Mendukung void/refund payment dengan audit.
13. Menyediakan receipt.
14. Menyediakan print/PDF readiness.
15. Menghubungkan revenue report dengan invoice/payment.
16. Mengunci transaksi yang sudah dibayar.
17. Menyimpan audit trail.

## 2.2 Non-Functional Objectives

-   Atomic transaction.
-   Idempotent payment.
-   Decimal-safe money calculation.
-   Concurrency-safe.
-   Role-based authorization.
-   Full auditability.
-   API-first.
-   Mobile-friendly cashier workflow.
-   Print/PDF-ready.

------------------------------------------------------------------------

# 3. SCOPE

## Included

-   Invoice generation.
-   Invoice detail.
-   Invoice calculation.
-   Discount.
-   Tax.
-   Payment.
-   Partial payment.
-   Payment history.
-   Receipt.
-   Invoice void.
-   Payment void/refund.
-   Audit.
-   Revenue hooks.
-   Dashboard/reports integration.
-   API.
-   Frontend UI contract.

## Out of Scope V1

-   Full accounting/general ledger.
-   Accounts receivable aging.
-   Accounts payable.
-   Bank reconciliation.
-   Multi-currency.
-   E-wallet settlement reconciliation.
-   Automated tax filing.
-   Payment gateway settlement API.
-   Credit financing integration.

------------------------------------------------------------------------

# 4. CORE BUSINESS PRINCIPLES

## 4.1 Invoice is Financial Snapshot

Invoice harus menyimpan snapshot:

-   customer
-   vehicle
-   WO number
-   service description
-   part description
-   quantity
-   unit price
-   discount
-   tax
-   totals

Perubahan master service atau spare part setelah invoice dibuat tidak
boleh mengubah invoice lama.

## 4.2 Server is Source of Truth

Frontend hanya mengirim:

-   selected payment method
-   amount
-   reference number
-   notes
-   optional idempotency key

Frontend tidak boleh menentukan:

-   subtotal
-   tax amount
-   grand total
-   paid amount
-   outstanding amount
-   invoice status

Semua dihitung server.

## 4.3 Money Rules

Gunakan:

``` text
DECIMAL(18,2)
```

Jangan gunakan floating point JavaScript untuk menyimpan nilai
transaksi.

## 4.4 No Silent Mutation

Invoice/payment yang sudah menjadi transaksi resmi tidak boleh diedit
bebas.

Jika terjadi koreksi:

``` text
VOID
REFUND
ADJUSTMENT
```

harus digunakan sesuai business rule.

------------------------------------------------------------------------

# 5. INVOICE LIFECYCLE

## 5.1 Invoice Status

``` text
DRAFT
ISSUED
PARTIAL
PAID
VOID
```

### DRAFT

Invoice sudah dibuat tetapi belum resmi diterbitkan.

### ISSUED

Invoice resmi dan belum menerima pembayaran.

### PARTIAL

Invoice sudah menerima sebagian pembayaran.

### PAID

Outstanding = 0.

### VOID

Invoice dibatalkan dan tidak dapat dibayar.

------------------------------------------------------------------------

# 6. INVOICE STATE MACHINE

``` text
DRAFT
  │
  ├── ISSUE
  ↓
ISSUED
  │
  ├── PAYMENT
  ↓
PARTIAL
  │
  └── PAYMENT
       ↓
      PAID

DRAFT ──VOID──> VOID
ISSUED ──VOID──> VOID
```

Tidak diperbolehkan:

``` text
PAID → EDIT
PAID → VOID
PAID → DELETE
VOID → ISSUE
VOID → PAYMENT
```

Koreksi setelah PAID harus melalui mekanisme refund/adjustment pada fase
yang relevan.

------------------------------------------------------------------------

# 7. INVOICE GENERATION

Invoice hanya dapat dibuat dari:

``` text
WO.status = READY
```

Validation:

1.  WO exists.
2.  WO belongs to valid customer.
3.  Vehicle valid.
4.  WO has billable lines.
5.  WO has no existing active invoice.
6.  WO is not cancelled/rejected.
7.  User has `invoice.create`.

Jika validation gagal:

``` text
409 WO_NOT_READY_FOR_INVOICE
```

------------------------------------------------------------------------

# 8. INVOICE NUMBER

Format rekomendasi:

``` text
INV-YYYYMM-000001
```

Contoh:

``` text
INV-202609-000127
```

Invoice number:

-   generated server-side
-   unique
-   never reused
-   not editable by frontend
-   indexed UNIQUE

Untuk multi-branch di masa depan, format dapat diperluas:

``` text
INV-B01-202609-000127
```

------------------------------------------------------------------------

# 9. INVOICE DATA SNAPSHOT

Ketika invoice dibuat, copy data berikut:

## Header

-   customer_id
-   customer_name_snapshot
-   customer_phone_snapshot
-   vehicle_id
-   vehicle_plate_snapshot
-   vehicle_brand_snapshot
-   vehicle_model_snapshot
-   work_order_id
-   work_order_number
-   invoice_number
-   invoice_date

## Service Line

-   service_id
-   service_code_snapshot
-   service_name_snapshot
-   qty
-   unit_price
-   discount
-   line_total

## Part Line

-   spare_part_id
-   part_code_snapshot
-   part_name_snapshot
-   qty
-   unit_price
-   discount
-   line_total

Snapshot diperlukan agar invoice historis tetap benar meskipun master
data berubah.

------------------------------------------------------------------------

# 10. DATABASE DESIGN

## 10.1 invoices

Recommended fields:

``` text
id BIGINT UNSIGNED PK
invoice_number VARCHAR(50) UNIQUE
work_order_id BIGINT UNSIGNED NOT NULL
customer_id BIGINT UNSIGNED NOT NULL
vehicle_id BIGINT UNSIGNED NOT NULL

invoice_date DATETIME NOT NULL

status VARCHAR(20) NOT NULL

subtotal DECIMAL(18,2) NOT NULL DEFAULT 0
line_discount_total DECIMAL(18,2) NOT NULL DEFAULT 0
document_discount DECIMAL(18,2) NOT NULL DEFAULT 0
tax_rate DECIMAL(8,4) NOT NULL DEFAULT 0
tax_amount DECIMAL(18,2) NOT NULL DEFAULT 0
grand_total DECIMAL(18,2) NOT NULL DEFAULT 0

paid_amount DECIMAL(18,2) NOT NULL DEFAULT 0
outstanding_amount DECIMAL(18,2) NOT NULL DEFAULT 0

notes TEXT NULL

issued_at DATETIME NULL
voided_at DATETIME NULL
void_reason TEXT NULL

created_by BIGINT UNSIGNED NOT NULL
updated_by BIGINT UNSIGNED NULL

created_at DATETIME NOT NULL
updated_at DATETIME NOT NULL
deleted_at DATETIME NULL
```

Recommended indexes:

``` text
UNIQUE(invoice_number)
INDEX(work_order_id)
INDEX(customer_id)
INDEX(vehicle_id)
INDEX(status)
INDEX(invoice_date)
INDEX(created_at)
```

------------------------------------------------------------------------

# 11. invoice_items

Recommended fields:

``` text
id BIGINT UNSIGNED PK
invoice_id BIGINT UNSIGNED NOT NULL

line_type VARCHAR(20) NOT NULL
source_service_id BIGINT UNSIGNED NULL
source_part_id BIGINT UNSIGNED NULL

item_code_snapshot VARCHAR(100) NULL
item_name_snapshot VARCHAR(255) NOT NULL
description_snapshot TEXT NULL

qty DECIMAL(18,3) NOT NULL
unit_price DECIMAL(18,2) NOT NULL

discount_type VARCHAR(20) NOT NULL DEFAULT 'AMOUNT'
discount_value DECIMAL(18,2) NOT NULL DEFAULT 0
discount_amount DECIMAL(18,2) NOT NULL DEFAULT 0

taxable BOOLEAN NOT NULL DEFAULT TRUE
tax_rate DECIMAL(8,4) NOT NULL DEFAULT 0
tax_amount DECIMAL(18,2) NOT NULL DEFAULT 0

line_subtotal DECIMAL(18,2) NOT NULL
line_total DECIMAL(18,2) NOT NULL

sort_order INT NOT NULL DEFAULT 0

created_at DATETIME NOT NULL
updated_at DATETIME NOT NULL
```

`line_type`:

``` text
SERVICE
PART
OTHER
```

------------------------------------------------------------------------

# 12. payments

Recommended fields:

``` text
id BIGINT UNSIGNED PK
payment_number VARCHAR(50) UNIQUE
invoice_id BIGINT UNSIGNED NOT NULL

payment_date DATETIME NOT NULL

amount DECIMAL(18,2) NOT NULL

payment_method VARCHAR(30) NOT NULL
status VARCHAR(20) NOT NULL DEFAULT 'SUCCESS'

reference_number VARCHAR(100) NULL
notes TEXT NULL

received_by BIGINT UNSIGNED NOT NULL

voided_at DATETIME NULL
void_reason TEXT NULL

refunded_at DATETIME NULL
refund_reference VARCHAR(100) NULL
refund_reason TEXT NULL

idempotency_key VARCHAR(100) UNIQUE NULL

created_at DATETIME NOT NULL
updated_at DATETIME NOT NULL
```

Indexes:

``` text
UNIQUE(payment_number)
UNIQUE(idempotency_key)
INDEX(invoice_id)
INDEX(payment_date)
INDEX(status)
INDEX(payment_method)
INDEX(reference_number)
```

------------------------------------------------------------------------

# 13. PAYMENT METHODS

V1 supports:

``` text
CASH
TRANSFER
QRIS
DEBIT_CARD
CREDIT_CARD
OTHER
```

Payment method labels should be configurable for UI, but stored
enum-like as stable codes.

Example:

``` json
{
  "code": "QRIS",
  "label": "QRIS"
}
```

------------------------------------------------------------------------

# 14. PAYMENT STATUS

``` text
SUCCESS
VOID
REFUNDED
```

Only:

``` text
SUCCESS
```

counts toward invoice paid amount.

Formula:

``` text
paid_amount =
SUM(successful payment.amount)
```

------------------------------------------------------------------------

# 15. INVOICE CALCULATION

## 15.1 Line Subtotal

``` text
line_subtotal = qty × unit_price
```

## 15.2 Line Discount

If amount:

``` text
discount_amount = discount_value
```

If percentage:

``` text
discount_amount = line_subtotal × discount_value / 100
```

Never allow:

``` text
discount_amount > line_subtotal
```

## 15.3 Line Tax

``` text
taxable_base = line_subtotal - discount_amount
tax_amount = taxable_base × tax_rate / 100
```

## 15.4 Line Total

``` text
line_total =
line_subtotal
- discount_amount
+ tax_amount
```

## 15.5 Invoice Subtotal

``` text
subtotal = SUM(line_subtotal)
```

## 15.6 Total Line Discount

``` text
line_discount_total =
SUM(discount_amount)
```

## 15.7 Document Discount

Document discount applies after line discounts.

Validation:

``` text
document_discount <= subtotal - line_discount_total
```

## 15.8 Tax

Recommended:

``` text
taxable_base =
subtotal
- line_discount_total
- document_discount
```

Then:

``` text
tax_amount =
taxable_base × tax_rate / 100
```

## 15.9 Grand Total

``` text
grand_total =
subtotal
- line_discount_total
- document_discount
+ tax_amount
```

All calculations must happen server-side.

------------------------------------------------------------------------

# 16. ROUNDING

Use deterministic rounding.

Recommended:

``` text
ROUND_HALF_UP
```

Round monetary values to 2 decimals.

Example:

``` text
10,000.005 → 10,000.01
```

Use a decimal library/service layer rather than native floating
arithmetic.

------------------------------------------------------------------------

# 17. PAYMENT BALANCE

``` text
paid_amount =
SUM(SUCCESS payments)
```

``` text
outstanding =
grand_total - paid_amount
```

Status logic:

``` text
paid_amount = 0
→ ISSUED

0 < paid_amount < grand_total
→ PARTIAL

paid_amount = grand_total
→ PAID
```

Use a tolerance of exactly 0.00 after decimal normalization; do not use
arbitrary floating-point epsilon.

------------------------------------------------------------------------

# 18. OVERPAYMENT POLICY

V1:

**OVERPAYMENT IS REJECTED.**

Validation:

``` text
payment_amount <= outstanding_amount
```

If:

``` text
payment_amount > outstanding_amount
```

return:

``` text
422 PAYMENT_EXCEEDS_OUTSTANDING
```

Future versions may support customer credit balance.

------------------------------------------------------------------------

# 19. PAYMENT IDEMPOTENCY

Payment creation must support:

``` text
Idempotency-Key
```

Example:

``` http
POST /api/v1/invoices/INV-202609-000127/payments
Idempotency-Key: PAY-8f2c...
```

If the same key is submitted again:

-   do not create another payment
-   return the original payment result

This protects against:

-   double click
-   network retry
-   mobile reconnect
-   browser retry
-   frontend timeout

------------------------------------------------------------------------

# 20. CONCURRENCY CONTROL

Payment creation must use database transaction + row lock.

Recommended flow:

``` text
BEGIN TRANSACTION

SELECT invoice
FOR UPDATE

calculate current paid
calculate outstanding

validate amount

INSERT payment

UPDATE invoice paid_amount/status/outstanding

COMMIT
```

Two simultaneous payments must never be able to exceed outstanding
balance.

Example:

``` text
Invoice = 500,000
Outstanding = 500,000

Request A = 300,000
Request B = 300,000
```

Only one combination that remains within balance may succeed.

The second request must re-read the locked invoice and receive:

``` text
PAYMENT_EXCEEDS_OUTSTANDING
```

if balance is insufficient.

------------------------------------------------------------------------

# 21. INVOICE ISSUE

Endpoint:

``` http
POST /api/v1/invoices/:id/issue
```

Requirements:

-   status = DRAFT
-   totals calculated
-   invoice number exists
-   billable lines exist

Result:

``` text
DRAFT → ISSUED
```

Set:

``` text
issued_at = NOW()
```

After issue, invoice lines should not be freely editable.

------------------------------------------------------------------------

# 22. CREATE INVOICE FROM WORK ORDER

Endpoint:

``` http
POST /api/v1/work-orders/:id/invoice
```

Recommended flow:

``` text
Validate WO READY
       ↓
Check existing invoice
       ↓
Load WO services
       ↓
Load WO parts
       ↓
Create invoice
       ↓
Snapshot lines
       ↓
Calculate totals
       ↓
Return invoice
```

Default behavior:

Invoice starts as:

``` text
DRAFT
```

Admin/cashier then reviews and issues it.

Optional future setting:

``` text
auto_issue_invoice_on_ready = true
```

------------------------------------------------------------------------

# 23. PAYMENT CREATION

Endpoint:

``` http
POST /api/v1/invoices/:id/payments
```

Request:

``` json
{
  "amount": 250000,
  "payment_method": "CASH",
  "reference_number": null,
  "notes": "Pembayaran pertama"
}
```

Response:

``` json
{
  "success": true,
  "data": {
    "payment_number": "PAY-202609-000091",
    "invoice_number": "INV-202609-000127",
    "amount": 250000,
    "payment_method": "CASH",
    "status": "SUCCESS",
    "invoice_status": "PARTIAL",
    "paid_amount": 250000,
    "outstanding_amount": 350000
  }
}
```

------------------------------------------------------------------------

# 24. PAYMENT NUMBER

Recommended:

``` text
PAY-YYYYMM-000001
```

Example:

``` text
PAY-202609-000091
```

Rules:

-   server-generated
-   unique
-   immutable
-   never reused

------------------------------------------------------------------------

# 25. PARTIAL PAYMENT

Example:

``` text
Grand Total: 1,250,000

Payment 1:
500,000 CASH

Payment 2:
500,000 TRANSFER

Payment 3:
250,000 QRIS
```

Final:

``` text
Paid: 1,250,000
Outstanding: 0
Status: PAID
```

Payment history must show each payment independently.

------------------------------------------------------------------------

# 26. PAYMENT ALLOCATION

V1 uses:

``` text
one payment → one invoice
```

No payment splitting across multiple invoices.

This keeps reconciliation simple and safe.

Future:

``` text
one receipt → multiple invoices
```

can be implemented using a `payment_allocations` table.

------------------------------------------------------------------------

# 27. WORK ORDER INTEGRATION

When invoice becomes PAID:

``` text
Invoice PAID
     ↓
WO PAID
     ↓
WO COMPLETED
```

Recommended transaction:

``` text
Payment transaction
   ├── update payment
   ├── update invoice
   ├── if fully paid:
   │     invoice.status = PAID
   │     WO.status = PAID
   │     WO.status = COMPLETED
   └── audit
```

The exact WO transition must still use the existing Work Order
state-machine service rather than direct uncontrolled status assignment.

------------------------------------------------------------------------

# 28. COMPLETION RULE

WO can only become COMPLETED if:

``` text
invoice.status = PAID
```

unless an explicit configuration/exception policy is implemented.

Default V1:

``` text
NO PAYMENT → NO COMPLETION
```

------------------------------------------------------------------------

# 29. INVOICE VOID

Endpoint:

``` http
POST /api/v1/invoices/:id/void
```

Allowed:

``` text
DRAFT → VOID
ISSUED → VOID
```

Not allowed:

``` text
PARTIAL → VOID
PAID → VOID
```

unless a controlled reversal workflow is implemented.

Required:

``` json
{
  "reason": "Salah customer / invoice dibuat ulang"
}
```

Minimum reason length:

``` text
10 characters
```

Audit:

``` text
invoice.voided
```

------------------------------------------------------------------------

# 30. PAYMENT VOID

Endpoint:

``` http
POST /api/v1/payments/:id/void
```

Recommended rule:

Payment void is only allowed under controlled authorization.

For a payment that causes invoice balance to change, system must
recalculate:

``` text
paid_amount
outstanding_amount
status
```

Example:

``` text
Invoice 1,000,000
Payment 1,000,000
Status PAID

Payment void
↓
Paid 0
Outstanding 1,000,000
Status ISSUED
```

If the related WO was already COMPLETED, system must use a controlled
reversal flow rather than silently modifying history.

------------------------------------------------------------------------

# 31. REFUND

V1 supports data readiness for refund but should keep the workflow
restricted.

Recommended rule:

``` text
SUCCESS → REFUNDED
```

Refund requires:

-   authorization
-   refund amount
-   reason
-   refund reference
-   audit

Never delete the original payment.

The original payment remains visible with:

``` text
status = REFUNDED
```

------------------------------------------------------------------------

# 32. AUDIT EVENTS

Minimum audit events:

``` text
invoice.created
invoice.issued
invoice.voided

payment.created
payment.voided
payment.refunded

invoice.paid
invoice.status_changed
wo.payment_completed
```

Audit should record:

``` text
actor_user_id
action
entity_type
entity_id
before
after
ip_address
user_agent
request_id
timestamp
```

Sensitive data such as passwords/tokens must never be stored in audit
payload.

------------------------------------------------------------------------

# 33. PERMISSIONS

Recommended permissions:

``` text
invoice.view
invoice.create
invoice.issue
invoice.void

payment.view
payment.create
payment.void
payment.refund

receipt.view
receipt.print

report.revenue.view
```

Role matrix:

  Permission         OWNER        ADMIN   MECHANIC   WAREHOUSE
  ---------------- ------- ------------ ---------- -----------
  invoice.view           ✓            ✓    limited          \-
  invoice.create         ✓            ✓         \-          \-
  invoice.issue          ✓            ✓         \-          \-
  invoice.void           ✓          ✓\*         \-          \-
  payment.view           ✓            ✓         \-          \-
  payment.create         ✓            ✓         \-          \-
  payment.void           ✓          ✓\*         \-          \-
  payment.refund         ✓   restricted         \-          \-
  receipt.print          ✓            ✓         \-          \-
  revenue report         ✓            ✓         \-          \-

`*` should be controlled by business policy.

------------------------------------------------------------------------

# 34. INVOICE UI

## 34.1 Invoice List

Route:

``` text
/invoices
```

Columns:

-   Invoice Number
-   Date
-   Customer
-   Vehicle
-   WO
-   Total
-   Paid
-   Outstanding
-   Status
-   Actions

Filters:

-   date range
-   status
-   customer
-   invoice number
-   WO number

Actions:

``` text
View
Print
Issue
Record Payment
Void
```

------------------------------------------------------------------------

# 35. INVOICE DETAIL

Route:

``` text
/invoices/:id
```

Layout:

``` text
Invoice Header
Customer / Vehicle
WO Reference
--------------------------------
Services
Parts
--------------------------------
Subtotal
Discount
Tax
Grand Total
--------------------------------
Payment Summary
Paid
Outstanding
--------------------------------
Payment History
--------------------------------
Timeline / Audit
```

Primary CTA based on status:

``` text
DRAFT:
[Issue Invoice]

ISSUED:
[Receive Payment]

PARTIAL:
[Receive Payment]

PAID:
[Print Receipt]

VOID:
[View Only]
```

------------------------------------------------------------------------

# 36. PAYMENT MODAL

Desktop:

``` text
┌───────────────────────────────┐
│ Receive Payment                │
├───────────────────────────────┤
│ Invoice       INV-202609-127  │
│ Total         Rp 750.000      │
│ Paid          Rp 250.000      │
│ Outstanding   Rp 500.000      │
│                               │
│ Amount        [ Rp 500.000 ]  │
│                               │
│ Method                         │
│ [ Cash ▼ ]                    │
│                               │
│ Reference     [ optional ]    │
│ Notes         [ optional ]    │
│                               │
│ [Cancel]       [Confirm]      │
└───────────────────────────────┘
```

Mobile:

Use bottom sheet/full-screen payment form.

Amount field should be large and numeric-keyboard friendly.

Quick amount buttons:

``` text
[Exact]
[100K]
[250K]
[500K]
```

Only show values that do not exceed outstanding.

------------------------------------------------------------------------

# 37. PAYMENT CONFIRMATION

Before submit:

``` text
Payment Amount
Payment Method
Reference
```

Show:

``` text
Outstanding Before
Payment
Outstanding After
```

Example:

``` text
Outstanding       Rp 500.000
Payment           Rp 500.000
Remaining         Rp 0
```

CTA:

``` text
Konfirmasi Pembayaran
```

After success:

``` text
Pembayaran berhasil
Invoice PAID
```

Actions:

``` text
[Print Receipt]
[View Invoice]
[Close]
```

------------------------------------------------------------------------

# 38. RECEIPT

Receipt should contain:

``` text
GARAGE PRO
Workshop Name
Address
Phone

RECEIPT

Payment Number
Invoice Number
Date

Customer
Vehicle
Plate Number

Total Invoice
Payment Amount
Payment Method
Remaining Balance

Cashier

Thank you
```

V1 should be PDF/print-ready.

Do not hard-code workshop identity; read from settings.

------------------------------------------------------------------------

# 39. PRINT/PDF ARCHITECTURE

Recommended abstraction:

``` text
ReceiptTemplate
InvoiceTemplate
```

Backend can initially return printable HTML or structured JSON.

Future implementation:

``` text
HTML → PDF
```

or browser print:

``` text
window.print()
```

Templates must support:

-   A4 invoice
-   thermal receipt 58mm
-   thermal receipt 80mm

------------------------------------------------------------------------

# 40. REVENUE REPORT INTEGRATION

Revenue should be based on financial transaction definitions.

Recommended report dimensions:

``` text
invoice_date
payment_date
invoice_status
payment_status
payment_method
service revenue
part revenue
discount
tax
net revenue
```

Important distinction:

### Sales/Invoice Revenue

Based on issued invoices.

### Cash Collection

Based on successful payments.

Do not mix these metrics.

Example:

``` text
Invoice issued:
Rp 10,000,000

Cash collected:
Rp 6,000,000

Invoice Revenue = Rp 10,000,000
Cash Collection = Rp 6,000,000
Outstanding = Rp 4,000,000
```

------------------------------------------------------------------------

# 41. DASHBOARD HOOKS

Dashboard should be able to consume:

``` text
today_invoice_count
today_invoice_value
today_payment_count
today_payment_value
today_outstanding
```

Additional:

``` text
monthly_revenue
monthly_collection
unpaid_invoice_count
partial_invoice_count
```

------------------------------------------------------------------------

# 42. API ENDPOINTS

## Invoice

``` http
GET    /api/v1/invoices
POST   /api/v1/work-orders/:id/invoice
GET    /api/v1/invoices/:id
POST   /api/v1/invoices/:id/issue
POST   /api/v1/invoices/:id/void
```

## Invoice Items

Normally read through invoice detail.

Optional:

``` http
GET /api/v1/invoices/:id/items
```

## Payment

``` http
GET  /api/v1/invoices/:id/payments
POST /api/v1/invoices/:id/payments

GET  /api/v1/payments/:id
POST /api/v1/payments/:id/void
POST /api/v1/payments/:id/refund
```

## Receipt

``` http
GET /api/v1/payments/:id/receipt
```

## Revenue

``` http
GET /api/v1/reports/revenue
GET /api/v1/reports/payments
GET /api/v1/reports/outstanding
```

------------------------------------------------------------------------

# 43. API VALIDATION

Invoice creation:

``` text
WO must be READY
No active invoice exists
WO must have billable lines
```

Issue:

``` text
Invoice DRAFT
Grand total >= 0
At least one line
```

Payment:

``` text
Invoice not VOID
Invoice not PAID
Amount > 0
Amount <= outstanding
Valid payment method
```

Void:

``` text
Valid status
Reason required
Permission required
```

------------------------------------------------------------------------

# 44. ERROR CODES

Recommended:

``` text
INVOICE_NOT_FOUND
INVOICE_ALREADY_EXISTS
INVOICE_NOT_ISSUABLE
INVOICE_ALREADY_ISSUED
INVOICE_ALREADY_PAID
INVOICE_VOID
INVOICE_NOT_VOIDABLE

PAYMENT_NOT_FOUND
PAYMENT_INVALID_AMOUNT
PAYMENT_EXCEEDS_OUTSTANDING
PAYMENT_INVALID_METHOD
PAYMENT_ALREADY_VOID
PAYMENT_ALREADY_REFUNDED
PAYMENT_NOT_REFUNDABLE
PAYMENT_IDEMPOTENCY_CONFLICT

WO_NOT_READY_FOR_INVOICE
WO_ALREADY_INVOICED
```

------------------------------------------------------------------------

# 45. BACKEND ARCHITECTURE

Recommended:

``` text
controllers/
  invoice.controller.ts
  payment.controller.ts

services/
  invoice.service.ts
  invoice-calculation.service.ts
  payment.service.ts
  receipt.service.ts
  revenue.service.ts

repositories/
  invoice.repository.ts
  invoice-item.repository.ts
  payment.repository.ts

validators/
  invoice.validator.ts
  payment.validator.ts

routes/
  invoice.routes.ts
  payment.routes.ts

types/
  invoice.types.ts
  payment.types.ts
```

------------------------------------------------------------------------

# 46. INVOICE SERVICE RESPONSIBILITIES

`InvoiceService`:

``` text
createFromWorkOrder()
getById()
list()
issue()
void()
calculate()
```

Never place business logic only inside controller.

Controller:

``` text
parse request
authorize
validate
call service
format response
```

Service:

``` text
business rules
transaction
state transition
calculation
audit
```

Repository:

``` text
database access
queries
locking
persistence
```

------------------------------------------------------------------------

# 47. PAYMENT SERVICE RESPONSIBILITIES

`PaymentService`:

``` text
createPayment()
getPayment()
listInvoicePayments()
voidPayment()
refundPayment()
calculateInvoiceBalance()
```

Critical:

``` text
createPayment()
```

must execute inside a DB transaction.

Pseudo-flow:

``` ts
return sequelize.transaction(async (transaction) => {
  const invoice = await invoiceRepository.findByIdForUpdate(
    invoiceId,
    transaction
  );

  validateInvoicePayable(invoice);

  const balance =
    await paymentRepository.calculateOutstandingForUpdate(
      invoice,
      transaction
    );

  validatePaymentAmount(amount, balance);

  const payment = await paymentRepository.create(
    {...},
    { transaction }
  );

  const totals =
    await invoiceService.recalculatePaymentTotals(
      invoice.id,
      transaction
    );

  await invoiceRepository.updateStatusAndBalance(
    invoice.id,
    totals,
    { transaction }
  );

  if (totals.outstandingAmount === 0) {
    await workOrderService.markPaidAndComplete(
      invoice.workOrderId,
      { transaction }
    );
  }

  await auditService.log(...);

  return payment;
});
```

------------------------------------------------------------------------

# 48. IDEMPOTENCY IMPLEMENTATION

At payment start:

``` text
if idempotency_key exists:
    find payment by key

    if found:
        return existing payment

    else:
        continue
```

Because concurrent requests can race, the database UNIQUE constraint
remains mandatory.

If unique constraint collision occurs:

1.  rollback failed transaction.
2.  retrieve existing payment by idempotency key.
3.  return existing result.

------------------------------------------------------------------------

# 49. FRONTEND DATA MODEL

Suggested TypeScript:

``` ts
export type InvoiceStatus =
  | "DRAFT"
  | "ISSUED"
  | "PARTIAL"
  | "PAID"
  | "VOID";

export type PaymentMethod =
  | "CASH"
  | "TRANSFER"
  | "QRIS"
  | "DEBIT_CARD"
  | "CREDIT_CARD"
  | "OTHER";

export interface InvoiceSummary {
  id: number;
  invoiceNumber: string;
  status: InvoiceStatus;
  subtotal: string;
  lineDiscountTotal: string;
  documentDiscount: string;
  taxAmount: string;
  grandTotal: string;
  paidAmount: string;
  outstandingAmount: string;
}
```

Money should preferably remain strings from API to avoid frontend
floating-point errors.

------------------------------------------------------------------------

# 50. REACT QUERY KEYS

Recommended:

``` ts
invoiceKeys = {
  all: ["invoices"],
  lists: () => [...invoiceKeys.all, "list"],
  list: (params) => [...invoiceKeys.lists(), params],
  detail: (id) => [...invoiceKeys.all, "detail", id],
  payments: (id) => [...invoiceKeys.all, "payments", id],
};
```

After successful payment:

``` text
invalidate invoice detail
invalidate invoice list
invalidate payment list
invalidate dashboard summary
invalidate revenue report
```

------------------------------------------------------------------------

# 51. FRONTEND FLOW

## Cashier Flow

``` text
Invoice List
   ↓
Invoice Detail
   ↓
Review
   ↓
Issue Invoice
   ↓
Receive Payment
   ↓
Confirm
   ↓
Success
   ↓
Print Receipt
```

## From Work Order

``` text
WO READY
 ↓
Create Invoice
 ↓
Invoice Draft
 ↓
Issue
 ↓
Payment
```

------------------------------------------------------------------------

# 52. UX RULES

## Do

-   Show outstanding amount prominently.
-   Use large payment CTA.
-   Show current status.
-   Show payment history.
-   Prevent impossible actions.
-   Confirm void.
-   Confirm refund.
-   Show receipt after successful payment.
-   Make cashier workflow fast.

## Don't

-   Allow manual grand total editing.
-   Allow payment above outstanding.
-   Hide partial payment history.
-   Allow paying VOID invoice.
-   Allow deleting payments.
-   Use floating point for money.
-   Show confusing status transitions.

------------------------------------------------------------------------

# 53. MOBILE CASHIER UX

Payment page should prioritize:

``` text
Invoice
Customer
Vehicle
Outstanding
Amount
Payment Method
Confirm
```

Use:

-   numeric input
-   sticky bottom CTA
-   full-screen modal
-   large status badge
-   simple receipt action

Avoid unnecessary information during payment.

------------------------------------------------------------------------

# 54. SECURITY

Required:

-   authentication
-   authorization
-   rate limiting
-   request ID
-   audit
-   CSRF strategy where applicable
-   secure headers
-   validation
-   SQL injection protection through ORM/query binding
-   no sensitive payment data logging

Do not store:

-   card number
-   CVV
-   PIN
-   bank password
-   QRIS secret
-   payment gateway secret

Only store safe references.

------------------------------------------------------------------------

# 55. TRANSACTION BOUNDARIES

## Create Invoice

Single DB transaction:

``` text
validate WO
create invoice
create invoice items
calculate totals
audit
commit
```

## Issue Invoice

Single DB transaction:

``` text
lock invoice
validate
issue
audit
commit
```

## Payment

Single DB transaction:

``` text
lock invoice
validate
create payment
recalculate
update invoice
update WO when paid
audit
commit
```

## Void Payment

Single DB transaction:

``` text
lock payment
lock invoice
validate
void payment
recalculate
update invoice
reverse related state if required
audit
commit
```

------------------------------------------------------------------------

# 56. TESTING MATRIX

## Invoice Creation

-   READY WO succeeds.
-   Non-READY WO rejected.
-   Duplicate invoice rejected.
-   No billable lines rejected.
-   Snapshot values correct.
-   Totals correct.

## Calculation

Test:

``` text
No discount
Line discount
Document discount
Tax
Multiple lines
Zero tax
Boundary rounding
```

## Payment

-   Exact payment.
-   Partial payment.
-   Multiple payments.
-   Zero payment rejected.
-   Negative rejected.
-   Overpayment rejected.
-   VOID invoice rejected.
-   PAID invoice rejected.
-   Invalid payment method rejected.

## Idempotency

-   Same key once.
-   Same key twice.
-   Concurrent same key.
-   Same key with different payload.

## Concurrency

Test:

``` text
Invoice = 1,000,000

Concurrent:
600,000
600,000
```

Only one request should succeed fully; final balance must never become
negative.

## Void

-   DRAFT invoice void.
-   ISSUED invoice void.
-   PARTIAL invoice protected.
-   PAID invoice protected.
-   Invalid reason rejected.

## Refund

-   Successful refund.
-   Duplicate refund rejected.
-   Invalid payment status rejected.
-   Audit created.

------------------------------------------------------------------------

# 57. INTEGRATION TEST

Scenario:

``` text
Customer
 ↓
Vehicle
 ↓
WO
 ↓
Inspection
 ↓
Service
 ↓
Part
 ↓
Issue Part
 ↓
QC
 ↓
READY
 ↓
Invoice
 ↓
Issue
 ↓
Payment
 ↓
PAID
 ↓
COMPLETED
```

Assertions:

``` text
WO status = COMPLETED
Invoice status = PAID
Paid amount = Grand total
Outstanding = 0
Payment status = SUCCESS
Stock already reduced from issue transaction
Revenue report includes transaction
Audit exists
```

------------------------------------------------------------------------

# 58. REPORTING DATA CONTRACT

Revenue report should expose:

``` json
{
  "period": {
    "from": "2026-09-01",
    "to": "2026-09-30"
  },
  "invoice": {
    "count": 125,
    "value": "125000000"
  },
  "collection": {
    "count": 110,
    "value": "108500000"
  },
  "outstanding": {
    "count": 15,
    "value": "16500000"
  }
}
```

All monetary values returned as strings.

------------------------------------------------------------------------

# 59. DATA CONSISTENCY RULE

Invoice summary fields:

``` text
paid_amount
outstanding_amount
status
```

are projections for fast reads, but payment ledger remains
authoritative.

Reconciliation job can calculate:

``` text
SUM(SUCCESS payments)
```

and compare against invoice.

If mismatch:

``` text
FINANCIAL_RECONCILIATION_MISMATCH
```

must be surfaced for admin/owner review.

Never silently overwrite financial history.

------------------------------------------------------------------------

# 60. DAILY RECONCILIATION

Recommended future scheduled process:

``` text
Every night:
  load issued/partial/paid invoices
  calculate successful payment total
  compare invoice paid_amount
  verify outstanding
  verify status
  report mismatch
```

This is a safety mechanism, not a substitute for transaction integrity.

------------------------------------------------------------------------

# 61. AUDIT TIMELINE UI

Invoice detail should display:

``` text
14:05 Invoice created
14:06 Invoice issued by Admin
14:20 Payment Rp 300.000 CASH
14:45 Payment Rp 500.000 QRIS
14:45 Invoice marked PAID
14:45 WO marked COMPLETED
```

Timeline is read-only.

------------------------------------------------------------------------

# 62. EMPTY / ERROR STATES

No invoices:

``` text
Belum ada invoice
Invoice akan muncul setelah Work Order siap ditagihkan.
```

No payment:

``` text
Belum ada pembayaran
Invoice belum menerima pembayaran.
```

Payment conflict:

``` text
Saldo invoice berubah.
Silakan periksa kembali outstanding sebelum melakukan pembayaran.
```

Invoice already paid:

``` text
Invoice sudah lunas.
Pembayaran baru tidak dapat ditambahkan.
```

------------------------------------------------------------------------

# 63. PERFORMANCE

Invoice list should support:

``` text
pagination
search
filter
sorting
```

Do not load all invoices.

Payment history should be paginated if volume grows.

Use indexed queries for:

``` text
invoice_number
work_order_id
customer_id
status
invoice_date
payment_date
```

------------------------------------------------------------------------

# 64. API RESPONSE STANDARD

Success:

``` json
{
  "success": true,
  "data": {}
}
```

Error:

``` json
{
  "success": false,
  "error": {
    "code": "PAYMENT_EXCEEDS_OUTSTANDING",
    "message": "Payment exceeds outstanding balance.",
    "requestId": "req_..."
  }
}
```

Validation:

``` json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid payment data.",
    "fields": {
      "amount": "Must be greater than zero."
    }
  }
}
```

------------------------------------------------------------------------

# 65. MIGRATION PLAN

If existing migration already contains `invoices`, verify and extend.

Recommended migration order:

``` text
01 create/alter invoices
02 create invoice_items
03 create payments
04 add indexes
05 add unique constraints
06 add foreign keys
07 add payment idempotency
08 seed payment methods/settings if required
```

Do not modify old production transaction data destructively.

------------------------------------------------------------------------

# 66. SEED SETTINGS

Recommended:

``` text
invoice_prefix = INV
payment_prefix = PAY

default_tax_rate = 0

allow_overpayment = false

require_invoice_issue_before_payment = true
require_payment_before_wo_completion = true

receipt_width_default = 80mm
```

Settings should be database-driven.

------------------------------------------------------------------------

# 67. DEFINITION OF DONE

Phase 14 is complete when:

### Database

-   [ ] invoice table ready
-   [ ] invoice_items ready
-   [ ] payments ready
-   [ ] indexes ready
-   [ ] constraints ready
-   [ ] migrations tested

### Backend

-   [ ] invoice service
-   [ ] calculation service
-   [ ] payment service
-   [ ] invoice APIs
-   [ ] payment APIs
-   [ ] idempotency
-   [ ] locking
-   [ ] audit
-   [ ] receipt endpoint
-   [ ] report hooks

### Frontend

-   [ ] invoice list
-   [ ] invoice detail
-   [ ] issue invoice
-   [ ] payment modal
-   [ ] payment history
-   [ ] receipt
-   [ ] void confirmation
-   [ ] status handling
-   [ ] permission handling

### Testing

-   [ ] unit calculation tests
-   [ ] invoice integration tests
-   [ ] payment tests
-   [ ] concurrency test
-   [ ] idempotency test
-   [ ] role authorization test
-   [ ] end-to-end WO → Invoice → Payment test

------------------------------------------------------------------------

# 68. CLAUDE CODE MASTER PROMPT

Use the following prompt directly in Claude Code after the existing
GARAGE PRO phases are implemented.

``` text
You are implementing Phase 14 of GARAGE PRO — Workshop Management System V1.

PROJECT CONTEXT:
GARAGE PRO is a motorcycle workshop management system.

Existing stack:
- Frontend: React + Vite + TypeScript + Tailwind CSS
- React Router
- TanStack Query
- React Hook Form
- Zod
- Axios
- Backend: Node.js + Express + TypeScript
- Sequelize
- MySQL 8+
- JWT authentication
- RBAC
- Pino logging

Existing completed domains:
- Database
- Authentication/RBAC
- Master Data
- Work Order Engine
- Inventory Engine

Your task:
IMPLEMENT INVOICE + PAYMENT ENGINE V1.

DO NOT redesign existing architecture unless required.
Read existing code before modifying it.
Reuse existing:
- auth
- RBAC
- transaction utilities
- audit service
- error handling
- response format
- number generator
- Work Order state machine
- inventory transaction architecture
- design system

CORE FLOW:

READY WO
→ CREATE INVOICE
→ DRAFT
→ ISSUE
→ ISSUED
→ PAYMENT
→ PARTIAL
→ PAYMENT
→ PAID
→ WO PAID
→ WO COMPLETED

INVOICE STATUS:
DRAFT
ISSUED
PARTIAL
PAID
VOID

PAYMENT STATUS:
SUCCESS
VOID
REFUNDED

PAYMENT METHODS:
CASH
TRANSFER
QRIS
DEBIT_CARD
CREDIT_CARD
OTHER

IMPLEMENT:

1. DATABASE
- invoices
- invoice_items
- payments
- migrations
- indexes
- unique constraints
- foreign keys
- idempotency_key
- audit compatibility

Money:
DECIMAL(18,2)

Never use floating-point storage.

2. INVOICE CREATION
Implement:
POST /api/v1/work-orders/:id/invoice

Requirements:
- WO must be READY
- no existing active invoice
- billable lines required
- snapshot customer
- snapshot vehicle
- snapshot service lines
- snapshot part lines
- freeze transaction prices
- server-side calculation

Invoice starts as DRAFT.

3. INVOICE APIs
GET /api/v1/invoices
GET /api/v1/invoices/:id
POST /api/v1/invoices/:id/issue
POST /api/v1/invoices/:id/void

4. CALCULATION
Implement server-side:
line subtotal
line discount
document discount
tax
grand total

Use deterministic decimal arithmetic.

5. PAYMENT
POST /api/v1/invoices/:id/payments

Headers:
Idempotency-Key

Request:
{
  amount,
  payment_method,
  reference_number?,
  notes?
}

Rules:
- invoice must be ISSUED or PARTIAL
- amount > 0
- amount <= outstanding
- overpayment rejected
- payment number server-generated
- successful payment recorded
- invoice recalculated
- status updated
- if fully paid, use existing WO state machine to mark WO paid/completed

6. CONCURRENCY
Payment creation MUST use a DB transaction.

Lock invoice row:
SELECT ... FOR UPDATE

Recalculate outstanding while locked.

Never allow double payment to create negative outstanding.

7. IDEMPOTENCY
Use idempotency_key UNIQUE.

Same key:
- return original payment
- never duplicate payment

Handle race conditions safely.

8. PAYMENT APIs
GET /api/v1/invoices/:id/payments
GET /api/v1/payments/:id
POST /api/v1/payments/:id/void
POST /api/v1/payments/:id/refund

Use controlled authorization.

Never delete financial transactions.

9. INVOICE/PAYMENT AUDIT
Record:
invoice.created
invoice.issued
invoice.voided
payment.created
payment.voided
payment.refunded
invoice.paid
invoice.status_changed
wo.payment_completed

10. FRONTEND
Implement:
- Invoice List
- Invoice Detail
- Issue Invoice action
- Payment Modal/Drawer
- Payment History
- Receipt
- Void confirmation
- status badges
- permission guards

Use existing GARAGE PRO design system.

Mobile payment UX:
- large outstanding amount
- large numeric amount input
- payment method cards/select
- sticky confirm CTA
- success receipt action

11. RECEIPT
Implement receipt data endpoint.

Receipt must contain:
- workshop identity from settings
- payment number
- invoice number
- date
- customer
- vehicle
- plate
- invoice total
- payment
- payment method
- remaining balance
- cashier

Prepare for:
- 58mm
- 80mm
- A4

12. REPORTING
Expose:
- invoice count/value
- payment count/value
- outstanding count/value

Do not mix invoice revenue and cash collection.

13. SECURITY
Use existing:
- authentication
- RBAC
- rate limit
- validation
- audit
- request ID
- secure logging

Never log sensitive payment credentials.

14. TESTS
Implement:
- invoice creation
- invalid WO state
- duplicate invoice
- calculation
- discount
- tax
- partial payment
- full payment
- overpayment
- VOID invoice payment rejection
- PAID invoice payment rejection
- idempotency
- concurrent payment
- payment void
- refund
- authorization
- WO completion integration

Concurrency test:
invoice = 1,000,000
two concurrent payments = 600,000 each

Final financial state must be consistent.

15. CODE QUALITY
Follow existing project conventions.

Do not:
- put business logic in controllers
- bypass services
- directly mutate WO status
- directly mutate inventory
- use floating-point money arithmetic
- hard-code workshop identity
- delete financial records
- bypass audit

Architecture:

controller
→ validator
→ service
→ repository
→ Sequelize/MySQL

Use transactions for financial mutations.

16. FINAL OUTPUT
After implementation:
- summarize files changed
- summarize migrations
- summarize APIs
- summarize tests
- identify any required environment variables
- identify any migration command
- identify any remaining TODO
- ensure TypeScript build succeeds
- ensure lint succeeds
- ensure tests pass

Do not claim completion unless the implementation actually exists and tests pass.
```

------------------------------------------------------------------------

# 69. IMPLEMENTATION ORDER

Claude Code should implement in this order:

``` text
1. Inspect existing repository
2. Verify Work Order Engine
3. Verify existing transaction utilities
4. Verify audit utilities
5. Migration
6. Models
7. Repositories
8. Calculation service
9. Invoice service
10. Payment service
11. Controllers
12. Routes
13. Validators
14. Tests
15. Frontend API hooks
16. Invoice screens
17. Payment UI
18. Receipt UI
19. Integration tests
20. Build/lint/test
```

Never start frontend before backend contracts are stable.

------------------------------------------------------------------------

# 70. FINAL ARCHITECTURE

``` text
                    ┌───────────────────┐
                    │    WORK ORDER     │
                    │      READY        │
                    └─────────┬─────────┘
                              │
                              ▼
                    ┌───────────────────┐
                    │ INVOICE SERVICE   │
                    │ snapshot + calc   │
                    └─────────┬─────────┘
                              │
                              ▼
                    ┌───────────────────┐
                    │      INVOICE      │
                    │      DRAFT        │
                    └─────────┬─────────┘
                              │ ISSUE
                              ▼
                    ┌───────────────────┐
                    │      ISSUED       │
                    └─────────┬─────────┘
                              │
                              ▼
              ┌─────────────────────────────────┐
              │        PAYMENT SERVICE           │
              │ transaction + lock + idempotency│
              └────────────────┬────────────────┘
                               │
                 ┌─────────────┴─────────────┐
                 ▼                           ▼
            PARTIAL                        PAID
                 │                           │
                 │ payment                   │
                 └─────────────┐             │
                               ▼             ▼
                         ┌──────────┐   ┌────────────┐
                         │  PAID    │──▶│ WO COMPLETE│
                         └──────────┘   └────────────┘
```

------------------------------------------------------------------------

# 71. PHASE 14 ACCEPTANCE CRITERIA

A workshop cashier can:

``` text
1. Open READY WO
2. Create invoice
3. Review services and parts
4. Issue invoice
5. Receive partial payment
6. See remaining balance
7. Receive second payment
8. Invoice becomes PAID
9. WO becomes COMPLETED
10. Print receipt
```

Owner can:

``` text
View invoice revenue
View payment collection
View outstanding
View audit trail
```

Admin can:

``` text
Manage invoice
Receive payment
Print receipt
Perform authorized void/refund
```

Mechanic:

``` text
Cannot receive payment
Cannot edit invoice financial data
Can see relevant WO/payment completion status according to permission
```

Warehouse:

``` text
No financial mutation access
```

------------------------------------------------------------------------

# 72. NEXT PHASE

After Invoice + Payment is stable:

``` text
14 ✅ INVOICE + PAYMENT
        ↓
15 🔜 FRONTEND INTEGRATION
        ↓
16    TESTING / QA
        ↓
17    DEPLOYMENT
```

Phase 15 should integrate all completed engines into one cohesive
application:

``` text
AUTH
  ↓
MASTER DATA
  ↓
WORK ORDER
  ↓
INVENTORY
  ↓
INVOICE
  ↓
PAYMENT
  ↓
REPORTS
```

The target is a complete end-to-end GARAGE PRO workflow, not isolated
modules.
