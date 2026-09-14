# GARAGE PRO --- PROJECT ARCHITECTURE V1

**Document Type:** Technical Architecture Specification\
**Product:** GARAGE PRO --- Workshop Management System\
**Version:** V1.0\
**Status:** Development Ready\
**Date:** 14 September 2026

------------------------------------------------------------------------

# 1. PURPOSE

Dokumen ini menjadi blueprint teknis implementasi GARAGE PRO V1.

Tujuan:

-   memastikan frontend dan backend memiliki struktur yang jelas
-   memisahkan UI, business logic, database dan infrastructure
-   memudahkan pengembangan dengan Claude Code
-   memudahkan maintenance
-   mencegah business rule tersebar di frontend
-   menyediakan fondasi yang dapat berkembang ke V2

Architecture harus mengikuti:

``` text
UI
 ↓
Feature Layer
 ↓
API Client
 ↓
Backend Controller
 ↓
Application Service
 ↓
Repository / ORM
 ↓
MySQL
```

Business rule utama berada di backend.

------------------------------------------------------------------------

# 2. TECHNOLOGY STACK

## Frontend

``` text
React
Vite
TypeScript
Tailwind CSS
React Router
TanStack Query
React Hook Form
Zod
Axios
Lucide React
```

## Backend

``` text
Node.js
Express
TypeScript
Sequelize
MySQL 8+
JWT
bcrypt / Argon2
Zod
Pino
```

## Infrastructure

``` text
Linux Ubuntu
Nginx
PM2
MySQL
Git
GitHub
HTTPS
```

Optional:

``` text
Docker
Docker Compose
```

------------------------------------------------------------------------

# 3. HIGH LEVEL ARCHITECTURE

``` text
                    GARAGE PRO
                         │
              ┌──────────┴──────────┐
              │                     │
          Frontend               Backend
              │                     │
        React + Vite            Express API
              │                     │
        TanStack Query         Controller
              │                     │
          API Client           Service Layer
              │                     │
              └──────────┬──────────┘
                         │
                    Repository
                         │
                     Sequelize
                         │
                       MySQL
```

------------------------------------------------------------------------

# 4. REQUEST FLOW

Example: issue spare part.

``` text
Mechanic
   ↓
Parts UI
   ↓
useIssuePart()
   ↓
API Client
   ↓
POST /work-orders/:id/parts/:itemId/issue
   ↓
Auth Middleware
   ↓
Permission Middleware
   ↓
WO Controller
   ↓
WO Service
   ↓
Inventory Service
   ↓
Database Transaction
   ↓
Stock Movement
   ↓
Work Order Part Update
   ↓
Commit
   ↓
API Response
   ↓
TanStack Query Cache Update
   ↓
UI
```

------------------------------------------------------------------------

# 5. CORE ARCHITECTURE PRINCIPLES

## 5.1 Backend Owns Business Rules

Frontend may validate UX.

Backend must validate business rules.

Example:

``` text
Frontend:
qty > 0

Backend:
qty > 0
part exists
warehouse exists
WO valid
WO status allows issue
user has permission
stock sufficient
transaction safe
```

## 5.2 Database Is Source of Truth

Inventory, payment, invoice and WO state must not rely on frontend
calculations.

## 5.3 Explicit Transactions

Use DB transactions for operations that change multiple related records.

Examples:

``` text
Receive Purchase
Issue Part
Return Part
Stock Adjustment
Stock Opname Posting
Create Invoice
Record Payment
```

## 5.4 Immutable Financial / Stock History

Do not overwrite historical transactions.

Use reversal/adjustment transactions where required.

------------------------------------------------------------------------

# 6. SYSTEM MODULES

``` text
AUTH
CUSTOMERS
VEHICLES
MECHANICS
SERVICES
PARTS
WORK ORDERS
INSPECTION
RECOMMENDATIONS
INVENTORY
PURCHASING
INVOICES
PAYMENTS
REPORTS
USERS
ROLES
SETTINGS
AUDIT
```

------------------------------------------------------------------------

# 7. FRONTEND ARCHITECTURE

Use feature-oriented architecture.

