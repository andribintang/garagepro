# GARAGE PRO

> **Workshop Management System V1**  
> Sistem manajemen bengkel motor umum berbasis web untuk mengelola customer, kendaraan, inspeksi, Work Order, mekanik, spare part, inventory, invoice, pembayaran, laporan, dan service history.

---

## 🚀 Project Overview

**GARAGE PRO** adalah aplikasi manajemen operasional bengkel motor yang dirancang untuk mengubah proses bengkel dari pencatatan manual menjadi workflow digital yang terstruktur, terukur, dan mudah digunakan.

### Core Flow

```text
Customer
   ↓
Vehicle
   ↓
Inspection / Diagnosis
   ↓
Work Order
   ↓
Service + Spare Part
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
   ↓
Payment
   ↓
Completed
   ↓
Service History
```

---

## 🎯 Project Goals

GARAGE PRO dibangun dengan tujuan:

- Mempercepat proses penerimaan kendaraan.
- Mengurangi kesalahan pencatatan service dan spare part.
- Mengontrol penggunaan dan stok spare part secara real-time.
- Memisahkan proses estimate, approval, execution, invoicing, dan payment.
- Menyediakan service history kendaraan.
- Menyediakan kontrol akses berdasarkan role.
- Menyediakan audit trail untuk aktivitas penting.
- Menjadi fondasi ERP/Workshop Management yang dapat dikembangkan ke multi-branch.

---

## 🧩 Main Modules

### 1. Authentication & RBAC
- Login / logout
- Session management
- Role & permission
- Protected routes
- Audit login/activity

### 2. Customer Management
- Customer CRUD
- Customer search
- Customer history
- Contact information

### 3. Vehicle Management
- Data kendaraan
- Nomor polisi
- Brand / model
- Tahun
- Kilometer
- Relasi customer
- Service history

### 4. Master Data
- Service category
- Service
- Part category
- Spare part
- Mechanic
- Supplier
- Warehouse
- Warehouse location
- Inspection item

### 5. Work Order
Work Order merupakan core engine GARAGE PRO.

Status utama:

```text
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

Status tambahan:

```text
REJECTED
CANCELLED
```

### 6. Inspection & Diagnosis
- Checklist inspeksi
- Diagnosis
- Catatan mekanik
- Temuan kerusakan
- Rekomendasi pekerjaan
- Additional work

### 7. Inventory
- Multi warehouse
- Stock balance
- Stock movement ledger
- Stock issue
- Stock return
- Receiving
- Adjustment
- Stock opname
- Stock transfer
- Low stock alert
- Inventory reconciliation

**Prinsip utama:**

> Inventory ledger adalah source of truth.

Stock tidak berkurang ketika estimate dibuat. Stock berkurang ketika spare part benar-benar di-issue/consume.

### 8. Purchasing
- Supplier
- Purchase
- Purchase item
- Receiving
- Stock update
- Purchase history

### 9. Invoice & Payment
- Invoice generation
- Discount
- Tax
- Grand total
- Partial payment
- Multiple payment methods
- Payment history
- Receipt
- Void / refund control

Payment methods:

```text
CASH
TRANSFER
QRIS
DEBIT_CARD
CREDIT_CARD
OTHER
```

### 10. Reports & Dashboard
- Revenue
- Collection
- Outstanding
- Work Orders
- Services
- Spare Parts
- Mechanic performance
- Stock
- Low stock
- Purchasing
- Operational dashboard

---

## 👥 User Roles

| Role | Main Responsibility |
|---|---|
| OWNER | Dashboard, reports, configuration, full control |
| ADMIN | Customer, vehicle, WO, invoice, payment |
| MECHANIC | Inspection, diagnosis, service execution, QC |
| WAREHOUSE | Inventory, receiving, issue, return, stock opname |

Permission-based authorization digunakan di seluruh backend dan frontend.

---

## 🏗️ Technology Stack

### Frontend

- React
- Vite
- TypeScript
- Tailwind CSS
- React Router
- TanStack Query
- React Hook Form
- Zod
- Axios
- Lucide Icons
- PWA

### Backend

- Node.js
- Express
- TypeScript
- Sequelize
- MySQL 8+
- JWT / session authentication
- bcrypt / Argon2
- Pino logging

### Infrastructure

```text
Internet
   ↓
Cloudflare
   ↓
