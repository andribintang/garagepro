# GARAGE PRO --- TESTING + QA V1

## Comprehensive Quality Assurance, Security, Integrity & Release Specification

**Project:** GARAGE PRO --- Workshop Management System\
**Phase:** 16 --- Testing / QA\
**Version:** V1.0\
**Status:** Implementation Ready\
**Primary stack:** React + Vite + TypeScript / Node.js + Express +
Sequelize + MySQL 8+\
**Recommended test stack:** Vitest/Jest + Supertest + Playwright +
Testing Library

------------------------------------------------------------------------

# 1. PURPOSE

Phase 16 memastikan GARAGE PRO bukan hanya berjalan secara teknis,
tetapi benar-benar aman digunakan sebagai sistem operasional bengkel.

Target QA:

``` text
CODE
 ↓
UNIT TEST
 ↓
COMPONENT TEST
 ↓
API TEST
 ↓
INTEGRATION TEST
 ↓
DATABASE INTEGRITY
 ↓
CONCURRENCY TEST
 ↓
SECURITY TEST
 ↓
E2E TEST
 ↓
RESPONSIVE TEST
 ↓
UAT
 ↓
RELEASE
```

Fokus terbesar:

1.  Work Order state integrity.
2.  Inventory/stock integrity.
3.  Invoice calculation integrity.
4.  Payment/financial integrity.
5.  Authentication/RBAC.
6.  Auditability.
7.  End-to-end workshop workflow.
8.  Mobile usability untuk mechanic.
9.  Desktop usability untuk admin/owner/warehouse.
10. Production readiness.

------------------------------------------------------------------------

# 2. QA OBJECTIVES

System harus membuktikan:

-   Data tidak hilang.
-   Stock tidak menjadi negatif karena race condition.
-   Payment tidak terduplikasi.
-   Invoice tidak berubah secara ilegal.
-   WO tidak dapat melompati state secara ilegal.
-   User tidak dapat mengakses fitur di luar permission.
-   Financial total selalu server-authoritative.
-   Audit trail tersedia untuk mutation penting.
-   Service history konsisten dengan transaksi.
-   Report tidak mencampur revenue dan cash collection.
-   UI menangani loading/error/conflict dengan benar.
-   Sistem tetap stabil pada penggunaan normal dan beban realistis.

------------------------------------------------------------------------

# 3. QUALITY GATES

Setiap release harus melewati:

``` text
GATE 1 — Build
GATE 2 — Lint / Type Check
GATE 3 — Unit Test
GATE 4 — API / Integration Test
GATE 5 — Security Test
GATE 6 — Data Integrity Test
GATE 7 — E2E
GATE 8 — UAT
GATE 9 — Production Smoke Test
```

Release ditolak jika terdapat:

``` text
CRITICAL defect
HIGH defect pada financial/inventory/security
failed migration
failed authentication
failed core E2E flow
negative stock caused by system defect
incorrect payment balance
```

------------------------------------------------------------------------

# 4. TEST PYRAMID

``` text
                    ┌─────────────┐
                    │    E2E      │
                    │   Small     │
                    └──────┬──────┘
                           │
                 ┌─────────▼─────────┐
                 │ Integration/API   │
                 │      Medium       │
                 └─────────┬─────────┘
                           │
              ┌────────────▼────────────┐
              │ Component / Service     │
              │         Large            │
              └────────────┬────────────┘
                           │
          ┌────────────────▼────────────────┐
          │          Unit Tests             │
          │             Largest             │
          └─────────────────────────────────┘
```

Jangan membuat seluruh coverage bergantung pada E2E.

------------------------------------------------------------------------

# 5. TEST ENVIRONMENT

Recommended:

``` text
Development
QA/Test
Staging
Production
```

Database:

``` text
garage_pro_dev
garage_pro_test
garage_pro_staging
garage_pro
```

Test database harus terisolasi dari production.

------------------------------------------------------------------------

# 6. TEST DATA POLICY

Gunakan synthetic test data.

Contoh:

``` text
Customer:
Budi Test

Phone:
080000000001

Vehicle:
F 0001 TEST

Mechanic:
Mechanic Test

Part:
PART-TEST-001

Service:
SERVICE-TEST-001
```

Jangan menggunakan data pelanggan nyata untuk automated test.

------------------------------------------------------------------------

# 7. TEST DATA FIXTURE

Minimal fixture:

``` text
1 OWNER
1 ADMIN
1 MECHANIC
1 WAREHOUSE USER

3 CUSTOMERS
5 VEHICLES
3 MECHANICS
10 SERVICES
20 PARTS
2 WAREHOUSES
3 SUPPLIERS
```

Transaction fixtures:

``` text
NEW WO
IN_PROGRESS WO
QC WO
READY WO
ISSUED INVOICE
PARTIAL INVOICE
PAID INVOICE
VOID INVOICE
```

------------------------------------------------------------------------

# 8. TEST IDs

Automated test data harus mudah dikenali:

``` text
TEST-CUST-001
TEST-VEH-001
TEST-WO-001
TEST-INV-001
TEST-PAY-001
```

Jangan mengandalkan numeric ID hard-coded jika tidak diperlukan.

------------------------------------------------------------------------

# 9. UNIT TEST SCOPE