``` text
frontend/
└── src/
    ├── app/
    ├── assets/
    ├── components/
    ├── features/
    ├── hooks/
    ├── layouts/
    ├── lib/
    ├── pages/
    ├── routes/
    ├── services/
    ├── stores/
    ├── types/
    ├── utils/
    └── main.tsx
```

------------------------------------------------------------------------

# 8. FRONTEND DIRECTORY DETAIL

``` text
src/
├── app/
│   ├── App.tsx
│   ├── providers.tsx
│   └── config.ts
│
├── assets/
│
├── components/
│   ├── ui/
│   ├── forms/
│   ├── tables/
│   ├── feedback/
│   ├── navigation/
│   └── charts/
│
├── features/
│   ├── auth/
│   ├── dashboard/
│   ├── customers/
│   ├── vehicles/
│   ├── mechanics/
│   ├── services/
│   ├── parts/
│   ├── work-orders/
│   ├── inventory/
│   ├── purchasing/
│   ├── invoices/
│   ├── payments/
│   ├── reports/
│   └── settings/
│
├── hooks/
│
├── layouts/
│   ├── AppLayout.tsx
│   ├── AuthLayout.tsx
│   └── MechanicLayout.tsx
│
├── lib/
│   ├── api.ts
│   ├── query-client.ts
│   └── permissions.ts
│
├── routes/
│   ├── index.tsx
│   ├── protected-route.tsx
│   └── permission-route.tsx
│
├── stores/
│
├── types/
│
└── utils/
```

------------------------------------------------------------------------

# 9. FEATURE MODULE STRUCTURE

Example:

``` text
features/work-orders/

├── api/
│   ├── work-orders.api.ts
│   ├── inspection.api.ts
│   └── recommendations.api.ts
│
├── components/
│   ├── WorkOrderCard.tsx
│   ├── WorkOrderStatus.tsx
│   ├── WorkOrderSummary.tsx
│   ├── WorkOrderItems.tsx
│   └── WorkOrderActionBar.tsx
│
├── hooks/
│   ├── useWorkOrders.ts
│   ├── useWorkOrder.ts
│   ├── useCreateWorkOrder.ts
│   └── useWorkOrderActions.ts
│
├── pages/
│   ├── WorkOrderListPage.tsx
│   ├── WorkOrderCreatePage.tsx
│   └── WorkOrderDetailPage.tsx
│
├── schemas/
│   └── work-order.schema.ts
│
├── types/
│   └── work-order.types.ts
│
└── index.ts
```

------------------------------------------------------------------------

# 10. FRONTEND STATE MANAGEMENT

Use three categories.

## Server State

Use:

``` text
TanStack Query
```

For: - customers - vehicles - WO - services - parts - inventory -
purchases - invoices - payments - reports

## UI State

Use React state or lightweight store.

Examples:

``` text
sidebar open
modal open
selected filters
active tab
drawer state
```

## Auth State

Central auth provider/store:

``` text
current user
access token
permissions
session status
```

Do not duplicate server state into global state unnecessarily.

------------------------------------------------------------------------

# 11. TANSTACK QUERY RULES

Query keys must be predictable.

Examples:

``` text
['customers', filters]
['customer', customerId]
['vehicles', filters]
['vehicle', vehicleId]
['work-orders', filters]
['work-order', workOrderId]
['parts', filters]
['inventory', filters]
['invoice', invoiceId]
```

After mutation: - invalidate affected queries - update cache where
simple - refetch authoritative totals when necessary

------------------------------------------------------------------------

# 12. API CLIENT

Central Axios instance:

``` text
src/lib/api.ts
```

Responsibilities: - base URL - auth header - timeout - response
normalization - error normalization - refresh/session handling if
implemented

Example:

``` text
api.get()
api.post()
api.put()
api.delete()
```

Feature APIs should use this client.

------------------------------------------------------------------------

# 13. AUTHENTICATION ARCHITECTURE

Login:

``` text
POST /api/v1/auth/login
```

Server returns authenticated session/token.

Recommended V1 approach:

``` text
Short-lived access token
Refresh/session mechanism
```