Nginx HTTPS
   ↓
React Frontend
   ↓
Node.js / Express API
   ↓
MySQL
```

Production process manager:

- PM2

Optional:

- Docker
- CI/CD
- Cloud object storage

---

## 📁 Repository Structure

Target structure:

```text
garage-pro/
│
├── frontend/
│   ├── src/
│   │   ├── app/
│   │   ├── components/
│   │   ├── features/
│   │   ├── layouts/
│   │   ├── pages/
│   │   ├── hooks/
│   │   ├── lib/
│   │   ├── services/
│   │   └── types/
│   ├── public/
│   └── package.json
│
├── backend/
│   ├── src/
│   │   ├── config/
│   │   ├── controllers/
│   │   ├── middlewares/
│   │   ├── models/
│   │   ├── repositories/
│   │   ├── routes/
│   │   ├── services/
│   │   ├── validators/
│   │   ├── utils/
│   │   └── app.ts
│   ├── migrations/
│   ├── seeders/
│   └── package.json
│
├── docs/
│   ├── GARAGE_PRO_MASTER_DEVELOPMENT_SPECIFICATION_V1.md
│   ├── GARAGE_PRO_DATABASE_DESIGN_V1.md
│   ├── GARAGE_PRO_API_SPECIFICATION_V1.md
│   ├── GARAGE_PRO_UI_UX_SCREEN_BIBLE_V1.md
│   ├── GARAGE_PRO_DESIGN_SYSTEM_V1.md
│   ├── GARAGE_PRO_PROJECT_ARCHITECTURE_V1.md
│   ├── GARAGE_PRO_BUSINESS_RULES_V1.md
│   ├── GARAGE_PRO_PROJECT_SCAFFOLD_V1.md
│   ├── GARAGE_PRO_DATABASE_MIGRATION_V1.md
│   ├── GARAGE_PRO_AUTH_RBAC_V1.md
│   ├── GARAGE_PRO_MASTER_DATA_V1.md
│   ├── GARAGE_PRO_WORK_ORDER_ENGINE_V1.md
│   ├── GARAGE_PRO_INVENTORY_ENGINE_V1.md
│   ├── GARAGE_PRO_INVOICE_PAYMENT_V1.md
│   ├── GARAGE_PRO_FRONTEND_INTEGRATION_V1.md
│   ├── GARAGE_PRO_TESTING_QA_V1.md
│   └── GARAGE_PRO_DEPLOYMENT_PRODUCTION_OPERATIONS_V1.md
│
├── infrastructure/
├── scripts/
├── .github/
├── .gitignore
├── .editorconfig
├── docker-compose.yml
└── README.md
```

---

## 📚 Documentation

The `docs/` directory is the authoritative source for product, technical, business, and implementation requirements.

### Architecture & Product

1. Master Development Specification
2. Database Design & ERD
3. API Specification
4. UI/UX Screen Bible
5. Design System
6. Project Architecture

### Business & Core Engine

7. Business Rules
8. Project Scaffold
9. Database Migration
10. Authentication & RBAC
11. Master Data
12. Work Order Engine
13. Inventory Engine
14. Invoice & Payment Engine

### Delivery

15. Frontend Integration
16. Testing & QA
17. Deployment & Production Operations

> **Development rule:** source code must follow the requirements in `docs/`. If implementation conflicts with the documentation, update the documentation or explicitly document the decision before changing the core business behavior.

---

## 🔐 Critical Business Rules

### Work Order

- WO number generated by server.
- WO status can only change through the state transition service.
- Invalid status transitions must be rejected.
- Paid/completed WO cannot be freely edited.
- Cancellation must be audited.
- Additional work requires customer approval when applicable.

### Inventory

- Stock movements are immutable.
- Inventory uses ledger-based tracking.
- Estimate does not reduce stock.
- Issue/consume reduces stock.
- Return increases stock.
- Negative stock is not allowed unless explicitly configured.
- Stock operations must use database transactions and row locking.
- Inventory reconciliation must be possible from the ledger.

### Financial

- Invoice totals are calculated server-side.
- Historical transaction prices are frozen.
- Overpayment is rejected.
- Payment operations are transactional.
- Invoice/payment mutations require audit records.
- Paid invoices cannot be silently modified.
- Revenue, collection, and outstanding are separate metrics.

### Security

- Authentication is mandatory for protected endpoints.
- Authorization is permission-based.
- IDOR must be prevented.
- Sensitive mutations must be audited.
- Passwords must never be stored in plain text.
- Secrets must come from environment variables.
- MySQL must not be publicly exposed in production.

---

## 🔄 Development Roadmap

Current architecture/documentation baseline:

```text
01  Master Development Specification     ✅
02  Database Design + ERD                ✅
03  API Specification                    ✅
04  UI/UX Screen Bible                   ✅
05  Design System                        ✅
06  Project Architecture                 ✅
07  Business Rules                       ✅
08  Project Scaffold                     ✅
09  Database Migration + Seed             ✅
10  Authentication + RBAC                 ✅
11  Master Data                           ✅
12  Work Order Engine                     ✅
13  Inventory Engine                      ✅
14  Invoice + Payment Engine               ✅
15  Frontend Integration                  ✅
16  Testing + QA                          ✅
17  Deployment + Production Operations    ✅
```

### Next Phase — Actual Source Code

```text
18  Repository Bootstrap
19  Database + Migration Implementation
20  Authentication + RBAC Implementation
21  Master Data Implementation
22  Work Order Implementation
23  Inventory Implementation
24  Invoice + Payment Implementation
25  Frontend Integration
26  Automated Testing
27  Staging Deployment
28  UAT
29  Production Release
```

---

## 🧪 Quality Standards

Before production release:

- P0 defects = 0
- P1 defects = 0
- Critical business rules tested
- RBAC tested
- Inventory concurrency tested
- Payment concurrency tested
- Idempotency tested
- Audit trail tested
- Database migration tested
- Backup/restore tested
- API integration tested
- Mobile responsive tested
- Desktop responsive tested
- Accessibility checked
- Production smoke test passed

### Golden Path

The complete end-to-end test must support:

```text
Create Customer
→ Create Vehicle
→ Create WO
→ Inspection
→ Add Service
→ Add Spare Part
→ Estimate
→ Approval
→ Issue Part
→ Start Work
→ QC
→ Ready
→ Invoice
→ Payment
→ Complete
→ Verify Service History
→ Verify Inventory Ledger
```

---

## 🧑‍💻 Development with AI / Claude Code

GARAGE PRO is designed to be developed incrementally using AI coding agents such as Claude Code.

### AI Development Rules

AI coding agent MUST:

1. Read relevant documents in `docs/` before implementing a module.
2. Follow the existing architecture.
3. Never invent business rules when requirements already exist.
4. Never bypass service-layer business logic.
5. Never directly mutate inventory balances without ledger movements.
6. Never calculate final invoice/payment totals only on the frontend.
7. Never bypass authorization.
8. Add validation for every mutation endpoint.
9. Add audit logging for important mutations.
10. Add automated tests for critical business logic.
11. Keep changes modular and reviewable.
12. Avoid unnecessary dependencies.
13. Preserve backward compatibility unless a breaking change is explicitly approved.

### Recommended Implementation Order

```text
Database
  ↓