Unit test minimum:

``` text
Money calculations
Invoice calculations
Discount calculations
Tax calculations
Status transition rules
Permission helpers
Currency formatter
Date formatter
Validation schemas
Search/filter utilities
```

------------------------------------------------------------------------

# 10. MONEY UNIT TEST

Example:

``` text
qty = 2
unit_price = 50,000

subtotal = 100,000
```

Assert:

``` text
100000.00
```

Test decimal boundary:

``` text
10,000.005
```

Expected:

``` text
10,000.01
```

Do not use native floating point as financial source of truth.

------------------------------------------------------------------------

# 11. INVOICE CALCULATION TESTS

Test:

### Case A --- No discount

``` text
Service 100,000
Part 50,000

Subtotal = 150,000
Discount = 0
Tax = 0
Grand Total = 150,000
```

### Case B --- Line discount

``` text
Service = 100,000
Discount = 10%

Discount = 10,000
Total = 90,000
```

### Case C --- Document discount

``` text
Subtotal = 150,000
Line discount = 10,000
Document discount = 5,000

Taxable base = 135,000
```

### Case D --- Tax

``` text
Taxable base = 100,000
Tax = 11%

Tax = 11,000
Grand Total = 111,000
```

### Case E --- Multiple lines

Ensure sum is deterministic and correctly rounded.

------------------------------------------------------------------------

# 12. INVOICE PROPERTY TESTS

For valid invoice:

``` text
grand_total >= 0
paid_amount >= 0
outstanding_amount >= 0
```

Invariant:

``` text
grand_total =
subtotal
- line_discount_total
- document_discount
+ tax_amount
```

And:

``` text
outstanding_amount =
grand_total - paid_amount
```

------------------------------------------------------------------------

# 13. WORK ORDER STATE TESTS

Allowed transitions:

``` text
NEW → CHECKING
CHECKING → ESTIMATE
ESTIMATE → WAITING_APPROVAL
WAITING_APPROVAL → APPROVED
WAITING_APPROVAL → REJECTED
APPROVED → IN_PROGRESS
IN_PROGRESS → QC
QC → READY
QC → REWORK
REWORK → IN_PROGRESS
READY → INVOICED
INVOICED → PAID
PAID → COMPLETED
```

Every illegal transition must be rejected.

Examples:

``` text
NEW → PAID
NEW → COMPLETED
QC → PAID
COMPLETED → IN_PROGRESS
PAID → NEW
```

Expected:

``` text
409 INVALID_STATE_TRANSITION
```

------------------------------------------------------------------------

# 14. STATE MACHINE PROPERTY

No endpoint may bypass:

``` text
transitionWorkOrder()
```

Direct status mutation from arbitrary controller/repository code should
be treated as architecture defect.

------------------------------------------------------------------------

# 15. MASTER DATA TESTS

Customer:

``` text
Create
Read
Update
Search
Duplicate handling
Soft delete
Reference protection
```

Vehicle:

``` text
Create
Update
Customer relation
Duplicate plate
Soft delete
History
```

Service:

``` text
Create
Update
Category
Price
Active/inactive
Reference protection
```

Part:

``` text
Create
Update
SKU
Category
Selling price
Cost
Minimum stock
Active/inactive
```

------------------------------------------------------------------------

# 16. AUTHENTICATION TESTS

Test:

``` text
Valid login
Invalid password
Unknown user
Inactive user
Missing credentials
Expired access token
Invalid token
Refresh
Logout
Change password
```

Security expectations:

-   no password in response
-   no password in logs
-   no token in audit payload
-   correct 401 handling

------------------------------------------------------------------------

# 17. RBAC TEST MATRIX

Test each role against each protected operation.

Example:

  Operation           OWNER        ADMIN   MECHANIC   WAREHOUSE
  ----------------- ------- ------------ ---------- -----------
  View dashboard          ✓            ✓          ✓           ✓
  Create WO               ✓            ✓     policy          \-
  Edit customer           ✓            ✓         \-          \-
  Add service             ✓            ✓          ✓          \-
  Issue stock             ✓       policy         \-           ✓
  Create invoice          ✓            ✓         \-          \-
  Receive payment         ✓            ✓         \-          \-
  Refund payment          ✓   restricted         \-          \-
  View revenue            ✓            ✓         \-          \-
  User management         ✓   restricted         \-          \-

The exact matrix must follow the RBAC specification already implemented.

------------------------------------------------------------------------

# 18. API TESTING

Use Supertest or equivalent.

Every API category:

``` text
Auth
Customers
Vehicles
Mechanics
Services
Parts
Work Orders
Inspection
Recommendations
Inventory
Purchasing
Invoices
Payments
Reports
Users
Roles
Settings
Audit
```

Test:

``` text
200
201
400
401
403
404
409
422
500 handling
```

------------------------------------------------------------------------

# 19. API CONTRACT TEST

Verify:

``` text
response shape
field names
data types
pagination
error structure
requestId
```

Financial fields should remain exact strings/decimal-safe
representation.

Example:

``` json
{
  "grandTotal": "1250000.00"
}
```

------------------------------------------------------------------------

# 20. CUSTOMER API TEST