If JWT is stored client-side, avoid exposing long-lived tokens
unnecessarily.

For browser security, prefer: - secure HTTP-only cookies for
refresh/session credentials - HTTPS - short-lived access credentials

------------------------------------------------------------------------

# 14. AUTHORIZATION

Permission format:

``` text
resource.action
```

Examples:

``` text
customers.view
customers.create
customers.update

work_orders.view
work_orders.create
work_orders.approve
work_orders.issue_part
work_orders.complete

inventory.view
inventory.issue
inventory.adjust
inventory.opname

payments.create
payments.view

reports.view
```

Backend must enforce permissions.

Frontend hides unavailable actions for UX but is not the security
boundary.

------------------------------------------------------------------------

# 15. ROLE MODEL

Roles:

``` text
OWNER
ADMIN
MECHANIC
WAREHOUSE
```

Role → Permissions:

``` text
roles
   ↓
role_permissions
   ↓
permissions
```

User:

``` text
user.role
```

V1 assumes one primary role per user unless later extended to
many-to-many roles.

------------------------------------------------------------------------

# 16. BACKEND DIRECTORY

Recommended:

``` text
backend/
└── src/
    ├── app.ts
    ├── server.ts
    │
    ├── config/
    ├── middlewares/
    ├── routes/
    ├── controllers/
    ├── services/
    ├── repositories/
    ├── models/
    ├── migrations/
    ├── seeders/
    ├── validators/
    ├── policies/
    ├── utils/
    ├── errors/
    ├── logger/
    ├── types/
    └── docs/
```

------------------------------------------------------------------------

# 17. BACKEND RESPONSIBILITIES

## Controller

Responsible for: - request parsing - calling service - response
formatting

Controller must be thin.

## Service

Responsible for: - business rules - workflow - transactions -
orchestration

## Repository

Responsible for: - database access - queries - Sequelize operations

## Validator

Responsible for: - request schema - type validation - field validation

------------------------------------------------------------------------

# 18. CONTROLLER PATTERN

Bad:

``` text
Controller
  ├── SQL
  ├── business rule
  ├── stock calculation
  └── response
```

Good:

``` text
Controller
   ↓
Service
   ↓
Repository
```

------------------------------------------------------------------------

# 19. SERVICE LAYER

Examples:

``` text
AuthService
CustomerService
VehicleService
WorkOrderService
InspectionService
RecommendationService
InventoryService
PurchaseService
InvoiceService
PaymentService
ReportService
UserService
AuditService
```

------------------------------------------------------------------------

# 20. CRITICAL DOMAIN SERVICES

## WorkOrderService

Responsibilities: - create WO - transition status - assign mechanic -
add service - add part - add recommendation - approve - start - QC -
rework - ready - invoice - complete - cancel

## InventoryService

Responsibilities: - stock balance - issue - return - receive -
adjustment - opname - movement ledger

## PaymentService

Responsibilities: - validate invoice - validate payment - create
payment - update invoice status - completion rules

------------------------------------------------------------------------

# 21. WORK ORDER STATE MACHINE

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
 ├── REWORK → IN_PROGRESS
 ↓
READY
 ↓
INVOICED
 ↓
PAID
 ↓
COMPLETED
```

Cancellation allowed only where business rules permit.

Backend must reject invalid transitions.

------------------------------------------------------------------------

# 22. STATE TRANSITION SERVICE

Use explicit function:

``` text
transitionWorkOrder(
  workOrderId,
  targetStatus,
  actor,
  metadata
)
```

Never allow:

``` text
PATCH status = "PAID"
```

without validation.

------------------------------------------------------------------------

# 23. INVENTORY ARCHITECTURE

Inventory is ledger-based.

Core:

``` text
stock_movements
```

Movement types:

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

Balance:

``` text
Opening
+ IN
- OUT
= Current Balance
```

------------------------------------------------------------------------

# 24. INVENTORY TRANSACTION

Issue flow:

``` text
BEGIN TRANSACTION

Lock stock row
      ↓
Check available quantity
      ↓
Create stock movement
      ↓
Update stock balance/cache if used
      ↓
Update WO part
      ↓
