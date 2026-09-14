# GARAGE PRO

**Workshop Management System V1**

GARAGE PRO adalah aplikasi manajemen bengkel motor berbasis web/PWA yang dirancang untuk mengelola operasional bengkel secara terintegrasi, mulai dari data pelanggan dan kendaraan, inspeksi, Work Order, service, spare part, inventory, purchasing, invoice, pembayaran, hingga service history dan reporting.

> **Status:** Development / V1  
> **Repository:** `andribintang/garagepro`  
> **Default Branch:** `main`

---

## 1. Vision

GARAGE PRO dibangun sebagai **single source of truth** untuk operasional bengkel.

Tujuan utama:

- Mempercepat proses penerimaan kendaraan.
- Mengurangi kesalahan pencatatan Work Order.
- Mengontrol penggunaan dan pergerakan spare part.
- Memisahkan estimate, approval, pekerjaan aktual, invoice, dan pembayaran.
- Menjaga histori service kendaraan secara lengkap.
- Menyediakan dashboard dan laporan operasional yang dapat dipercaya.
- Mendukung penggunaan desktop untuk admin/owner dan mobile untuk mekanik.
- Menjadi fondasi ERP bengkel yang dapat dikembangkan ke V2.

---

## 2. Core Business Flow

```text
CUSTOMER
   ↓
VEHICLE
   ↓
INSPECTION
   ↓
WORK ORDER
   ↓
SERVICE + SPARE PART
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
   ↓
SERVICE HISTORY
```

### Work Order State Machine

```text
NEW
 ↓
CHECKING
 ↓
ESTIMATE
 ↓
WAITING_APPROVAL
 ├── REJECTED
 └── APPROVED
       ↓
   IN_PROGRESS
       ↓
       QC
      ├── REWORK → IN_PROGRESS
      └── READY
           ↓
        INVOICED
           ↓
          PAID
           ↓
       COMPLETED
```

---

## 3. Main Modules

### Master Data

- Customer
- Vehicle
- Mechanic
- Service Category
- Service
- Part Category
- Spare Part
- Supplier
- Warehouse
- Warehouse Location
- Inspection Item

### Workshop

- Work Order
- Inspection
- Diagnosis
- Service
- Spare Part
- Recommendation
- Additional Work
- Customer Approval
- Mechanic Assignment
- QC
- Rework
- Service History

### Inventory

- Multi Warehouse
- Stock Balance
- Stock Movement
- Issue / Consumption
- Return
- Receiving
- Stock Adjustment
- Stock Opname
- Stock Transfer
- Minimum Stock Alert
- Inventory Reconciliation

### Purchasing

- Supplier
- Purchase
- Purchase Items
- Receiving
- Stock Update

### Finance

- Estimate
- Invoice
- Payment
- Partial Payment
- Outstanding
- Receipt
- Void / Refund
- Revenue Report
- Collection Report

### Reporting

- Dashboard
- Revenue
- Work Order
- Service
- Spare Part
- Mechanic Performance
- Stock
- Outstanding Payment

---

## 4. User Roles

| Role | Primary Responsibility |
|---|---|
| OWNER | Monitoring, reporting, configuration, full business control |
| ADMIN | Customer, vehicle, WO, invoice, payment, operational administration |
| MECHANIC | Inspection, diagnosis, service execution, spare part usage, QC |
| WAREHOUSE | Inventory, receiving, issue, return, stock opname, transfer |

Semua akses harus melalui **RBAC (Role-Based Access Control)**.

---

## 5. Technology Stack

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
- JWT / Session Authentication
- bcrypt / Argon2
- Pino Logger

### Infrastructure

- Linux / Ubuntu LTS
- Nginx
- PM2
- MySQL 8+
- Cloudflare
- HTTPS
- Docker (optional)
- GitHub

---

## 6. High-Level Architecture

```text
┌──────────────────────────────┐
│           USER               │
│ Desktop / Tablet / Mobile    │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│       React + Vite           │
│       Tailwind + PWA         │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│       TanStack Query         │
│         API Client           │
└──────────────┬───────────────┘
               │ HTTPS
               ▼
┌──────────────────────────────┐
│       Express API            │
│ Auth / RBAC / Validation     │
│ Controllers / Services       │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│       Domain Services        │
│ WO / Inventory / Finance     │
│ Purchasing / Audit           │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│ Sequelize / Repository       │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│          MySQL 8+            │
└──────────────────────────────┘
```

---

## 7. Repository Structure