``` http
POST /customers
GET /customers
GET /customers/:id
PUT /customers/:id
```

Test:

-   valid create
-   invalid phone
-   duplicate data
-   missing required fields
-   unauthorized access
-   pagination
-   search

------------------------------------------------------------------------

# 21. VEHICLE API TEST

Test:

``` text
customer exists
plate required
duplicate plate behavior
update
history
```

Vehicle cannot reference nonexistent customer.

------------------------------------------------------------------------

# 22. WORK ORDER API TEST

Test:

``` text
create
get
list
inspection
service
part
recommendation
approval
start
QC
rework
ready
invoice
cancel
```

Every transition should be tested from both valid and invalid prior
states.

------------------------------------------------------------------------

# 23. INVENTORY INTEGRITY TEST

Core invariant:

``` text
Current Stock =
Opening Balance
+ IN
- OUT
+ RETURN
+ ADJUSTMENT
+ TRANSFER NET
```

Inventory ledger must remain traceable.

------------------------------------------------------------------------

# 24. STOCK ISSUE TEST

Example:

``` text
Initial stock = 10
Issue = 3
Expected = 7
```

Then:

``` text
Return = 1
Expected = 8
```

All movements must be visible.

------------------------------------------------------------------------

# 25. NEGATIVE STOCK TEST

If:

``` text
Available = 5
Issue = 6
```

Expected:

``` text
422 INSUFFICIENT_STOCK
```

No stock movement should be created.

Transaction must rollback.

------------------------------------------------------------------------

# 26. INVENTORY ROLLBACK TEST

Force an error after stock mutation but before commit.

Expected:

``` text
stock movement rollback
stock balance unchanged
transaction rolled back
```

No partial inventory transaction may remain.

------------------------------------------------------------------------

# 27. STOCK CONCURRENCY TEST

Initial:

``` text
stock = 10
```

Concurrent:

``` text
request A issue 7
request B issue 7
```

Expected:

``` text
only one succeeds
final stock = 3
```

or an equivalent valid serialization where total issue never exceeds 10.

Never:

``` text
stock = -4
```

------------------------------------------------------------------------

# 28. INVOICE CREATION TEST

READY WO:

``` text
services = 200,000
parts = 150,000
```

Create invoice.

Expected:

``` text
grand_total = 350,000
```

Invoice item snapshots must match transaction values.

------------------------------------------------------------------------

# 29. DUPLICATE INVOICE TEST

Run:

``` text
POST create invoice
POST create invoice again
```

Expected:

``` text
first succeeds
second rejected
```

No duplicate active invoice.

------------------------------------------------------------------------

# 30. INVOICE SNAPSHOT TEST

Change master service price after invoice creation.

Example:

``` text
Invoice price = 100,000
Master price = 125,000
```

Invoice must remain:

``` text
100,000
```

------------------------------------------------------------------------

# 31. PAYMENT TEST

Invoice:

``` text
1,000,000
```

Payment:

``` text
400,000
```

Expected:

``` text
paid = 400,000
outstanding = 600,000
status = PARTIAL
```

Second:

``` text
600,000
```

Expected:

``` text
paid = 1,000,000
outstanding = 0
status = PAID
```

------------------------------------------------------------------------

# 32. PAYMENT OVERPAYMENT TEST

Invoice:

``` text
1,000,000
```

Payment:

``` text
1,000,001
```

Expected:

``` text
422 PAYMENT_EXCEEDS_OUTSTANDING
```

No payment created.

------------------------------------------------------------------------

# 33. PAYMENT ZERO/NEGATIVE TEST

Reject:

``` text
0
-1
```

Expected:

``` text
422 PAYMENT_INVALID_AMOUNT
```

------------------------------------------------------------------------

# 34. PAYMENT VOID INVOICE TEST

Invoice:

``` text
VOID
```

Payment attempt must fail.

Expected:

``` text
409 INVOICE_VOID
```

------------------------------------------------------------------------

# 35. PAYMENT PAID INVOICE TEST

Invoice:

``` text
PAID
```

New payment must fail.

Expected:

``` text
409 INVOICE_ALREADY_PAID
```

------------------------------------------------------------------------

# 36. IDEMPOTENCY TEST

Request:

``` text
Idempotency-Key = ABC123
Amount = 500,000
```

Send twice.

Expected:

``` text
one payment
same payment returned
```

Never:

``` text
two payments
```

------------------------------------------------------------------------

# 37. IDEMPOTENCY CONFLICT TEST

Same key:

``` text
ABC123
```

First:

``` text
500,000 CASH
```

Second:

``` text
300,000 CASH
```

Expected behavior:

``` text
reject conflict
```

Do not silently reinterpret the original key.

------------------------------------------------------------------------

# 38. PAYMENT CONCURRENCY TEST

Invoice:

``` text
1,000,000
```

Concurrent requests:

``` text
600,000
600,000
```

Expected:

``` text
one succeeds
one fails due to insufficient remaining balance
```

Final:

``` text
paid <= 1,000,000
outstanding >= 0
```

------------------------------------------------------------------------

# 39. PAYMENT VOID TEST

After successful payment:

``` text
payment.status = SUCCESS
```

Void it.

Expected:

``` text
payment.status = VOID
invoice totals recalculated
audit created
```