Commit

ROLLBACK on error
```

Use row locking to prevent race conditions.

------------------------------------------------------------------------

# 25. PURCHASE FLOW

``` text
Draft PO
   ↓
Submitted
   ↓
Ordered
   ↓
Partially Received
   ↓
Received
```

Receiving creates inventory movements.

Purchase cost becomes historical cost for the received transaction.

------------------------------------------------------------------------

# 26. INVOICE ARCHITECTURE

Invoice is generated from WO items.

Snapshot:

``` text
service name
part name
SKU
qty
unit price
discount
subtotal
```

Once issued: - historical values are frozen.

------------------------------------------------------------------------

# 27. PAYMENT ARCHITECTURE

Payment must: 1. validate invoice 2. validate amount 3. create payment
4. calculate paid amount 5. calculate outstanding 6. update payment
status 7. optionally transition WO 8. audit

Example:

``` text
Invoice: 190.000
Payment: 100.000
Outstanding: 90.000
Status: PARTIAL
```

Full payment:

``` text
Outstanding: 0
Status: PAID
```

------------------------------------------------------------------------

# 28. REPORTING ARCHITECTURE

Reports should use optimized queries.

Avoid loading all transactions into Node.js and calculating everything
in memory.

Examples: - revenue aggregation in SQL - service ranking in SQL - part
consumption in SQL - mechanic performance in SQL

For V1, reports can query transactional tables directly.

Future V2 may introduce: - reporting views - materialized aggregates -
analytics database

------------------------------------------------------------------------

# 29. AUDIT ARCHITECTURE

Audit important actions:

``` text
LOGIN
CREATE
UPDATE
DELETE/DEACTIVATE
STATUS_CHANGE
STOCK_ISSUE
STOCK_RETURN
STOCK_ADJUSTMENT
OPNAME_POST
INVOICE_CREATE
PAYMENT_CREATE
PAYMENT_VOID
USER_PERMISSION_CHANGE
SETTINGS_CHANGE
```

Record:

``` text
user
action
entity
entity_id
old_values
new_values
ip
user_agent
timestamp
```

------------------------------------------------------------------------

# 30. ERROR ARCHITECTURE

Create custom errors:

``` text
AppError
ValidationError
AuthenticationError
AuthorizationError
NotFoundError
ConflictError
BusinessRuleError
InsufficientStockError
InvalidStatusTransitionError
```

API response:

``` json
{
  "success": false,
  "error": {
    "code": "INSUFFICIENT_STOCK",
    "message": "Stok tidak mencukupi."
  }
}
```

Do not expose stack traces in production.

------------------------------------------------------------------------

# 31. VALIDATION ARCHITECTURE

Frontend:

``` text
React Hook Form + Zod
```

Backend:

``` text
Zod
```

Validation layers:

``` text
UI validation
      ↓
API schema validation
      ↓
Business validation
      ↓
Database constraints
```

All four have different responsibilities.

------------------------------------------------------------------------

# 32. DATABASE ACCESS

Sequelize models should represent database structure.

Repositories should encapsulate common queries.

Avoid direct model access from controllers.

------------------------------------------------------------------------

# 33. DATABASE TRANSACTION POLICY

Use transaction for: - multiple writes - inventory - payment - invoice
generation - stock opname - receiving

Example:

``` text
sequelize.transaction(async (transaction) => {
   ...
});
```

All participating operations receive the transaction object.

------------------------------------------------------------------------

# 34. DATABASE CONSTRAINTS

Enforce at DB level where appropriate:

``` text
UNIQUE SKU
UNIQUE WO number
UNIQUE invoice number
UNIQUE username
UNIQUE role name
FOREIGN KEY
NOT NULL
CHECK where supported
```

Application validation does not replace database constraints.

------------------------------------------------------------------------

# 35. ID STRATEGY

Recommended:

``` text
BIGINT UNSIGNED
```

Internal IDs.

Human-readable numbers:

``` text
WO-20260914-0001
INV-20260914-0001
PO-20260914-0001
PAY-20260914-0001
```

Generate server-side.

------------------------------------------------------------------------

# 36. NUMBER GENERATOR

Create centralized service:

``` text
DocumentNumberService
```

Methods:

``` text
generateWO()
generateInvoice()
generatePurchase()
generatePayment()
```

Must be concurrency-safe.

------------------------------------------------------------------------

# 37. SOFT DELETE

Use soft delete for master data:

``` text
customers
vehicles
services
parts
suppliers
users
```

Transaction records: - do not hard delete.

Use: - void - cancel - reverse - deactivate

depending on domain.

------------------------------------------------------------------------

# 38. FRONTEND ROUTING

Use protected routes.

``` text
PublicRoute
ProtectedRoute
PermissionRoute
```

Example:

``` tsx
<Route
  path="/reports/revenue"
  element={
    <PermissionRoute permission="reports.view">
      <RevenueReportPage />
    </PermissionRoute>
  }