Models
  ↓
Repositories
  ↓
Domain Services
  ↓
Validators
  ↓
Controllers
  ↓
Routes
  ↓
API Tests
  ↓
Frontend API Client
  ↓
React Query
  ↓
Pages
  ↓
Forms / Tables / Components
  ↓
E2E Tests
```

---

## 🛠️ Local Development

Once the application scaffold is implemented:

### Backend

```bash
cd backend
npm install
npm run dev
```

### Frontend

```bash
cd frontend
npm install
npm run dev
```

### Database

Recommended:

```text
MySQL 8+
Database: garage_pro
```

Environment variables must be configured using `.env` files and must never be committed.

Example:

```env
NODE_ENV=development
PORT=4000

DB_HOST=localhost
DB_PORT=3306
DB_NAME=garage_pro
DB_USER=garage_pro
DB_PASSWORD=change-me

JWT_SECRET=change-me
```

---

## 🌐 API Convention

Base API:

```text
/api/v1
```

Example:

```text
POST   /api/v1/auth/login
GET    /api/v1/auth/me

GET    /api/v1/customers
POST   /api/v1/customers

GET    /api/v1/vehicles
POST   /api/v1/vehicles

GET    /api/v1/work-orders
POST   /api/v1/work-orders

POST   /api/v1/work-orders/:id/inspection
POST   /api/v1/work-orders/:id/approve
POST   /api/v1/work-orders/:id/start
POST   /api/v1/work-orders/:id/qc
POST   /api/v1/work-orders/:id/ready
POST   /api/v1/work-orders/:id/invoice
POST   /api/v1/work-orders/:id/complete