Original payment record remains.

------------------------------------------------------------------------

# 40. REFUND TEST

Successful payment:

``` text
500,000
```

Refund:

``` text
500,000
```

Expected:

``` text
payment.status = REFUNDED
```

No deletion.

Audit:

``` text
payment.refunded
```

------------------------------------------------------------------------

# 41. DOUBLE REFUND TEST

Refund same payment twice.

Expected:

``` text
409 PAYMENT_ALREADY_REFUNDED
```

------------------------------------------------------------------------

# 42. FINANCIAL INVARIANTS

At all times:

``` text
paid_amount >= 0
outstanding_amount >= 0
```

And:

``` text
paid_amount <= grand_total
```

unless future credit/overpayment feature explicitly changes policy.

------------------------------------------------------------------------

# 43. REVENUE INTEGRITY

Given:

``` text
Invoice A = 500,000
Invoice B = 750,000
```

Invoice revenue:

``` text
1,250,000
```

Payments:

``` text
A = 500,000
B = 250,000
```

Collection:

``` text
750,000
```

Outstanding:

``` text
500,000
```

Reports must show those values separately.

------------------------------------------------------------------------

# 44. AUDIT TESTING

For each critical mutation verify:

``` text
actor
action
entity
entity_id
timestamp
request_id
before
after
```

Minimum tested events:

``` text
WO status change
stock issue
stock return
stock adjustment
invoice create
invoice issue
invoice void
payment create
payment void
payment refund
```

------------------------------------------------------------------------

# 45. AUDIT IMMUTABILITY

Audit records must not be editable through normal UI/API.

Users should not be able to:

``` text
update audit
delete audit
```

except through controlled infrastructure-level retention procedures if
ever required.

------------------------------------------------------------------------

# 46. SERVICE HISTORY TEST

After completed WO:

``` text
vehicle history
```

must contain:

``` text
date
WO
services
parts
mechanic
amount
mileage
```

Cancelled/rejected WO should not appear as completed service history
unless explicitly defined.

------------------------------------------------------------------------

# 47. E2E MASTER SCENARIO

Use Playwright.

Scenario:

``` text
1. Login ADMIN
2. Create customer
3. Create vehicle
4. Create WO
5. Assign mechanic
6. Open inspection
7. Complete inspection
8. Add service
9. Add part
10. Issue part
11. Approve work
12. Start work
13. Send QC
14. Pass QC
15. READY
16. Create invoice
17. Issue invoice
18. Receive partial payment
19. Verify PARTIAL
20. Receive final payment
21. Verify PAID
22. Verify WO COMPLETED
23. Print/open receipt
24. Open vehicle history
25. Open revenue report
```

------------------------------------------------------------------------

# 48. E2E MECHANIC SCENARIO

Login:

``` text
MECHANIC
```

Expected:

``` text
My Work Orders
```

Test:

``` text
open WO
inspection
service
part
recommendation
QC
```

Verify mechanic cannot:

``` text
receive payment
refund payment
manage users
change system settings
```

------------------------------------------------------------------------

# 49. E2E WAREHOUSE SCENARIO

Login:

``` text
WAREHOUSE
```

Test:

``` text
open inventory
search part
issue
receive
view movement
stock opname
```

Verify no financial mutation access.

------------------------------------------------------------------------

# 50. E2E OWNER SCENARIO

Login:

``` text
OWNER
```

Verify:

``` text
dashboard
revenue
collection
outstanding
mechanic report
stock report
audit
```

------------------------------------------------------------------------

# 51. UI COMPONENT TESTS

Test:

``` text
Button
Input
Select
Combobox
Modal
Drawer
Table
Pagination
Badge
Toast
ConfirmDialog
```

Focus:

-   keyboard navigation
-   disabled state
-   loading state
-   validation
-   accessibility

------------------------------------------------------------------------

# 52. PAYMENT MODAL COMPONENT TEST

Verify:

``` text
amount field
payment method
reference
notes
submit
loading
error
success
```

If outstanding:

``` text
500,000
```

and user enters:

``` text
600,000
```

frontend should show validation, but backend must still reject if
bypassed.

------------------------------------------------------------------------

# 53. FORM VALIDATION TEST

Every major form:

``` text
empty required
invalid format
too short
too long
boundary values
server validation error
```

Error should appear near field.

------------------------------------------------------------------------

# 54. RESPONSIVE TESTING

Breakpoints:

``` text
Mobile ~360px
Mobile ~390px
Tablet ~768px
Desktop ~1280px
Large desktop ~1440px+
```

Test:

``` text
Login
Dashboard
WO detail
Inspection
Part selection
Invoice
Payment
Inventory
Reports
```

------------------------------------------------------------------------

# 55. MOBILE TESTING

Mechanic must be able to:

``` text
open WO
read complaint
inspect
add service
add part
submit
send QC
```

without horizontal layout breakage.

------------------------------------------------------------------------

# 56. DESKTOP TESTING

Admin must be able to:

``` text
search
filter
create WO
manage invoice
receive payment
print receipt
run report
```

without excessive modal stacking.

------------------------------------------------------------------------

# 57. ACCESSIBILITY TESTING

Recommended tools:

``` text
axe
Lighthouse
keyboard-only testing
screen reader spot checks
```