/>
```

------------------------------------------------------------------------

# 39. PAGE ACCESS

Frontend route guards improve UX.

Backend middleware remains mandatory.

``` text
Frontend Guard ≠ Security
Backend Authorization = Security
```

------------------------------------------------------------------------

# 40. API ROUTING

Structure:

``` text
/api/v1
├── /auth
├── /customers
├── /vehicles
├── /mechanics
├── /services
├── /parts
├── /work-orders
├── /inventory
├── /suppliers
├── /purchases
├── /invoices
├── /payments
├── /reports
├── /users
├── /roles
├── /settings
└── /audit-logs
```

------------------------------------------------------------------------

# 41. MIDDLEWARE STACK

Recommended order:

``` text
Request ID
 ↓
Security Headers
 ↓
CORS
 ↓
Body Parser
 ↓
Logger
 ↓
Authentication
 ↓
Authorization
 ↓
Validation
 ↓
Controller
 ↓
Error Handler
```

Not every route requires authentication.

------------------------------------------------------------------------

# 42. SECURITY

Minimum V1:

``` text
HTTPS
Password hashing
JWT/session security
Rate limiting on login
Input validation
SQL injection protection
CORS policy
Security headers
Authorization
Audit logs
No secrets in source
Environment variables
```

Never store: - plain passwords - API secrets in Git - database passwords
in frontend

------------------------------------------------------------------------

# 43. ENVIRONMENT VARIABLES

Frontend:

``` text
VITE_API_BASE_URL
```

Backend:

``` text
NODE_ENV
PORT
APP_URL
FRONTEND_URL

DB_HOST
DB_PORT
DB_NAME
DB_USER
DB_PASSWORD

JWT_SECRET
JWT_EXPIRES_IN

