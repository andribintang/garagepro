# GARAGE PRO --- PROJECT SCAFFOLD V1

**Document Type:** Implementation Scaffold Specification\
**Product:** GARAGE PRO --- Workshop Management System\
**Version:** V1.0\
**Status:** Development Ready\
**Date:** 14 September 2026

------------------------------------------------------------------------

## 1. PURPOSE

Dokumen ini mendefinisikan struktur repository GARAGE PRO yang akan
menjadi fondasi implementasi source code.

Target:

``` text
GARAGE PRO
├── frontend
├── backend
├── docs
├── scripts
└── infrastructure
```

Tujuan: - reproducible development environment - clean separation
frontend/backend - strict TypeScript - predictable module structure -
siap digunakan Claude Code - siap untuk local development dan production
deployment

------------------------------------------------------------------------

# 2. REPOSITORY STRUCTURE

``` text
garage-pro/
├── frontend/
├── backend/
├── docs/
├── scripts/
├── infrastructure/
├── .github/
├── .gitignore
├── .editorconfig
├── README.md
└── docker-compose.yml
```

------------------------------------------------------------------------

# 3. FRONTEND STACK

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

Development tooling:

``` text
ESLint
Prettier
Vitest
React Testing Library
```

Optional:

``` text
Playwright
MSW
```

------------------------------------------------------------------------

# 4. BACKEND STACK

``` text
Node.js
Express
TypeScript
Sequelize
MySQL
Zod
Pino
JWT/session authentication
```

Development:

``` text
tsx
ESLint
Prettier
Vitest
Supertest
```

------------------------------------------------------------------------

# 5. INITIAL FRONTEND CREATION

Recommended:

``` bash
npm create vite@latest frontend -- --template react-ts
cd frontend
npm install
```

Install runtime dependencies:

``` bash
npm install react-router-dom
npm install @tanstack/react-query
npm install react-hook-form zod @hookform/resolvers
npm install axios
npm install lucide-react
```

Development dependencies:

``` bash
npm install -D tailwindcss @tailwindcss/vite
npm install -D eslint prettier
npm install -D vitest @testing-library/react @testing-library/jest-dom
```

Do not install packages that are not required by the implementation.

------------------------------------------------------------------------

# 6. INITIAL BACKEND CREATION

``` bash
mkdir backend
cd backend
npm init -y
```

Runtime:

``` bash
npm install express cors helmet
npm install sequelize mysql2
npm install zod
npm install pino pino-http
npm install jsonwebtoken
npm install bcrypt
```

Development:

``` bash
npm install -D typescript tsx
npm install -D @types/node @types/express
npm install -D @types/cors
npm install -D @types/jsonwebtoken
npm install -D @types/bcrypt
npm install -D eslint prettier
npm install -D vitest supertest
npm install -D @types/supertest
```

------------------------------------------------------------------------

# 7. TYPESCRIPT CONFIGURATION

Frontend:

``` json
{
  "compilerOptions": {
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true
  }
}
```

Backend:

``` json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "noImplicitAny": true,
    "noUncheckedIndexedAccess": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "outDir": "dist",
    "sourceMap": true
  }
}
```

Adjust module mode only if required by the selected Node
runtime/tooling.

------------------------------------------------------------------------

# 8. FRONTEND DIRECTORY

``` text
frontend/
├── public/
├── src/
│   ├── app/
│   ├── assets/
│   ├── components/
│   │   ├── ui/
│   │   ├── forms/
│   │   ├── tables/
│   │   ├── feedback/
│   │   ├── navigation/
│   │   └── charts/
│   ├── features/
│   │   ├── auth/
│   │   ├── dashboard/
│   │   ├── customers/
│   │   ├── vehicles/
│   │   ├── mechanics/
│   │   ├── services/
│   │   ├── parts/
│   │   ├── work-orders/
│   │   ├── inventory/
│   │   ├── purchasing/
│   │   ├── invoices/
│   │   ├── payments/
│   │   ├── reports/
│   │   └── settings/
│   ├── hooks/
│   ├── layouts/
│   ├── lib/
│   ├── pages/
│   ├── routes/
│   ├── stores/
│   ├── types/
│   ├── utils/
│   ├── main.tsx
│   └── index.css
├── .env.example
├── package.json
├── tsconfig.json
├── vite.config.ts
└── README.md
```

------------------------------------------------------------------------

# 9. FRONTEND FEATURE TEMPLATE

Example:

``` text
features/customers/
├── api/
│   └── customers.api.ts
├── components/
│   ├── CustomerCard.tsx
│   └── CustomerForm.tsx
├── hooks/
│   ├── useCustomers.ts
│   └── useCustomerMutations.ts
├── pages/
│   ├── CustomerListPage.tsx
│   ├── CustomerCreatePage.tsx
│   └── CustomerDetailPage.tsx
├── schemas/
│   └── customer.schema.ts
├── types/
│   └── customer.types.ts
└── index.ts
```

------------------------------------------------------------------------

# 10. BACKEND DIRECTORY

``` text
backend/
├── src/
│   ├── config/
│   ├── controllers/
│   ├── errors/
│   ├── logger/
│   ├── middlewares/
│   ├── models/
│   ├── migrations/
│   ├── seeders/
│   ├── policies/
│   ├── repositories/
│   ├── routes/
│   ├── services/
│   ├── types/
│   ├── utils/
│   ├── validators/
│   ├── app.ts
│   └── server.ts
├── tests/
├── .env.example
├── package.json
├── tsconfig.json
└── README.md
```

------------------------------------------------------------------------

# 11. BACKEND FEATURE TEMPLATE

Example:

``` text
services/
└── work-orders/
    ├── work-order.service.ts
    ├── inspection.service.ts
    └── recommendation.service.ts

repositories/
└── work-orders/
    └── work-order.repository.ts

controllers/
└── work-orders/
    └── work-order.controller.ts

validators/
└── work-orders/
    └── work-order.validator.ts
```

------------------------------------------------------------------------

# 12. FRONTEND APP BOOTSTRAP

``` text
main.tsx
   ↓
QueryClientProvider
   ↓
AuthProvider
   ↓
RouterProvider
   ↓
App
```

Provider order must be intentional.

------------------------------------------------------------------------

# 13. BACKEND APP BOOTSTRAP

``` text
server.ts
   ↓
load config
   ↓
connect database
   ↓
create Express app
   ↓
register middleware
   ↓
register routes
   ↓
error handler
   ↓
listen
```

------------------------------------------------------------------------

# 14. APP.TS RESPONSIBILITY

`app.ts`: - create Express app - middleware - routes - error handler

`server.ts`: - startup - database connection - server listen - graceful
shutdown

Keep startup logic out of controllers.

------------------------------------------------------------------------

# 15. CONFIGURATION

``` text
backend/src/config/
├── env.ts
├── app.config.ts
├── db.config.ts
├── auth.config.ts
└── storage.config.ts
```

`env.ts` validates required environment variables at startup.

If invalid:

``` text
Application fails fast.
```

------------------------------------------------------------------------

# 16. ENVIRONMENT FILES

Frontend:

``` text
.env.example
```

Example:

``` text
VITE_API_BASE_URL=http://localhost:3000/api/v1
```

Backend:

``` text
.env.example
```

Example:

``` text
NODE_ENV=development
PORT=3000

DB_HOST=localhost
DB_PORT=3306
DB_NAME=garage_pro
DB_USER=root
DB_PASSWORD=

JWT_SECRET=change-me
JWT_EXPIRES_IN=15m

FRONTEND_URL=http://localhost:5173
LOG_LEVEL=info
```

Never commit actual secrets.

------------------------------------------------------------------------

# 17. DATABASE CONNECTION

Sequelize configuration:

``` text
backend/src/config/db.config.ts
```

Requirements: - connection pool - timezone consistency - logging
disabled or controlled in production - retry only where appropriate

Health check:

``` text
GET /api/v1/health
```

Response:

``` json
{
  "success": true,
  "data": {
    "status": "ok"
  }
}
```

------------------------------------------------------------------------

# 18. MODEL STRUCTURE

Models grouped by domain:

``` text
models/
├── User.ts
├── Role.ts
├── Permission.ts
├── Customer.ts
├── Vehicle.ts
├── Mechanic.ts
├── Service.ts
├── ServiceCategory.ts
├── SparePart.ts
├── PartCategory.ts
├── Warehouse.ts
├── WarehouseLocation.ts
├── WorkOrder.ts
├── WorkOrderInspection.ts
├── WorkOrderService.ts
├── WorkOrderPart.ts
├── WorkOrderRecommendation.ts
├── StockMovement.ts
├── StockOpname.ts
├── StockOpnameItem.ts
├── Supplier.ts
├── Purchase.ts
├── PurchaseItem.ts
├── Invoice.ts
├── Payment.ts
├── Setting.ts
└── AuditLog.ts
```