Verify:

-   labels
-   focus
-   contrast
-   semantic headings
-   button names
-   dialog focus
-   error announcements

------------------------------------------------------------------------

# 58. SECURITY TESTING

Test:

``` text
SQL injection inputs
XSS payloads
CSRF strategy
broken access control
IDOR
rate limiting
token expiry
session invalidation
privilege escalation
```

Never test destructive payloads against production.

------------------------------------------------------------------------

# 59. IDOR TEST

Example:

User A has:

``` text
WO ID 100
```

Attempt access:

``` text
GET /work-orders/101
```

Expected:

``` text
403
```

or:

``` text
404
```

according to data visibility policy.

Do not leak existence of restricted resources.

------------------------------------------------------------------------

# 60. BROKEN ACCESS CONTROL

MECHANIC attempts:

``` http
POST /invoices/123/payments
```

Expected:

``` text
403
```

WAREHOUSE attempts:

``` http
POST /payments/123/refund
```

Expected:

``` text
403
```

Frontend hiding button is insufficient.

------------------------------------------------------------------------

# 61. RATE LIMIT TEST

Repeated login attempts:

``` text
many invalid requests
```

Expected:

``` text
rate limit applied
```

No brute-force unlimited access.

------------------------------------------------------------------------

# 62. INPUT SECURITY

Test:

``` text
<script>alert(1)</script>
```

and SQL-like inputs in:

``` text
customer name
notes
complaint
invoice notes
reference
search
```

Expected:

``` text
escaped
validated
parameterized
```

------------------------------------------------------------------------

# 63. LOGGING SECURITY

Verify logs never contain:

``` text
password
JWT
refresh token
card number
CVV
PIN
secrets
```

Safe:

``` text
user ID
request ID
endpoint
error code
```

------------------------------------------------------------------------

# 64. DATABASE INTEGRITY TEST

Verify:

``` text
FK integrity
unique invoice number
unique payment number
unique idempotency key
decimal precision
not-null rules
status values
soft delete behavior
```

------------------------------------------------------------------------

# 65. MIGRATION TEST

Every migration:

``` text
up
down where supported
rebuild database
seed
```

Test from clean DB.

Recommended:

``` bash
npm run db:migrate
npm run db:seed
npm run test
```

No migration should rely on manually altered database state.

------------------------------------------------------------------------

# 66. BACKUP / RESTORE TEST

Before production:

``` text
Create backup
Drop test DB
Restore backup
Run integrity check
```

Verify:

``` text
customers
vehicles
WO
stock
invoices
payments
audit
```

are recoverable.

------------------------------------------------------------------------

# 67. DATA RECONCILIATION TEST

Run reconciliation:

``` text
Invoice paid_amount
vs
SUM successful payments
```

And:

``` text
Invoice outstanding
=
grand_total - successful payments
```

Inventory:

``` text
stock_balances
vs
ledger reconstruction
```

Any mismatch is release blocker for financial/inventory-critical data.

------------------------------------------------------------------------

# 68. PERFORMANCE TEST

Baseline targets for normal workshop usage:

API:

``` text
simple GET p95 < 500ms
normal mutation p95 < 800ms
```

Critical payment:

``` text
p95 < 1s
```

These are practical targets, not hard infrastructure guarantees.

------------------------------------------------------------------------

# 69. LOAD TEST SCENARIOS

Simulate:

``` text
10 concurrent users
25 concurrent users
50 concurrent users
```

Operations:

``` text
search customers
list WO
open WO
search parts
issue parts
create invoices
record payments
dashboard
reports
```

Monitor:

``` text
CPU
RAM
DB connections
query time
error rate
p95
p99
```

------------------------------------------------------------------------

# 70. DATABASE PERFORMANCE

Use:

``` text
EXPLAIN
slow query log
indexes
pagination
```

Check high-frequency queries:

``` text
WO list
Customer search
Part search
Stock balance
Invoice list
Payment history
Dashboard
Revenue report
```

Avoid N+1 queries.

------------------------------------------------------------------------

# 71. REPORT PERFORMANCE

Reports must use server-side aggregation.

Do not:

``` text
download 100,000 transactions
calculate everything in browser
```

Prefer:

``` text
SQL aggregation
pagination
date filters
indexed columns
```

------------------------------------------------------------------------

# 72. UAT PERSONAS

UAT participants:

``` text
Owner
Admin/Cashier
Mechanic
Warehouse
```

Each validates real-world workflow from their perspective.

------------------------------------------------------------------------

# 73. UAT --- OWNER

Tasks:

``` text
Login
Review dashboard
Check revenue
Check collection
Check outstanding
Check mechanic performance
Check stock report
Review audit
```

Acceptance:

``` text
Can understand business condition quickly.
```

------------------------------------------------------------------------

# 74. UAT --- ADMIN

Tasks:

``` text
Create customer
Create vehicle
Create WO
Manage invoice
Receive partial payment
Receive final payment
Print receipt
```

Acceptance:

``` text
Daily transaction can be completed without workaround.
```

------------------------------------------------------------------------

# 75. UAT --- MECHANIC

Tasks:

``` text
Open assigned WO
Perform inspection
Add service
Add parts
Create recommendation
Complete work
Send QC
Handle rework
```