Target repository structure:

```text
garagepro/
│
├── frontend/
│   ├── src/
│   │   ├── app/
│   │   ├── components/
│   │   ├── features/
│   │   ├── layouts/
│   │   ├── lib/
│   │   ├── routes/
│   │   ├── hooks/
│   │   ├── types/
│   │   └── styles/
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
│   │   ├── services/
│   │   ├── validators/
│   │   ├── routes/
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
├── scripts/
├── infrastructure/
├── .github/
├── .gitignore
├── .editorconfig
├── docker-compose.yml
└── README.md
```

---

## 8. Database Principles

Database menggunakan **MySQL 8+ / InnoDB**.

Prinsip utama:

1. Transaction data tidak boleh di-hard-delete.
2. Master data menggunakan soft delete jika diperlukan.
3. Stock movement bersifat immutable.
4. Inventory ledger menjadi source of truth.
5. Historical price harus disimpan sebagai snapshot.
6. Invoice dan payment menggunakan perhitungan server-side.
7. Paid Work Order tidak boleh diedit secara bebas.
8. Perubahan penting harus masuk audit log.
9. Transaksi inventory menggunakan database transaction dan row locking.
10. Payment harus menggunakan concurrency protection.
11. Nomor dokumen dibuat oleh server.
12. Semua nominal uang menggunakan `DECIMAL`, bukan floating point.

---

## 9. Critical Business Rules

### Inventory

Estimate **tidak mengurangi stock**.

Stock berkurang ketika spare part benar-benar:

```text
ISSUED / CONSUMED
```

Stock bertambah melalui:

```text
PURCHASE RECEIVING
RETURN
ADJUSTMENT IN
```

Stock berkurang melalui:

```text
ISSUE
ADJUSTMENT OUT
OPNAME ADJUSTMENT
```

### Financial

Total invoice harus dihitung server:

```text
Subtotal
- Line Discount
- Document Discount
+ Tax
= Grand Total
```

Payment:

```text
Grand Total
- Total Paid
= Outstanding
```

Overpayment harus ditolak.

### Work Order

Perubahan status hanya boleh melalui **state transition service**, bukan dengan update database langsung dari controller.

---

## 10. API Convention

Base URL:

```text
/api/v1
```

Contoh:

```http
POST   /api/v1/auth/login
GET    /api/v1/auth/me

GET    /api/v1/customers
POST   /api/v1/customers
GET    /api/v1/customers/:id
PATCH  /api/v1/customers/:id

GET    /api/v1/work-orders
POST   /api/v1/work-orders

POST   /api/v1/work-orders/:id/inspection
POST   /api/v1/work-orders/:id/services
POST   /api/v1/work-orders/:id/parts
POST   /api/v1/work-orders/:id/approve
POST   /api/v1/work-orders/:id/start
POST   /api/v1/work-orders/:id/qc
POST   /api/v1/work-orders/:id/ready
POST   /api/v1/work-orders/:id/invoice
POST   /api/v1/work-orders/:id/complete

GET    /api/v1/invoices
POST   /api/v1/invoices/:id/payments

GET    /api/v1/reports/revenue
GET    /api/v1/reports/work-orders
GET    /api/v1/reports/stock
```

API response harus konsisten dan memiliki error code yang dapat digunakan frontend.

---

## 11. UI/UX Principles

GARAGE PRO menggunakan pendekatan **Premium SaaS UI**.

Inspirasi karakter:

- Clean
- Modern
- Professional
- Fast
- Data-oriented
- Mobile-first untuk mekanik
- Desktop-first untuk admin/owner
- Responsive
- Accessible
- Dark mode ready

### Responsive Strategy

```text
Mobile
  ↓
Mechanic / Warehouse workflow

Tablet
  ↓
Workshop operational workflow

Desktop
  ↓
Admin / Owner / Reporting
```

Touch target minimum:

```text
44 × 44 px
```

Status, warning, success, error dan confirmation harus menggunakan visual feedback yang konsisten.

---

## 12. Mobile Mechanic Experience

Mekanik harus dapat melakukan pekerjaan utama tanpa menggunakan desktop.

Prioritas mobile:

```text
My Work Orders
      ↓
Open Work Order
      ↓
Inspection
      ↓
Diagnosis
      ↓
Service
      ↓
Parts Used
      ↓
Additional Recommendation
      ↓
QC
      ↓
Complete
```

Interface harus meminimalkan:

- typing
- navigasi berulang
- modal bertingkat
- input yang tidak diperlukan