Exact schema must follow the Database Design document.

------------------------------------------------------------------------

# 19. MIGRATION STRUCTURE

``` text
migrations/
001-create-roles
002-create-permissions
003-create-users
004-create-customers
005-create-vehicles
006-create-mechanics
007-create-service-categories
008-create-services
009-create-part-categories
010-create-spare-parts
011-create-warehouses
012-create-warehouse-locations
013-create-suppliers
014-create-work-orders
015-create-work-order-inspections
016-create-work-order-services
017-create-work-order-parts
018-create-work-order-recommendations
019-create-stock-movements
020-create-stock-opnames
021-create-stock-opname-items
022-create-purchases
023-create-purchase-items
024-create-invoices
025-create-payments
026-create-settings
027-create-audit-logs
```

Migration order may be adjusted to actual FK dependencies.

------------------------------------------------------------------------

# 20. SEEDER STRUCTURE

``` text
seeders/
├── roles.seed.ts
├── permissions.seed.ts
├── role-permissions.seed.ts
├── admin-user.seed.ts
├── service-categories.seed.ts
├── services.seed.ts
├── part-categories.seed.ts
├── warehouses.seed.ts
└── settings.seed.ts
```

Seed data must be deterministic where possible.

------------------------------------------------------------------------

# 21. INITIAL ROLE SEED

``` text
OWNER
ADMIN
MECHANIC
WAREHOUSE
```

------------------------------------------------------------------------

# 22. INITIAL PERMISSION SEED

Minimum permission groups:

``` text
dashboard.*
customers.*
vehicles.*
mechanics.*
services.*
parts.*
work_orders.*
inventory.*
purchases.*
invoices.*
payments.*
reports.*
users.*
roles.*
settings.*
audit_logs.*
```

Use explicit permission records, not wildcard authorization at runtime
unless intentionally implemented.

------------------------------------------------------------------------

# 23. INITIAL ADMIN

Development seed may create:

``` text
username: admin
```

Password must be supplied through environment/configuration or a secure
development-only setup.

Never hard-code a production password in source.

------------------------------------------------------------------------

# 24. FRONTEND DESIGN SYSTEM BOOTSTRAP

Implement first:

``` text
Button
Input
Textarea
Select
Combobox
Badge
Card
Table
Modal
Drawer
Toast
Alert
Skeleton
EmptyState
ErrorState
PageHeader
```

These are the base building blocks.

------------------------------------------------------------------------

# 25. APP LAYOUT

Implement:

``` text
AppLayout
AuthLayout
MechanicLayout
Sidebar
MobileHeader
UserMenu
```

AppLayout handles desktop/admin experience.

MechanicLayout simplifies mobile workflow.

------------------------------------------------------------------------

# 26. ROUTING FOUNDATION

Public:

``` text
/login
```

Protected:

``` text
/dashboard
/*
```

Permission-aware route:

``` text
/reports/*
/settings/*
```

Unauthorized route:

``` text
/403
```

Not found:

``` text
/404
```

------------------------------------------------------------------------

# 27. API ERROR CONTRACT

All backend errors should normalize to:

``` json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Human readable message",
    "details": {}
  }
}
```

Frontend error handler maps codes to UX messages.

------------------------------------------------------------------------

# 28. API SUCCESS CONTRACT

Example:

``` json
{
  "success": true,
  "data": {},
  "meta": {}
}
```

List:

``` json
{
  "success": true,
  "data": [],
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 100,
    "totalPages": 5
  }
}
```

------------------------------------------------------------------------

# 29. LOGGER

Use Pino.

Development: - readable logs

Production: - structured JSON

Include:

``` text
request_id
user_id
method
route
status
duration
```

Do not log: - passwords - access tokens - payment secrets

------------------------------------------------------------------------

# 30. REQUEST ID

Every request should receive a request ID.

Header:

``` text
X-Request-ID
```

If client supplies a safe ID, validate and propagate it.

Otherwise generate one.

------------------------------------------------------------------------

# 31. HEALTH CHECK

Endpoint:

``` text
GET /api/v1/health
```

Must verify: - application process - database connectivity

Do not expose sensitive configuration.

------------------------------------------------------------------------

# 32. FRONTEND API BASE

Local:

``` text
http://localhost:3000/api/v1
```

Production:

``` text
https://api.garagepro.id/api/v1
```

Actual domain may be configured by deployment environment.

------------------------------------------------------------------------