Acceptance:

``` text
Workflow is fast and understandable on mobile.
```

------------------------------------------------------------------------

# 76. UAT --- WAREHOUSE

Tasks:

``` text
Search part
Check stock
Issue
Receive
Return
Adjustment
Opname
```

Acceptance:

``` text
Every movement is traceable and stock balance is understandable.
```

------------------------------------------------------------------------

# 77. DEFECT SEVERITY

## P0 --- Critical

Examples:

``` text
payment duplicated
stock corruption
data loss
authentication bypass
financial total wrong
production database corruption
```

Release:

``` text
BLOCKED
```

## P1 --- High

Examples:

``` text
core workflow broken
invoice cannot issue
payment cannot complete
RBAC serious defect
```

Release:

``` text
BLOCKED
```

## P2 --- Medium

Examples:

``` text
non-critical UI issue
report filter issue
minor workflow inconvenience
```

Release:

``` text
Conditional
```

## P3 --- Low

Examples:

``` text
cosmetic
copy
minor spacing
```

Release:

``` text
Can defer
```

------------------------------------------------------------------------

# 78. BUG REPORT TEMPLATE

``` text
Title:

Severity:

Environment:

User Role:

URL:

Precondition:

Steps:
1.
2.
3.

Expected:

Actual:

Request ID:

Screenshot/Video:

Browser:

Device:

Build Version:
```

------------------------------------------------------------------------

# 79. RELEASE BLOCKERS

Never release if:

``` text
[ ] Payment duplication possible
[ ] Negative stock possible
[ ] Unauthorized financial access possible
[ ] Invoice calculation incorrect
[ ] WO state bypass possible
[ ] Data loss
[ ] Migration failure
[ ] Core E2E failure
[ ] Production backup not verified
[ ] Critical secrets exposed
```

------------------------------------------------------------------------

# 80. PRE-RELEASE CHECKLIST

## Code

``` text
[ ] TypeScript passes
[ ] ESLint passes
[ ] Unit tests pass
[ ] Integration tests pass
[ ] E2E passes
```

## Database

``` text
[ ] Migration clean
[ ] Seed validated
[ ] Indexes validated
[ ] Backup tested
[ ] Restore tested
```

## Security

``` text
[ ] Auth tested
[ ] RBAC tested
[ ] IDOR tested
[ ] Rate limit tested
[ ] Input validation tested
[ ] Sensitive logging reviewed
```

## Business

``` text
[ ] WO lifecycle tested
[ ] Inventory tested
[ ] Invoice tested
[ ] Payment tested
[ ] Refund/void tested
[ ] Reports reconciled
```

------------------------------------------------------------------------

# 81. STAGING SMOKE TEST

Immediately after deployment to staging:

``` text
1. Login
2. Dashboard
3. Create test customer
4. Create test vehicle
5. Create test WO
6. Complete WO
7. Create invoice
8. Payment
9. Receipt
10. Report
11. Logout
```

If this fails:

``` text
STOP RELEASE
```

------------------------------------------------------------------------

# 82. PRODUCTION SMOKE TEST

Use a safe controlled transaction.

Verify:

``` text
HTTPS
login
dashboard
master data read
WO read
inventory read
invoice read
report read
```

For payment production test, use a controlled approved operational
transaction only.

------------------------------------------------------------------------

# 83. OBSERVABILITY

Production monitoring:

``` text
API errors
5xx
401/403 spikes
payment failures
inventory conflicts
DB connection pool
slow queries
CPU
RAM
disk
```

Alert conditions:

``` text
payment failure spike
database unavailable
5xx spike
disk nearly full
backup failure
financial reconciliation mismatch
```

------------------------------------------------------------------------

# 84. HEALTH CHECKS

Backend:

``` http
GET /health
```

Should return:

``` json
{
  "status": "ok"
}
```

Optional:

``` http
GET /health/ready
GET /health/live
```

Readiness can verify:

``` text
database connectivity
required dependencies
```

------------------------------------------------------------------------

# 85. CI/CD QUALITY PIPELINE

Recommended:

``` text
git push
 ↓
install
 ↓
typecheck
 ↓
lint
 ↓
unit test
 ↓
build
 ↓
API/integration test
 ↓
E2E
 ↓
security checks
 ↓
artifact
 ↓
deploy staging
 ↓
smoke test
```

Production deploy should require successful staging validation.

------------------------------------------------------------------------

# 86. GIT STRATEGY

Recommended branches:

``` text
main
develop
feature/*
fix/*
release/*
hotfix/*
```

Pull Request requires:

``` text
tests
review
build
lint
```

Never merge broken build.

------------------------------------------------------------------------

# 87. TEST COVERAGE TARGET

Do not chase coverage percentage alone.

Recommended target:

``` text
Business-critical services: ≥ 90%
Financial calculation: ≥ 95%
State machine: ≥ 95%
Inventory mutation: ≥ 95%
Payment service: ≥ 95%
General application code: ≥ 80%
```

Coverage is a signal, not proof of correctness.

------------------------------------------------------------------------

# 88. CRITICAL DOMAIN COVERAGE

Highest priority:

``` text
WorkOrderService
InventoryService
InvoiceCalculationService
InvoiceService
PaymentService
AuthService
AuthorizationService
```

These should have extensive edge-case tests.

------------------------------------------------------------------------

# 89. CHAOS / FAILURE TESTS

Simulate:

``` text
DB unavailable
network timeout
duplicate request
stale frontend
transaction rollback
server restart
worker restart
invalid token
expired session
```

Expected:

``` text
no financial corruption
no stock corruption
clear user error
recoverable application state
```

------------------------------------------------------------------------

# 90. STALE DATA TEST

Two admin users:

``` text
Admin A opens invoice
Admin B pays invoice
Admin A attempts another payment
```

Expected:

``` text
backend rejects based on current server state
frontend refreshes
no duplicate payment
```

------------------------------------------------------------------------

# 91. DOUBLE CLICK TEST

User double-clicks:

``` text
Confirm Payment
```

Expected:

``` text
one request/payment
```

Use:

``` text
button disabled
mutation state
idempotency key
backend uniqueness
```

All layers should protect against duplication.

------------------------------------------------------------------------

# 92. BROWSER REFRESH TEST

Refresh during:

``` text
WO detail
Invoice detail
Payment success
Report
```

Expected:

``` text
state remains consistent
```

Never rely only on in-memory React state for transaction truth.

------------------------------------------------------------------------

# 93. NETWORK INTERRUPTION TEST

During payment:

``` text
submit
network disconnect
```

Expected:

``` text
no automatic duplicate POST
user can check invoice/payment status
```

------------------------------------------------------------------------

# 94. SECURITY REGRESSION SUITE

Every release should rerun:

``` text
auth
RBAC
IDOR
input injection
XSS
rate limit
session expiry
protected endpoints
```

Security regressions are release blockers when critical.

------------------------------------------------------------------------

# 95. FINANCIAL REGRESSION SUITE

Every release:

``` text
invoice calculation
discount
tax
partial payment
full payment
overpayment
void
refund
reconciliation
```

No financial release without passing suite.

------------------------------------------------------------------------

# 96. INVENTORY REGRESSION SUITE

Every release:

``` text
issue
return
receive
adjust
opname
transfer
low stock
concurrency
rollback
reconciliation
```

------------------------------------------------------------------------

# 97. COMPLETE BUSINESS REGRESSION

The following must always pass:

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
Inventory Issue
 ↓
QC
 ↓
Invoice
 ↓
Payment
 ↓
Completed
 ↓
History
 ↓
Report
```

This is the GARAGE PRO V1 Golden Path.

------------------------------------------------------------------------

# 98. GOLDEN PATH ACCEPTANCE

The Golden Path is accepted only when:

``` text
Customer exists
Vehicle belongs to customer
WO created
Mechanic assigned
Inspection saved
Service saved
Part saved
Stock reduced correctly
QC passed
WO READY
Invoice created
Invoice issued
Payment successful
Invoice PAID
WO COMPLETED
Receipt available
Vehicle history updated
Revenue report updated
Audit exists
```

------------------------------------------------------------------------

# 99. FINAL QA REPORT FORMAT

At the end of QA produce:

``` text
GARAGE PRO V1 QA REPORT

Build:
PASS / FAIL

Lint:
PASS / FAIL

Unit:
PASS / FAIL

API:
PASS / FAIL

Integration:
PASS / FAIL

E2E:
PASS / FAIL

Security:
PASS / FAIL

Inventory Integrity:
PASS / FAIL

Financial Integrity:
PASS / FAIL

UAT:
PASS / FAIL

Open P0:
0

Open P1:
0

Open P2:
X

Open P3:
X

Release Recommendation:
GO / NO-GO
```

------------------------------------------------------------------------

# 100. CLAUDE CODE MASTER QA PROMPT

``` text
You are the QA and testing engineer for GARAGE PRO — Workshop Management System V1.

IMPORTANT:
This is an existing application.
Do NOT rewrite business logic simply to make tests pass.
First inspect:
- frontend
- backend
- database migrations
- authentication
- RBAC
- Work Order Engine
- Inventory Engine
- Invoice + Payment Engine
- Frontend Integration

GOAL:
Create and execute a comprehensive automated and manual QA suite for GARAGE PRO V1.

TECHNOLOGY:
Frontend:
- React
- Vite
- TypeScript
- TanStack Query
- React Hook Form
- Zod
- Tailwind

Backend:
- Node.js
- Express
- TypeScript
- Sequelize
- MySQL

Testing:
- Vitest or existing unit test framework
- Supertest for API
- Testing Library
- Playwright for E2E

DO NOT replace the existing test framework if one already exists unless necessary.

TEST PRIORITY:

1. Financial integrity
2. Inventory integrity
3. Work Order state integrity
4. Authentication/RBAC
5. Auditability
6. End-to-end workflow
7. UI
8. Performance

PHASE 1:
Inspect repository and identify:
- test setup
- scripts
- modules
- routes
- services
- database test strategy

PHASE 2:
Implement unit tests for:
- money
- invoice calculation
- discount
- tax
- status transitions
- permissions
- validation
- formatters

PHASE 3:
Implement API tests for:
- auth
- customers
- vehicles
- WO
- inventory
- invoices
- payments
- reports