GET    /api/v1/invoices
POST   /api/v1/invoices/:id/payments

GET    /api/v1/dashboard/summary
GET    /api/v1/reports/revenue
```

API response dan error format harus mengikuti `GARAGE_PRO_API_SPECIFICATION_V1.md`.

---

## 🌳 Git Workflow

Recommended branch:

```text
main
develop
feature/*
fix/*
hotfix/*
```

Example:

```bash
git checkout -b feature/work-order-engine
```

Commit convention:

```text
feat: add work order creation
feat: implement inventory issue
fix: prevent invoice overpayment
test: add payment concurrency tests
refactor: improve inventory service
docs: update API specification
chore: update dependencies
```

---

## 🚢 Production

Recommended production architecture:

```text
                    ┌───────────────┐
                    │   Internet    │
                    └───────┬───────┘
                            ↓
                    ┌───────────────┐
                    │   Cloudflare  │
                    └───────┬───────┘
                            ↓
                    ┌───────────────┐
                    │     Nginx     │
                    │ HTTPS / Proxy │
                    └───────┬───────┘
                            ↓
             ┌──────────────┴──────────────┐
             ↓                             ↓
     ┌───────────────┐             ┌───────────────┐
     │ React / PWA   │             │ Node / Express│
     │ Static Assets │             │      API      │
     └───────────────┘             └───────┬───────┘
                                           ↓
                                    ┌───────────────┐
                                    │    MySQL 8    │
                                    └───────┬───────┘
                                            ↓
                                    ┌───────────────┐
                                    │ Backup/Offsite│
                                    └───────────────┘
```

Production deployment should use:

- Ubuntu LTS
- Nginx
- Node.js LTS
- MySQL 8+
- PM2
- HTTPS
- Cloudflare
- Firewall
- Automated backup
- Offsite backup
- Monitoring
- Release/rollback strategy

See:

`docs/GARAGE_PRO_DEPLOYMENT_PRODUCTION_OPERATIONS_V1.md`

---

## 📌 Definition of Done

A feature is considered complete only when:

```text
Requirement
    ↓
Database
    ↓
Migration
    ↓
Model
    ↓
Repository
    ↓
Service / Business Logic
    ↓
Validation
    ↓
API
    ↓
Authorization
    ↓
Audit
    ↓
Frontend
    ↓
Error Handling
    ↓
Automated Tests
    ↓
UI/UX Verification
    ↓
Documentation
    ↓
Done
```

A feature that only "works on the screen" is **not considered complete**.

---

## 📈 Future Roadmap

Potential V2/V3 capabilities:

- Multi-branch workshop
- Multi-warehouse advanced
- WhatsApp integration
- Customer notification
- Digital approval via WhatsApp
- Online booking
- QRIS payment integration
- Loyalty program
- Membership
- Reminder service
- Customer mobile app
- Mechanic productivity analytics
- Advanced purchasing
- Accounting integration
- Parts supplier integration
- Marketplace / e-commerce integration
- Advanced BI dashboard
- AI-assisted diagnosis
- AI service recommendation

These features should not be implemented in V1 unless explicitly approved.

---

## 📄 Project Status

**Project:** GARAGE PRO  
**Product:** Workshop Management System  
**Version:** V1  
**Status:** Architecture & Development Specification Complete → Source Code Implementation  
**Primary Language:** TypeScript  
**Database:** MySQL 8+  
**Frontend:** React + Vite  
**Backend:** Node.js + Express  

---

## 👤 Project

**GARAGE PRO**

Repository:

`https://github.com/andribintang/garagepro`

---

## ⚠️ Important

GARAGE PRO handles operational, inventory, and financial data.

Therefore:

> **Correctness and traceability are more important than implementation speed.**

Any change involving:

- Work Order status
- Inventory
- Invoice
- Payment
- Price
- Customer approval
- User permission
- Audit trail

must be treated as a **critical business change** and must include appropriate validation, transaction handling, testing, and auditability.

---

**GARAGE PRO — Workshop Management System V1**