LOG_LEVEL
```

Production secrets must come from server environment/secret management.

------------------------------------------------------------------------

# 44. ENVIRONMENT FILES

``` text
.env
.env.example
.env.development
.env.production
```

`.env` and production secrets must not be committed.

`.env.example` contains names only.

------------------------------------------------------------------------

# 45. PROJECT ROOT

Recommended monorepo:

``` text
garage-pro/
├── frontend/
├── backend/
├── docs/
├── scripts/
├── docker/
├── .github/
├── .gitignore
├── README.md
└── docker-compose.yml
```

------------------------------------------------------------------------

# 46. DOCS STRUCTURE

``` text
docs/
├── MASTER_DEVELOPMENT_SPECIFICATION.md
├── DATABASE_DESIGN.md
├── API_SPECIFICATION.md
├── UI_UX_SCREEN_BIBLE.md
├── DESIGN_SYSTEM.md
├── PROJECT_ARCHITECTURE.md
├── BUSINESS_RULES.md
├── DEPLOYMENT.md
└── CLAUDE_CODE_PROMPT.md
```

------------------------------------------------------------------------

# 47. GIT STRATEGY

Branches:

``` text
main
develop
feature/*
fix/*
hotfix/*
```

Example:

``` text
feature/work-order-module
feature/inventory-module
fix/payment-status
```

Commit convention:

``` text
feat:
fix:
refactor:
docs:
test:
chore:
```

Examples:

``` text
feat: add work order creation flow
feat: implement inventory issue transaction
fix: prevent duplicate payment submission
test: add work order transition tests
```

------------------------------------------------------------------------

# 48. DEVELOPMENT SEQUENCE

## Phase 1 --- Foundation

``` text
Repository
Environment
TypeScript
Lint
Formatter
Base UI
Auth
Database connection
```

## Phase 2 --- Master Data

``` text
Users
Roles
Customers
Vehicles
Mechanics
Service Categories
Services
Part Categories
Parts
Warehouses
Suppliers
```

## Phase 3 --- Workshop

``` text
Work Order
Inspection
Services
Parts
Recommendations
Workshop Board
QC
```

## Phase 4 --- Inventory

``` text
Receiving
Issue
Return
Adjustment
Opname
Movement
Low Stock
```

## Phase 5 --- Transactions

``` text
Invoice
Payment
Completion
```

## Phase 6 --- Reporting

``` text
Dashboard
Revenue
WO
Services
Parts
Mechanics
Stock
```

## Phase 7 --- Hardening

``` text
Testing
Security
Performance
PWA
Backup
Monitoring
Deployment
```

------------------------------------------------------------------------

# 49. TEST ARCHITECTURE

Frontend: - unit tests - component tests - integration tests

Backend: - service unit tests - API integration tests - database
integration tests

E2E:

``` text
Create Customer
→ Vehicle
→ WO
→ Inspection
→ Service
→ Part
→ Issue
→ QC
→ Invoice
→ Payment
→ Complete
```

------------------------------------------------------------------------

# 50. TESTING TOOLS

Recommended:

``` text
Vitest
React Testing Library
Supertest
Playwright
```

Optional:

``` text
MSW
```

------------------------------------------------------------------------

# 51. CRITICAL TEST CASES

## Work Order

``` text
valid transition
invalid transition
cancel
rework
complete
```

## Inventory

``` text
sufficient stock
insufficient stock
concurrent issue
return
adjustment
opname
```

## Payment

``` text
full payment
partial payment
overpayment
duplicate submit
invalid invoice
```

## Authorization

``` text
owner allowed
admin allowed
mechanic denied
warehouse denied
```

------------------------------------------------------------------------

# 52. OBSERVABILITY

Backend logs should include:

``` text
timestamp
level
request_id
user_id
route
method
status
duration
error_code
```

Do not log: - passwords - tokens - sensitive payment credentials

------------------------------------------------------------------------

# 53. BACKUP

MySQL backup:

``` text
daily full backup
```

Recommended retention:

``` text
7 daily
4 weekly
3 monthly
```

Verify backup by restoration testing.

------------------------------------------------------------------------

# 54. DEPLOYMENT ARCHITECTURE

Simple V1:

``` text
Internet
   ↓
HTTPS
   ↓
Nginx
   ├── frontend static files
   └── /api → Node.js
                 ↓
               PM2
                 ↓
               MySQL
```

------------------------------------------------------------------------

# 55. DOMAIN STRUCTURE

Example:

``` text
app.garagepro.id
api.garagepro.id
```

Alternative:

``` text
garagepro.id
api.garagepro.id
```

Frontend should never connect directly to MySQL.

------------------------------------------------------------------------

# 56. NGINX RESPONSIBILITIES

``` text
TLS termination
Static frontend
API reverse proxy
Compression
Security headers
Cache static assets
```

------------------------------------------------------------------------

# 57. PM2

Process:

``` text
garage-pro-api
```

Use: - restart on crash - startup persistence - logs - memory monitoring

------------------------------------------------------------------------

# 58. PWA ARCHITECTURE

Frontend:

``` text
Service Worker
Web App Manifest
Offline detection
Installable app
```

V1 offline behavior: - allow viewing cached shell where possible - show
connection status - do not falsely queue critical financial/inventory
mutations unless a robust offline sync system is implemented

------------------------------------------------------------------------

# 59. FILE UPLOADS

V1 can support workshop logo and selected attachments.

Architecture:

``` text
Frontend
 ↓
API
 ↓
Storage Adapter
```

Storage adapter should be abstracted so provider can change later.

Potential providers: - local storage - S3-compatible object storage -
Cloudinary

Do not couple business logic directly to a provider.

------------------------------------------------------------------------

# 60. CONFIGURATION LAYER

Central config:

``` text
config/
├── app.config.ts
├── db.config.ts
├── auth.config.ts
└── storage.config.ts
```

Environment parsing must happen once at startup.

------------------------------------------------------------------------

# 61. CODING CONVENTIONS

TypeScript:

``` text
strict: true
```

Avoid:

``` text
any
```

Prefer:

``` text
unknown
```

with explicit narrowing.

Use: - async/await - explicit return types for important services -
typed API responses - small focused functions

------------------------------------------------------------------------

# 62. NAMING

Files:

``` text
kebab-case.ts
```

React components:

``` text
PascalCase
```

Functions:

``` text
camelCase
```

Database:

``` text
snake_case
```

API:

``` text
kebab-case or plural REST resources
```

------------------------------------------------------------------------

# 63. CODE QUALITY

Every module should avoid: - giant components - giant services -
duplicated validation - duplicated API logic - hidden side effects -
business rules in JSX - direct DB queries in controllers

------------------------------------------------------------------------

# 64. CLAUDE CODE WORKFLOW

Claude Code should work incrementally.

Never request the entire application in one uncontrolled generation.

Preferred sequence:

``` text
Read docs
 ↓
Inspect repository
 ↓
Create architecture
 ↓
Implement foundation
 ↓
Test
 ↓
Implement one module
 ↓
Test
 ↓
Review
 ↓
Commit
 ↓
Next module
```

------------------------------------------------------------------------

# 65. CLAUDE CODE RULES

Claude must:

1.  Read all project docs before implementation.
2.  Never invent database fields without updating DB specification.
3.  Never invent API contracts.
4.  Never move business logic into frontend merely for convenience.
5.  Never bypass authorization.
6.  Never modify stock without inventory service.
7.  Never modify payment directly from frontend.
8.  Never allow arbitrary WO status updates.
9.  Run tests after each meaningful module.
10. Report changed files.
11. Report unresolved assumptions.
12. Avoid unnecessary dependencies.
13. Keep TypeScript strict.
14. Preserve existing working features.
15. Prefer small commits.

------------------------------------------------------------------------

# 66. CLAUDE CODE MASTER IMPLEMENTATION PROMPT

Use the following prompt when starting implementation:

``` text
You are the lead full-stack engineer for GARAGE PRO — Workshop Management System V1.

Read and obey these documents before writing production code:

1. GARAGE PRO — MASTER DEVELOPMENT SPECIFICATION V1
2. GARAGE PRO — DATABASE DESIGN V1
3. GARAGE PRO — API SPECIFICATION V1
4. GARAGE PRO — UI/UX SCREEN BIBLE V1
5. GARAGE PRO — DESIGN SYSTEM V1
6. GARAGE PRO — PROJECT ARCHITECTURE V1

TECH STACK

Frontend:
- React
- Vite
- TypeScript
- Tailwind CSS
- React Router
- TanStack Query
- React Hook Form
- Zod
- Axios
- Lucide React

Backend:
- Node.js
- Express
- TypeScript
- Sequelize
- MySQL 8+
- JWT/session authentication
- Zod
- Pino

RULES

1. Do not invent requirements.
2. Follow the database specification exactly.
3. Follow API contracts exactly.
4. Backend owns business rules.
5. Frontend is not a security boundary.
6. Enforce authorization on backend.
7. Use database transactions for critical multi-write operations.
8. Inventory must use immutable stock movements.
9. Payment must use server-side calculations.
10. Work Order status changes must use explicit transition logic.
11. Do not directly modify stock from controllers.
12. Do not directly modify database from controllers.
13. Use service and repository layers.
14. Use TypeScript strict mode.
15. Avoid any unless technically justified.
16. Use reusable UI components.
17. Follow Design System tokens.
18. Follow UI/UX Screen Bible.
19. Support mobile-first mechanic UX.
20. Support desktop admin/owner UX.
21. Every data page must handle loading, empty and error states.
22. Prevent duplicate submissions.
23. Validate on frontend and backend.
24. Add tests for critical business rules.
25. Do not add unnecessary dependencies.

IMPLEMENTATION ORDER

Phase 1:
- repository structure
- frontend foundation
- backend foundation
- database connection
- environment configuration
- logging
- error handling
- authentication

Phase 2:
- users
- roles
- permissions
- customers
- vehicles
- mechanics
- services
- parts
- warehouses
- suppliers

Phase 3:
- work orders
- inspection
- services
- parts
- recommendations
- workshop board
- QC

Phase 4:
- inventory
- receiving
- issue
- return
- adjustment
- stock opname

Phase 5:
- invoice
- payment
- completion

Phase 6:
- dashboard
- reports
- audit logs
- settings

After each phase:
- run tests
- run typecheck
- run lint
- build frontend
- verify API
- summarize changes
- list remaining issues

Do not continue to the next major phase if the current phase has blocking failures.

OUTPUT FORMAT FOR EACH IMPLEMENTATION STEP

1. Objective
2. Files to create
3. Files to modify
4. Implementation
5. Tests
6. Validation result
7. Remaining issues
8. Next recommended step
```

------------------------------------------------------------------------

# 67. INITIAL REPOSITORY COMMAND PLAN

Recommended:

``` bash
mkdir garage-pro
cd garage-pro

mkdir frontend backend docs scripts
```

Frontend:

``` bash
npm create vite@latest frontend -- --template react-ts
```

Backend:

``` bash
cd backend
npm init -y
```

Do not execute these blindly if an existing repository already exists.

------------------------------------------------------------------------

# 68. DEFINITION OF ARCHITECTURE DONE

Architecture is complete when:

``` text
[ ] Frontend structure defined
[ ] Backend structure defined
[ ] Feature boundaries defined
[ ] API client defined
[ ] State management defined
[ ] Authentication defined
[ ] Authorization defined
[ ] Service layer defined
[ ] Repository layer defined
[ ] Validation defined
[ ] Error handling defined
[ ] Audit defined
[ ] Inventory architecture defined
[ ] Payment architecture defined
[ ] Testing defined
[ ] Deployment defined
[ ] Environment defined
[ ] Git strategy defined
[ ] Claude Code workflow defined
```

------------------------------------------------------------------------

# 69. FINAL ARCHITECTURE

``` text
                     USER
                       │
             ┌─────────┴─────────┐
             │                   │
        Desktop Admin       Mobile Mechanic
             │                   │
             └─────────┬─────────┘
                       │
                 React Frontend
                       │
               TanStack Query
                       │
                  API Client
                       │
                 HTTPS / REST
                       │
                Express Backend
                       │
        ┌──────────────┼──────────────┐
        │              │              │
   Middleware      Controllers     Validators
        │              │              │
        └──────────────┼──────────────┘
                       │
                 Service Layer
                       │
        ┌──────────────┼──────────────┐
        │              │              │
    WorkOrder      Inventory      Payment
     Service        Service        Service
        │              │              │
        └──────────────┼──────────────┘
                       │
                  Repository
                       │
                   Sequelize
                       │
                     MySQL
                       │
               Audit / Ledger Data
```

------------------------------------------------------------------------

# 70. NEXT STEP

After this document, the project moves from **architecture design** to
implementation.

Recommended immediate next document:

``` text
GARAGE_PRO_BUSINESS_RULES_V1.md
```

It should lock the detailed business logic for: - Work Order -
inspection - approval - service - spare part - inventory - purchasing -
invoice - payment - QC - cancellation - rework - permissions -
numbering - stock ledger - transaction integrity

Then implementation should begin with:

``` text
01 PROJECT SCAFFOLD
02 DATABASE MIGRATION
03 SEED DATA
04 AUTHENTICATION
05 RBAC
06 MASTER DATA
07 WORK ORDER ENGINE
08 INVENTORY ENGINE
09 INVOICE/PAYMENT
10 FRONTEND
11 TESTING
12 DEPLOYMENT
```

------------------------------------------------------------------------

# END OF DOCUMENT

**GARAGE PRO --- PROJECT ARCHITECTURE V1**\
**Status: Development Ready**