PHASE 4:
Implement integration tests for:
- WO → inventory
- WO → invoice
- invoice → payment
- payment → WO completion
- vehicle history
- reports

PHASE 5:
Implement concurrency tests.

CRITICAL PAYMENT TEST:
Invoice = 1,000,000
Two concurrent payments = 600,000 each.

Expected:
- only one can consume the available balance
- final paid <= 1,000,000
- outstanding >= 0
- no duplicate/negative state

CRITICAL INVENTORY TEST:
Stock = 10
Two concurrent issues = 7 each.

Expected:
- total issued <= 10
- final stock >= 0
- one transaction may fail
- no ledger corruption

IDEMPOTENCY TEST:
Send same payment request twice with same Idempotency-Key.

Expected:
- one payment
- same result returned

Test conflicting same key payload.

FINANCIAL INVARIANTS:
grand_total >= 0
paid_amount >= 0
outstanding_amount >= 0
paid_amount <= grand_total
outstanding = grand_total - paid_amount

INVOICE SNAPSHOT:
Change master price after invoice creation.
Invoice must remain unchanged.

WORK ORDER:
Test every valid transition.
Test every important invalid transition.

RBAC:
Test every role against protected endpoints.
Do not rely only on frontend PermissionGate.

SECURITY:
Test:
- IDOR
- broken access control
- invalid token
- expired token
- rate limit
- XSS input
- SQL injection-like input
- sensitive logging

E2E GOLDEN PATH:
login
→ customer
→ vehicle
→ WO
→ inspection
→ service
→ part
→ issue stock
→ QC
→ READY
→ invoice
→ issue
→ partial payment
→ final payment
→ PAID
→ COMPLETED
→ receipt
→ vehicle history
→ report

RESPONSIVE:
Test mobile:
- mechanic workflow
- inspection
- parts
- QC

Test desktop:
- admin
- cashier
- warehouse
- owner

FAILURE TESTS:
- network timeout
- duplicate click
- stale data
- DB rollback
- expired session
- server error

QUALITY GATES:
Run:
npm run typecheck
npm run lint
npm run test
npm run build
npm run test:e2e

If project uses different commands, inspect package.json and use existing scripts.

Do not claim tests pass unless they actually pass.

FINAL REPORT:
Provide:
1. test files created/changed
2. test suites
3. total tests
4. passed
5. failed
6. skipped
7. coverage
8. security findings
9. financial findings
10. inventory findings
11. open defects
12. release recommendation GO/NO-GO

Any P0 or P1 defect in financial, inventory, authentication, or core workflow means NO-GO.
```

------------------------------------------------------------------------

# 101. QA MASTER CHECKLIST

``` text
CODE
[ ] Typecheck
[ ] Lint
[ ] Build

UNIT
[ ] Money
[ ] Invoice
[ ] Payment
[ ] Inventory
[ ] WO State
[ ] RBAC

API
[ ] Auth
[ ] Customer
[ ] Vehicle
[ ] WO
[ ] Inventory
[ ] Invoice
[ ] Payment
[ ] Reports

INTEGRATION
[ ] WO + Inventory
[ ] WO + Invoice
[ ] Invoice + Payment
[ ] Payment + WO
[ ] History
[ ] Reports

CONCURRENCY
[ ] Payment
[ ] Inventory
[ ] Idempotency

SECURITY
[ ] RBAC
[ ] IDOR
[ ] XSS
[ ] SQL Injection
[ ] Rate Limit
[ ] Token
[ ] Logging

UI
[ ] Desktop
[ ] Mobile
[ ] Loading
[ ] Empty
[ ] Error
[ ] Forbidden
[ ] Validation

E2E
[ ] Golden Path
[ ] Mechanic
[ ] Warehouse
[ ] Admin
[ ] Owner

DATA
[ ] Migration
[ ] Backup
[ ] Restore
[ ] Reconciliation

UAT
[ ] Owner
[ ] Admin
[ ] Mechanic
[ ] Warehouse

RELEASE
[ ] P0 = 0
[ ] P1 = 0
[ ] Staging smoke PASS
[ ] Production checklist PASS
```

------------------------------------------------------------------------

# 102. RELEASE DECISION

GARAGE PRO V1 should use:

``` text
GO
```

only when:

``` text
Build PASS
AND
Core tests PASS
AND
Financial integrity PASS
AND
Inventory integrity PASS
AND
Security critical tests PASS
AND
Golden Path PASS
AND
UAT PASS
AND
P0 = 0
AND
P1 = 0
```

Otherwise:

``` text
NO-GO
```

------------------------------------------------------------------------

# 103. NEXT PHASE

After QA is passed:

``` text
14 ✅ Invoice + Payment
15 ✅ Frontend Integration
16 ✅ Testing / QA Specification
17 🔜 Deployment + Production Operations
```

Phase 17 will define:

``` text
Production Server
Nginx
SSL/HTTPS
Node.js
PM2
MySQL
Environment Variables
Database Migration
Backup
Restore
Monitoring
Logging
Domain/DNS
CI/CD
Security Hardening
Deployment Runbook
Rollback
Disaster Recovery
Production Smoke Test
```

The objective is to take the tested GARAGE PRO application from staging
into a reliable production environment.