# 33. FRONTEND API LAYER

Example:

``` text
features/work-orders/api/work-orders.api.ts
```

Responsibilities: - endpoint calls - typed request - typed response

Do not put business logic in API functions.

------------------------------------------------------------------------

# 34. QUERY HOOK

Example:

``` text
useWorkOrders(filters)
```

Responsibilities: - query key - API call - caching - loading/error state

------------------------------------------------------------------------

# 35. MUTATION HOOK

Example:

``` text
useCreateWorkOrder()
useIssuePart()
useRecordPayment()
```

Responsibilities: - mutation - loading - error - invalidation - success
behavior

------------------------------------------------------------------------

# 36. FORM ARCHITECTURE

Each important form:

``` text
Schema
 ↓
React Hook Form
 ↓
UI fields
 ↓
Mutation
```

Example:

``` text
work-order.schema.ts
```

Backend repeats validation independently.

------------------------------------------------------------------------

# 37. FRONTEND TYPES

Do not duplicate incompatible business types.

Example:

``` text
WorkOrderStatus
```

must use one canonical frontend type.

API DTO types may differ from UI view models, but transformation must be
explicit.

------------------------------------------------------------------------

# 38. DATE UTILITIES

Create centralized date utilities:

``` text
formatDate()
formatDateTime()
formatTime()
toApiDate()
```

Avoid scattered date formatting logic.

------------------------------------------------------------------------

# 39. CURRENCY UTILITIES

Central:

``` text
formatCurrency()
```

Example:

``` text
formatCurrency(75000)
→ Rp 75.000
```

Never implement currency formatting independently in each component.

------------------------------------------------------------------------

# 40. PERMISSION HELPERS

Example:

``` text
can(user, "work_orders.approve")
```

Use for: - hiding buttons - disabling actions - route guards

Backend always performs actual authorization.

------------------------------------------------------------------------

# 41. WORK ORDER FEATURE

Initial components:

``` text
WorkOrderList
WorkOrderCard
WorkOrderSummary
WorkOrderStatus
WorkOrderTimeline
WorkOrderItems
WorkOrderActionBar
InspectionChecklist
RecommendationCard
```

------------------------------------------------------------------------

# 42. INVENTORY FEATURE

Components:

``` text
StockCard
StockBadge
StockMovementTable
LowStockTable
StockOpnameForm
StockAdjustmentForm
WarehouseSelector
```

------------------------------------------------------------------------

# 43. INVOICE FEATURE

Components:

``` text
InvoiceSummary
InvoiceItems
PaymentSummary
PaymentMethodSelector
PaymentForm
PrintInvoice
```

------------------------------------------------------------------------

# 44. DEVELOPMENT SCRIPTS

Frontend package scripts:

``` json
{
  "dev": "vite",
  "build": "tsc -b && vite build",
  "preview": "vite preview",
  "lint": "eslint .",
  "test": "vitest run",
  "test:watch": "vitest"
}
```

Backend:

``` json
{
  "dev": "tsx watch src/server.ts",
  "build": "tsc",
  "start": "node dist/server.js",
  "lint": "eslint .",
  "test": "vitest run",
  "test:watch": "vitest"
}
```

Actual scripts may be adapted to installed tooling versions.

------------------------------------------------------------------------

# 45. ROOT SCRIPTS

Optional root package management can use:

``` text
npm workspaces
```

or keep frontend/backend independent.

V1 recommendation:

> Keep frontend and backend independently installable to reduce
> coupling.

------------------------------------------------------------------------

# 46. GITIGNORE

Must include:

``` text
node_modules
dist
.env
.env.*
!.env.example
coverage
*.log
.DS_Store
.vscode/*
```

Do not ignore required shared project configuration accidentally.

------------------------------------------------------------------------

# 47. EDITORCONFIG

Recommended:

``` text
root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
indent_style = space
indent_size = 2
```

------------------------------------------------------------------------

# 48. README

Root README must contain:

``` text
Project Overview
Features
Architecture
Requirements
Local Setup
Environment
Database Setup
Run Frontend
Run Backend
Testing
Deployment
Documentation
```

------------------------------------------------------------------------

# 49. LOCAL DEVELOPMENT FLOW

``` text
1. Clone repository
2. Install frontend dependencies
3. Install backend dependencies
4. Create .env files
5. Start MySQL
6. Run migrations
7. Run seeders
8. Start backend
9. Start frontend
10. Login with development account
```

------------------------------------------------------------------------