Gunakan:

- large action button
- quick selection
- searchable combobox
- bottom action bar
- status chips
- confirmation dialog untuk aksi kritis

---

## 13. Security

Security baseline:

- HTTPS
- Secure authentication
- Password hashing
- JWT/session protection
- RBAC
- Input validation
- IDOR protection
- SQL injection protection
- XSS protection
- CSRF strategy jika diperlukan
- Rate limiting
- Secure headers
- CORS whitelist
- Request ID
- Audit log
- Secret melalui environment variable
- Database tidak diekspos ke public internet

---

## 14. Testing & QA

GARAGE PRO menggunakan test pyramid:

```text
              E2E
             /   \
        Integration
          /       \
       API / Component
          /       \
           Unit
```

Critical areas:

- Authentication
- RBAC
- Work Order state machine
- Inventory
- Invoice
- Payment
- Concurrency
- Idempotency
- Audit log
- Service history

### Release Gate

Production release tidak boleh dilakukan jika:

```text
P0 > 0     ❌
P1 > 0     ❌
Critical test failed ❌
Migration failed ❌
Backup verification failed ❌
Smoke test failed ❌
```

Target:

```text
P0 = 0
P1 = 0
Critical flows PASS
```

---

## 15. Production Architecture

```text
Internet
   ↓
Cloudflare
   ↓
Nginx / HTTPS
   ├───────────────┐
   ↓               ↓
Frontend        Backend API
React/PWA       Node/Express
                    ↓
                  MySQL
                    ↓
             Backup / Offsite
```

Initial server recommendation:

```text
4 vCPU
8 GB RAM
100+ GB SSD
Ubuntu LTS
```

Untuk scale yang lebih besar:

```text
8+ vCPU
16+ GB RAM
200+ GB SSD
```

---

## 16. Development Roadmap

Dokumentasi desain V1 telah dibagi menjadi beberapa fase:

```text
01  Master Development Specification     ✅
02  Database Design + ERD                ✅
03  API Specification                    ✅
04  UI/UX Screen Bible                   ✅
05  Design System                        ✅
06  Project Architecture                 ✅
07  Business Rules                       ✅
08  Project Scaffold                     ✅
09  Database Migration + Seed            ✅
10  Authentication + RBAC                ✅
11  Master Data                           ✅
12  Work Order Engine                     ✅
13  Inventory Engine                      ✅
14  Invoice + Payment Engine              ✅
15  Frontend Integration                  ✅
16  Testing + QA                          ✅
17  Deployment + Production Operations    ✅

18  Actual Source Code Implementation     🚧
19  Staging Deployment                    ⏳
20  UAT                                   ⏳
21  Production Release                    ⏳
```

**Next major objective: Phase 18 — Actual Source Code Implementation.**

---

## 17. Documentation

Seluruh keputusan arsitektur dan business rules harus mengacu pada dokumen di folder `docs/`.

Urutan membaca yang direkomendasikan:

1. `GARAGE_PRO_MASTER_DEVELOPMENT_SPECIFICATION_V1.md`
2. `GARAGE_PRO_PROJECT_ARCHITECTURE_V1.md`
3. `GARAGE_PRO_DATABASE_DESIGN_V1.md`
4. `GARAGE_PRO_DATABASE_MIGRATION_V1.md`
5. `GARAGE_PRO_BUSINESS_RULES_V1.md`
6. `GARAGE_PRO_AUTH_RBAC_V1.md`
7. `GARAGE_PRO_MASTER_DATA_V1.md`
8. `GARAGE_PRO_WORK_ORDER_ENGINE_V1.md`
9. `GARAGE_PRO_INVENTORY_ENGINE_V1.md`
10. `GARAGE_PRO_INVOICE_PAYMENT_V1.md`
11. `GARAGE_PRO_API_SPECIFICATION_V1.md`
12. `GARAGE_PRO_FRONTEND_INTEGRATION_V1.md`
13. `GARAGE_PRO_UI_UX_SCREEN_BIBLE_V1.md`
14. `GARAGE_PRO_DESIGN_SYSTEM_V1.md`
15. `GARAGE_PRO_TESTING_QA_V1.md`
16. `GARAGE_PRO_DEPLOYMENT_PRODUCTION_OPERATIONS_V1.md`

---

## 18. AI / Claude Code Development Rules

GARAGE PRO dikembangkan dengan bantuan AI coding agent seperti Claude Code.

AI coding agent **WAJIB**:

1. Membaca dokumentasi di `docs/` sebelum implementasi.
2. Tidak mengubah business rule tanpa alasan yang jelas.
3. Tidak membuat asumsi yang bertentangan dengan specification.
4. Mengikuti architecture layer.
5. Menggunakan TypeScript strict.
6. Memisahkan controller, service, repository dan validation.
7. Tidak melakukan business logic kompleks di controller.
8. Tidak melakukan direct database mutation dari frontend.
9. Menggunakan API sebagai boundary antara frontend dan backend.
10. Menggunakan transaction untuk critical financial/inventory operation.
11. Menambahkan test untuk critical business logic.
12. Menjaga backward compatibility terhadap API.
13. Tidak menghapus data transaksi secara permanen.
14. Menambahkan audit log untuk mutation penting.
15. Tidak mengubah status Work Order secara langsung.
16. Tidak mengurangi stock hanya karena estimate.
17. Tidak menerima payment yang menyebabkan overpayment.
18. Tidak mengedit invoice/payment yang sudah final tanpa workflow yang sesuai.
19. Tidak menggunakan floating point untuk uang.
20. Tidak melakukan silent failure.

### Implementation Rule

Jika terdapat konflik antara kode lama dan specification:

```text
Business Rules
      ↓
Architecture
      ↓
API Contract
      ↓
Database Design
      ↓
Implementation
```

Kode harus disesuaikan dengan specification, bukan sebaliknya, kecuali terdapat keputusan teknis baru yang terdokumentasi.

---

## 19. Git Workflow

Recommended branch:

```text
main
develop
feature/*
fix/*
hotfix/*
```

Contoh:

```bash
git checkout -b feature/auth-rbac
```

Commit convention:

```text
feat: add customer module
feat: implement work order engine
fix: prevent payment overpayment
fix: resolve stock concurrency issue
refactor: improve inventory service
test: add invoice calculation tests
docs: update api specification
chore: update dependencies
```

Pull Request harus menjelaskan:

- What changed
- Why
- Impact
- Database migration
- API changes
- Testing performed
- Breaking changes

---

## 20. Definition of Done

Sebuah feature dianggap selesai jika:

```text
[ ] Business rule implemented
[ ] Database migration ready
[ ] API implemented
[ ] Validation implemented
[ ] Authorization implemented
[ ] Frontend implemented
[ ] Loading state
[ ] Empty state
[ ] Error state
[ ] Success feedback
[ ] Audit requirement checked
[ ] Unit/API tests
[ ] Integration test if applicable
[ ] Responsive UI
[ ] Mobile workflow tested
[ ] Documentation updated
[ ] No P0/P1 defect
```

---

## 21. Golden Path

End-to-end Golden Path GARAGE PRO:

```text
Create Customer
      ↓
Create Vehicle
      ↓
Create Work Order
      ↓
Inspection
      ↓
Add Service
      ↓
Add Spare Part
      ↓
Create Estimate
      ↓
Customer Approval
      ↓
Issue Spare Part
      ↓
Mechanic Work
      ↓
QC
      ↓
READY
      ↓
Generate Invoice
      ↓
Payment
      ↓
PAID
      ↓
COMPLETED
      ↓
Service History Updated
```

Golden Path harus selalu dapat dijalankan pada environment staging sebelum production release.

---

## 22. Project Principles

GARAGE PRO mengikuti prinsip:

> **Simple for users. Strict for transactions. Traceable for management.**

Artinya:

- User interface harus sederhana.
- Business transaction harus ketat.
- Semua transaksi penting harus dapat ditelusuri.
- Data operasional harus konsisten.
- Sistem harus dapat menjelaskan dari mana sebuah angka berasal.

---

## 23. Current Repository

Repository:

`andribintang/garagepro`

GitHub:

https://github.com/andribintang/garagepro

---

## 24. License

Project ini merupakan proprietary software milik project owner.

License dan penggunaan source code akan ditentukan secara terpisah.

---

## 25. Final Objective

GARAGE PRO V1 harus menghasilkan sistem bengkel yang:

```text
FAST
RELIABLE
TRACEABLE
SECURE
MOBILE READY
SCALABLE
MAINTAINABLE
```

Dengan satu alur data terintegrasi:

```text
Customer
   ↓
Vehicle
   ↓
Work Order
   ↓
Inventory
   ↓
Invoice
   ↓
Payment
   ↓
Service History
   ↓
Reports
```

**GARAGE PRO — Workshop Management System V1**