# 50. DATABASE DEVELOPMENT

Recommended local:

``` text
MySQL 8+
```

Database:

``` text
garage_pro
```

Test database:

``` text
garage_pro_test
```

Never run destructive test scripts against development production-like
data.

------------------------------------------------------------------------

# 51. MIGRATION COMMANDS

Project should expose:

``` bash
npm run db:migrate
npm run db:migrate:undo
npm run db:seed
npm run db:seed:undo
```

Exact Sequelize CLI integration may be used.

------------------------------------------------------------------------

# 52. TEST DATABASE

Automated tests should use isolated database/schema.

Test suite must be able to reset state safely.

------------------------------------------------------------------------

# 53. DOCKER DEVELOPMENT OPTION

Optional:

``` text
docker-compose.yml
```

Services:

``` text
mysql
backend
frontend
```

For V1, Docker is a development convenience, not a mandatory production
architecture.

------------------------------------------------------------------------

# 54. DOCKER MYSQL

Conceptual:

``` text
mysql:
  image: mysql:8
  environment:
    MYSQL_DATABASE: garage_pro
    MYSQL_USER: garage
    MYSQL_PASSWORD: ...
    MYSQL_ROOT_PASSWORD: ...
```

Never commit actual passwords.

------------------------------------------------------------------------

# 55. CLAUDE CODE IMPLEMENTATION ORDER

Claude Code should execute:

``` text
STEP 1
Create repository structure

STEP 2
Initialize frontend

STEP 3
Initialize backend

STEP 4
Configure TypeScript

STEP 5
Configure lint/format/test

STEP 6
Configure environment

STEP 7
Create database connection

STEP 8
Create health endpoint

STEP 9
Create frontend app shell

STEP 10
Create base design-system components

STEP 11
Verify build
```

Only after scaffold passes:

``` text
DATABASE MIGRATION
```

------------------------------------------------------------------------

# 56. SCAFFOLD ACCEPTANCE CRITERIA

``` text
[ ] npm install succeeds
[ ] frontend starts
[ ] backend starts
[ ] TypeScript passes
[ ] ESLint passes
[ ] frontend build passes
[ ] backend build passes
[ ] MySQL connects
[ ] health endpoint works
[ ] frontend can call health endpoint
[ ] environment validation works
[ ] README works
[ ] no secrets committed
```

------------------------------------------------------------------------

# 57. CLAUDE CODE SCAFFOLD PROMPT

``` text
You are implementing the initial project scaffold for GARAGE PRO V1.

Read:
- MASTER DEVELOPMENT SPECIFICATION
- DATABASE DESIGN
- API SPECIFICATION
- UI/UX SCREEN BIBLE
- DESIGN SYSTEM
- PROJECT ARCHITECTURE
- BUSINESS RULES
- PROJECT SCAFFOLD

Your current task is ONLY project scaffolding.

Do not implement business modules yet.

Create:
- frontend/
- backend/
- docs/
- scripts/
- infrastructure/
- root configuration

Frontend:
- React
- Vite
- TypeScript strict
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
- TypeScript strict
- Sequelize
- MySQL
- Zod
- Pino
- authentication dependencies

Implement:
1. Environment configuration
2. Database connection
3. Health endpoint
4. Express bootstrap
5. React bootstrap
6. Router foundation
7. App layout placeholder
8. Design system placeholder
9. API client
10. Query client
11. ESLint
12. Prettier
13. Vitest
14. README
15. .gitignore
16. .env.example

Rules:
- Do not invent domain fields.
- Do not implement stock logic.
- Do not implement payment logic.
- Do not implement Work Order business logic.
- Do not add unnecessary packages.
- Keep frontend/backend independent.
- Use strict TypeScript.
- No production secrets.
- Run lint, test and build.
- Fix all blocking errors before finishing.

Final response must contain:
1. Created files
2. Modified files
3. Commands executed
4. Build result
5. Test result
6. Remaining issues
7. Next step: database migration
```

------------------------------------------------------------------------

# 58. NEXT DOCUMENT

After scaffold, implement:

``` text
GARAGE_PRO_DATABASE_MIGRATION_V1.md
```

It should define: - exact migration order - exact SQL/Sequelize
migration - indexes - foreign keys - constraints - seed data - initial
roles - initial permissions - default settings - test database setup -
rollback strategy

------------------------------------------------------------------------

# END OF DOCUMENT

**GARAGE PRO --- PROJECT SCAFFOLD V1**\
**Status: Development Ready**
