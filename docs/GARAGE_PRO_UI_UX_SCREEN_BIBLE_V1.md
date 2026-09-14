# GARAGE PRO --- UI/UX SCREEN BIBLE V1

**Document Type:** UI/UX Screen Specification\
**Product:** GARAGE PRO --- Workshop Management System\
**Version:** V1.0\
**Target:** General Motorcycle Workshop\
**Frontend:** React + Vite + TypeScript + Tailwind CSS\
**Design Direction:** Modern Automotive SaaS\
**Primary Users:** Owner, Admin, Mechanic, Warehouse\
**Primary Language:** Bahasa Indonesia\
**Timezone:** Asia/Jakarta\
**Status:** Development Ready\
**Date:** 14 September 2026

------------------------------------------------------------------------

# 1. Purpose

Dokumen ini mendefinisikan seluruh pengalaman UI/UX GARAGE PRO V1.

Tujuannya agar developer dapat membangun interface tanpa menebak: -
struktur halaman - navigation - komponen - field - action - permission -
loading state - empty state - error state - confirmation - responsive
behavior - mobile mechanic experience

Dokumen ini harus konsisten dengan: - GARAGE PRO --- MASTER DEVELOPMENT
SPECIFICATION V1 - GARAGE PRO --- DATABASE DESIGN & ERD V1 - GARAGE PRO
--- API SPECIFICATION V1

------------------------------------------------------------------------

# 2. UX PRINCIPLES

## 2.1 Simple

Pengguna bengkel tidak membutuhkan interface kompleks.

Prioritas:

``` text
lihat → pilih → simpan
```

## 2.2 Fast

Tugas umum harus selesai dengan sedikit langkah.

Target: - Create WO: ≤ 60 detik untuk customer existing. - Tambah jasa:
≤ 10 detik. - Tambah spare part: ≤ 10 detik. - Catat inspection item: ≤
5 detik/item. - Payment: ≤ 20 detik.

Target ini adalah UX goal, bukan hard SLA.

## 2.3 Mobile First for Mechanic

Mekanik menggunakan: - smartphone - tangan mungkin kotor - kondisi
workshop terang - waktu interaksi singkat

UI harus: - tombol besar - teks jelas - kontras tinggi - minim typing -
minim modal bertumpuk

## 2.4 Desktop First for Admin

Admin membutuhkan: - tabel - filtering - keyboard - multi-column
information - transaksi cepat

## 2.5 Owner Focused

Owner membutuhkan: - KPI - status workshop - revenue - stock alert -
trend - actionable information

------------------------------------------------------------------------

# 3. INFORMATION ARCHITECTURE

``` text
GARAGE PRO
│
├── Dashboard
│
├── Workshop
│   ├── Work Orders
│   ├── Workshop Board
│   └── QC
│
├── Customers
│
├── Vehicles
│
├── Services
│
├── Spare Parts
│
├── Inventory
│   ├── Stock
│   ├── Stock Movement
│   ├── Low Stock
│   ├── Stock Opname
│   └── Adjustment
│
├── Purchasing
│   ├── Suppliers
│   ├── Purchases
│   └── Receiving
│
├── Transactions
│   ├── Invoices
│   └── Payments
│
├── Reports
│
└── Settings
    ├── Users
    ├── Roles
    ├── Workshop
    └── General
```

------------------------------------------------------------------------

# 4. RESPONSIVE BREAKPOINTS

Use Tailwind conventions.

``` text
Mobile       < 640px
Tablet       640–1023px
Desktop      >= 1024px
Large        >= 1280px
```

Behavior:

### Mobile

-   sidebar becomes drawer
-   tables become cards or horizontally scrollable
-   primary action becomes sticky bottom action where appropriate

### Tablet

-   compact sidebar/drawer
-   2-column forms where possible

### Desktop

-   persistent sidebar
-   multi-column layout
-   full data tables

------------------------------------------------------------------------

# 5. APPLICATION SHELL

## Desktop

``` text
┌──────────────┬─────────────────────────────────────┐
│              │ Header                              │
│   Sidebar    ├─────────────────────────────────────┤
│              │                                     │
│              │ Main Content                        │
│              │                                     │
│              │                                     │
└──────────────┴─────────────────────────────────────┘
```

Sidebar width:

``` text
240–260px
```

Header:

``` text
64px
```

## Mobile

``` text
┌────────────────────────────┐
│ ☰ GARAGE PRO        👤     │
├────────────────────────────┤
│                            │
│ Content                    │
│                            │
└────────────────────────────┘
```

------------------------------------------------------------------------

# 6. GLOBAL HEADER

Components: - Sidebar toggle - Page title - Breadcrumb where useful -
Search shortcut - Notification - User menu

User menu: - Profile - Change password - Logout

------------------------------------------------------------------------

# 7. GLOBAL COMPONENTS

Required reusable components:

``` text
Button
IconButton
Input
Textarea
Select
Combobox
SearchInput
DatePicker
DateRangePicker
NumberInput
CurrencyInput
Checkbox
Radio
Switch
Tabs
Badge
Card
Table
DataTable
Pagination
Modal
Drawer
Sheet
Dropdown
Tooltip
Toast
Alert
ConfirmDialog
Skeleton
EmptyState
ErrorState
LoadingState
StatCard
Timeline
KanbanColumn
KanbanCard
```

------------------------------------------------------------------------

# 8. GLOBAL UI STATES

Every data-driven screen must support:

## Loading

Use skeletons.

Example:

``` text
████████████████
████████
██████████████
```

## Empty

``` text
Belum ada Work Order

Belum ada transaksi untuk periode ini.

[ + Buat Work Order ]
```

## Error

``` text
Gagal memuat data

Terjadi masalah saat mengambil data.

[ Coba Lagi ]
```

## Forbidden

``` text
Akses Ditolak

Anda tidak memiliki izin untuk mengakses halaman ini.
```

## Not Found

``` text
Data Tidak Ditemukan
```

------------------------------------------------------------------------

# 9. GLOBAL FORM RULES

1.  Label selalu terlihat.
2.  Required field memiliki `*`.
3.  Validation error muncul dekat field.
4.  Jangan hanya mengandalkan warna.
5.  Submit button memiliki loading state.
6.  Prevent double submit.
7.  Unsaved changes harus diperingatkan.
8.  Currency menggunakan format Rupiah.
9.  Number input tidak menggunakan browser spinner bila mengganggu UX.
10. Error API ditampilkan dengan bahasa yang mudah dipahami.

------------------------------------------------------------------------

# 10. CURRENCY DISPLAY

Display:

``` text
Rp 75.000
Rp 1.250.000
```

Input dapat menerima:

``` text
75000
75.000
```

Backend menerima numeric value.

------------------------------------------------------------------------

# 11. STATUS BADGE

Status harus konsisten.

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

Display labels:

  Internal           UI
  ------------------ ----------------------
  NEW                Baru
  CHECKING           Pemeriksaan
  ESTIMATE           Estimasi
  WAITING_APPROVAL   Menunggu Persetujuan
  APPROVED           Disetujui
  IN_PROGRESS        Dikerjakan
  QC                 QC
  REWORK             Perbaikan
  READY              Siap Diambil
  INVOICED           Ditagihkan
  PAID               Lunas
  COMPLETED          Selesai
  CANCELLED          Dibatalkan

------------------------------------------------------------------------

# 12. SCREEN 01 --- LOGIN

Route:

``` text
/login
```

Audience: - all users

Layout:

``` text
┌────────────────────────────────┐
│                                │
│          GARAGE PRO             │
│   Workshop Management System   │
│                                │
│  Username                      │
│  [________________________]    │
│                                │
│  Password                      │
│  [________________________] 👁 │
│                                │
│  [        MASUK        ]       │
│                                │
└────────────────────────────────┘
```

States: - idle - loading - invalid credential - inactive account -
server error

Success: → `/dashboard`

------------------------------------------------------------------------

# 13. SCREEN 02 --- DASHBOARD

Route:

``` text
/dashboard
```

Audience: - Owner - Admin - Mechanic limited - Warehouse limited

## Header

``` text
Dashboard
14 September 2026

[ Today ▼ ]
```

## KPI

``` text
┌──────────────┐
│ WO Hari Ini  │
│     18       │
└──────────────┘

┌──────────────┐
│ Omzet        │
│ Rp 4,85 jt   │
└──────────────┘

┌──────────────┐
│ Dikerjakan   │
│      5       │
└──────────────┘

┌──────────────┐
│ Siap Diambil │
│      4       │
└──────────────┘
```

Sections: 1. KPI 2. Workshop status 3. Revenue trend 4. Low stock 5.
Recent WO 6. Recent payment

Primary action:

``` text
+ Work Order Baru
```

------------------------------------------------------------------------

# 14. SCREEN 03 --- WORK ORDER LIST

Route:

``` text
/work-orders
```

Header:

``` text
Work Orders

[ + Work Order Baru ]
```

Filters: - Search - Date - Status - Mechanic

Table:

``` text
WO
Customer
Vehicle
Mechanic
KM
Status
Opened
Action
```

Search placeholder:

``` text
Cari WO, customer, plat nomor...
```

Row action: - View - Edit - More

Mobile: Use cards.

------------------------------------------------------------------------

# 15. SCREEN 04 --- CREATE WORK ORDER

Route:

``` text
/work-orders/new
```

## Step 1 Customer

``` text
Customer
[ Cari customer... ]

[ + Customer Baru ]
```

If customer selected:

``` text
Budi Santoso
08123456789
2 kendaraan
```

## Step 2 Vehicle

``` text
Kendaraan
[ Pilih kendaraan... ]

[ + Kendaraan Baru ]
```

## Step 3 Mechanic

``` text
Mekanik
[ Pilih mekanik ]
```

## Step 4 Vehicle Condition

``` text
KM Saat Ini
[ 18.250 ]

Keluhan Customer
[________________________]
```

## Step 5 Notes

``` text
Catatan
[________________________]
```

Actions:

``` text
[Batal] [Simpan Work Order]
```

Success: → `/work-orders/:id`

Status:

``` text
NEW
```

------------------------------------------------------------------------

# 16. SCREEN 05 --- WORK ORDER DETAIL

Route:

``` text
/work-orders/:id
```

Header:

``` text
WO-20260914-0001
Budi Santoso
Honda Beat • B 1234 XYZ

[ Pemeriksaan ]
[ Dikerjakan ]
```

Status badge.

## Summary Card

``` text
Customer
Budi Santoso

Vehicle
Honda Beat
B 1234 XYZ

KM
18.250

Mechanic
Andi

Keluhan
Motor terasa berisik
```

## Tabs

``` text
Overview
Inspection
Services
Spare Parts
Recommendations
Timeline
Invoice
```

Actions depend on status.

------------------------------------------------------------------------

# 17. SCREEN 06 --- INSPECTION

Route:

``` text
/work-orders/:id/inspection
```

Mobile-first.

Header:

``` text
Pemeriksaan
WO-0001

Honda Beat
B 1234 XYZ
```

Checklist cards:

``` text
ENGINE

○ Normal
○ Perlu Perhatian
○ Ganti
○ Perbaiki

Catatan
[________________]
```

Items:

``` text
Mesin
Oli Mesin
Rem Depan
Rem Belakang
Ban Depan
Ban Belakang
Aki
CVT
Suspensi
Kelistrikan
Lampu
Rantai
Cooling
Body
```

Quick result buttons should be touch-friendly.

Sticky bottom:

``` text
[ Simpan Pemeriksaan ]
```

------------------------------------------------------------------------

# 18. SCREEN 07 --- ADD SERVICE

Can be modal/drawer.

Title:

``` text
Tambah Jasa
```

Fields:

``` text
Cari Jasa
[ Search... ]

Kategori
[ CVT ▼ ]

Harga
Rp 75.000

Qty
[ 1 ]

Diskon
[ 0 ]
```

Search result:

``` text
Service CVT
Rp75.000
60 menit
```

Action:

``` text
[ Tambahkan ]
```

------------------------------------------------------------------------

# 19. SCREEN 08 --- ADD SPARE PART

Drawer/modal.

Fields:

``` text
Cari Spare Part
[ SKU / barcode / nama ]

Warehouse
[ Gudang Utama ]

Lokasi
[ Rak A-03 ]

Qty
[ 1 ]
```

Search result:

``` text
Kampas Rem Beat
BRK-BEAT-001

Stock: 8 PCS
Harga: Rp50.000
```

If insufficient:

``` text
⚠ Stok tidak mencukupi.

Stok tersedia: 1
Permintaan: 3
```

Button disabled when invalid.

------------------------------------------------------------------------

# 20. SCREEN 09 --- RECOMMENDATIONS

Section:

``` text
Rekomendasi
```

Card:

``` text
Ganti Ban Depan

Prioritas: Sedang
Estimasi: Rp250.000

Ban mulai aus.

[ Setujui ] [ Tolak ]
```

Status: - Recommended - Approved - Declined - Converted

Conversion action:

``` text
[ Jadikan Pekerjaan ]
```

------------------------------------------------------------------------

# 21. SCREEN 10 --- WORKSHOP BOARD

Route:

``` text
/workshop/board
```

Desktop Kanban:

``` text
┌─────────┬──────────┬────────────┬──────┬────────┐
│ BARU    │ CEK      │ DIKERJAKAN │ QC   │ READY  │
├─────────┼──────────┼────────────┼──────┼────────┤
│ WO-001  │ WO-003   │ WO-002     │WO-04 │ WO-005 │
│ Beat    │ Vario    │ NMAX       │Beat  │ Scoopy │
└─────────┴──────────┴────────────┴──────┴────────┘
```

Card:

``` text
WO-002
Honda NMAX
B 1234 ABC

Andi
Service CVT

[ View ]
```

Mobile: - horizontal scroll columns - or list grouped by status

------------------------------------------------------------------------

# 22. SCREEN 11 --- QC

Route:

``` text
/work-orders/:id/qc
```

Fields:

``` text
Final KM
[ 18.260 ]

QC Checklist

☐ Mesin normal
☐ Tidak ada kebocoran
☐ Rem normal
☐ Lampu normal
☐ Test ride
☐ Area kerja bersih

Catatan
[________________]
```

Actions:

``` text
[ Lulus QC ]
[ Perlu Perbaikan ]
```

If rework:

``` text
Alasan Perbaikan
[________________]
```

------------------------------------------------------------------------

# 23. SCREEN 12 --- CUSTOMER LIST

Route:

``` text
/customers
```

Header:

``` text
Customers
[ + Customer Baru ]
```

Search:

``` text
Cari nama / nomor HP / plat...
```

Table:

``` text
Customer
Phone
Vehicles
Last Service
Visits
Status
```

Mobile cards:

``` text
Budi Santoso
08123456789

2 kendaraan
12 kunjungan

Terakhir:
14 Sep 2026
```

------------------------------------------------------------------------

# 24. SCREEN 13 --- CUSTOMER DETAIL

Route:

``` text
/customers/:id
```

Header:

``` text
Budi Santoso
08123456789

[ + Work Order ]
```

Tabs:

``` text
Overview
Vehicles
Service History
Transactions
```

Summary:

``` text
Total Visit
12

Total Spend
Rp 4.250.000
```

------------------------------------------------------------------------

# 25. SCREEN 14 --- CREATE CUSTOMER

Fields:

``` text
Nama *
Nomor HP *
Email
Alamat
Catatan
```

Action:

``` text
[Batal] [Simpan Customer]
```

After save: Offer:

``` text
Customer berhasil dibuat.

[ Tambah Kendaraan ]
[ Selesai ]
```

------------------------------------------------------------------------

# 26. SCREEN 15 --- EDIT CUSTOMER

Same as create.

Show: - created date - updated date

Soft-deactivation available based on permission.

------------------------------------------------------------------------

# 27. SCREEN 16 --- VEHICLE LIST

Route:

``` text
/vehicles
```

Columns:

``` text
Plate
Customer
Brand
Model
KM
Last Service
```

Actions: - Detail - Edit

------------------------------------------------------------------------

# 28. SCREEN 17 --- VEHICLE DETAIL

Header:

``` text
Honda Beat
B 1234 XYZ

Budi Santoso

KM 18.250

[ + Work Order ]
```

Cards: - Vehicle info - Owner - Last service - Recommendations

------------------------------------------------------------------------

# 29. SCREEN 18 --- VEHICLE SERVICE HISTORY

Timeline:

``` text
14 SEP 2026
18.250 KM

Service CVT
Ganti Oli
Ganti Kampas Rem

Rp190.000
```

Expandable: - services - parts - invoice - notes

------------------------------------------------------------------------

# 30. SCREEN 19 --- SERVICE LIST

Route:

``` text
/services
```

Columns:

``` text
Code
Service
Category
Price
Duration
Status
```

Actions: - Edit - Deactivate

Primary:

``` text
+ Jasa Baru
```

------------------------------------------------------------------------

# 31. SCREEN 20 --- SERVICE FORM

Fields:

``` text
Kode Jasa *
Nama Jasa *
Kategori *
Harga *
Durasi
Deskripsi
Status
```

Price input:

``` text
Rp 75.000
```

------------------------------------------------------------------------

# 32. SCREEN 21 --- SERVICE CATEGORY

CRUD table:

``` text
Code
Category
Status
Action
```

Create/edit modal.

------------------------------------------------------------------------

# 33. SCREEN 22 --- SPARE PART LIST

Route:

``` text
/parts
```

Filters: - Search - Category - Brand - Low stock - Status

Columns:

``` text
SKU
Part
Brand
Stock
Min
Selling Price
Status
```

Low stock badge:

``` text
LOW STOCK
```

------------------------------------------------------------------------

# 34. SCREEN 23 --- SPARE PART DETAIL

Sections:

``` text
Part Information
Pricing
Stock by Warehouse
Movement History
Usage History
```

Stock:

``` text
Gudang Utama
8 PCS

Minimum
5 PCS
```

------------------------------------------------------------------------

# 35. SCREEN 24 --- SPARE PART FORM

Fields:

``` text
SKU *
Barcode
Nama *
Kategori *
Brand
Unit *
Harga Beli *
Harga Jual *
Minimum Stock *
Maximum Stock
Warehouse Location
Status
```

------------------------------------------------------------------------

# 36. SCREEN 25 --- PART CATEGORY

CRUD list.

------------------------------------------------------------------------

# 37. SCREEN 26 --- STOCK OVERVIEW

Route:

``` text
/inventory
```

KPIs:

``` text
Total SKU
Total Unit
Stock Value
Low Stock
```

Table:

``` text
SKU
Part
Warehouse
Stock
Minimum
Status
```

------------------------------------------------------------------------

# 38. SCREEN 27 --- STOCK MOVEMENT

Route:

``` text
/inventory/movements
```

Filters:

``` text
Date
Part
Warehouse
Movement Type
Reference
```

Table:

``` text
Date
Part
Type
Reference
Qty
Balance
User
```

Example:

``` text
14 Sep
Kampas Rem
OUT
WO-001
-1
8
Andi
```

------------------------------------------------------------------------

# 39. SCREEN 28 --- LOW STOCK

Route:

``` text
/inventory/low-stock
```

Cards/table:

``` text
Kampas Rem Beat
Stock 2
Minimum 5

Shortage: 3

[ Buat Pembelian ]
```

------------------------------------------------------------------------

# 40. SCREEN 29 --- STOCK OPNAME

Route:

``` text
/inventory/stock-opname
```

List:

``` text
Opname #
Warehouse
Date
Status
Created By
```

Create flow:

``` text
Select Warehouse
      ↓
Generate Items
      ↓
Counting
      ↓
Review Variance
      ↓
Post
```

------------------------------------------------------------------------

# 41. SCREEN 30 --- STOCK ADJUSTMENT

Route:

``` text
/inventory/adjustment
```

Fields:

``` text
Part *
Warehouse *
Location
Adjustment Type *
Quantity *
Reason *
```

Confirmation:

``` text
Anda akan mengubah stok.

Part: Kampas Rem
Adjustment: -2 PCS

[ Batal ] [ Konfirmasi ]
```

------------------------------------------------------------------------

# 42. SCREEN 31 --- SUPPLIER LIST

Route:

``` text
/suppliers
```

Columns:

``` text
Code
Supplier
Phone
Email
Status
```

Actions: - Detail - Edit

------------------------------------------------------------------------

# 43. SCREEN 32 --- PURCHASE LIST

Route:

``` text
/purchases
```

Columns:

``` text
PO Number
Supplier
Date
Total
Status
```

Primary:

``` text
+ Pembelian Baru
```

------------------------------------------------------------------------

# 44. SCREEN 33 --- PURCHASE FORM

Fields:

``` text
Supplier *
Warehouse *

Items

[ + Tambah Spare Part ]

Part
Qty
Unit Cost
Subtotal

Discount

Total
```

Server calculates total.

------------------------------------------------------------------------

# 45. SCREEN 34 --- RECEIVING

Route:

``` text
/purchases/:id/receiving
```

Display:

``` text
PO-20260914-0001

Kampas Rem
Ordered: 10
Received: 0

Receive:
[ 10 ]
```

Action:

``` text
[ Terima Barang ]
```

Success: - stock increases - movement created - purchase status updated

------------------------------------------------------------------------

# 46. SCREEN 35 --- INVOICE LIST

Route:

``` text
/invoices
```

Columns:

``` text
Invoice
Customer
Vehicle
Total
Payment Status
Date
```

Actions: - View - Print - Payment

------------------------------------------------------------------------

# 47. SCREEN 36 --- INVOICE DETAIL

Header:

``` text
INV-20260914-0001

Budi Santoso
Honda Beat
B 1234 XYZ
```

Sections:

``` text
JASA
Service CVT          Rp75.000

SPARE PART
Oli                  Rp55.000
Kampas Rem           Rp50.000

Subtotal             Rp180.000
Discount              Rp5.000
TOTAL                Rp175.000
```

Actions:

``` text
[ Print ]
[ PDF ]
[ Bayar ]
```

------------------------------------------------------------------------

# 48. SCREEN 37 --- PAYMENT

Route:

``` text
/invoices/:id/payment
```

Large summary:

``` text
TOTAL
Rp 190.000

SUDAH BAYAR
Rp 0

SISA
Rp 190.000
```

Fields:

``` text
Payment Method
○ Cash
○ Transfer
○ QRIS
○ Debit
○ Credit Card
○ Other

Amount
[ Rp 190.000 ]

Reference
[ Optional ]
```

Action:

``` text
[ PROSES PEMBAYARAN ]
```

Success:

``` text
Pembayaran berhasil.

[ Cetak Struk ]
[ Selesai ]
```

------------------------------------------------------------------------

# 49. SCREEN 38 --- PAYMENT HISTORY

Route:

``` text
/payments
```

Columns:

``` text
Payment #
Invoice
Customer
Method
Amount
Date
Received By
```

------------------------------------------------------------------------

# 50. SCREEN 39 --- REVENUE REPORT

Route:

``` text
/reports/revenue
```

Filters:

``` text
Date Range
Payment Method
```

KPIs:

``` text
Gross Revenue
Discount
Net Revenue
Service Revenue
Part Revenue
Transactions
```

Chart: - daily revenue

Table: - date - service - parts - total

Export:

``` text
CSV
PDF
```

------------------------------------------------------------------------

# 51. SCREEN 40 --- WORK ORDER REPORT

Metrics:

``` text
Total WO
Completed
Cancelled
In Progress
Average Ticket
```

Filters: - date - mechanic - status

------------------------------------------------------------------------

# 52. SCREEN 41 --- SERVICE REPORT

Show: - top services - quantity - revenue

Chart: - top 10 services

------------------------------------------------------------------------

# 53. SCREEN 42 --- PART REPORT

Show: - top parts - consumed quantity - revenue - cost - margin

------------------------------------------------------------------------

# 54. SCREEN 43 --- MECHANIC REPORT

Show:

``` text
Mechanic
Assigned WO
Completed
Service Revenue
Part Revenue
Average Ticket
```

Owner/Admin only.

------------------------------------------------------------------------

# 55. SCREEN 44 --- STOCK REPORT

Show:

``` text
Total SKU
Total Units
Stock Value
Low Stock
```

Filters: - warehouse - category

------------------------------------------------------------------------

# 56. SCREEN 45 --- USERS

Route:

``` text
/settings/users
```

Columns:

``` text
Name
Username
Role
Status
Last Login
```

Actions: - Edit - Activate - Deactivate - Reset Password

------------------------------------------------------------------------

# 57. SCREEN 46 --- ROLES

Route:

``` text
/settings/roles
```

Permission matrix:

``` text
              OWNER ADMIN MECHANIC WAREHOUSE

Customer View    ✓     ✓      ✓
Customer Create  ✓     ✓
WO View          ✓     ✓      ✓
WO Approve       ✓     ✓
Issue Part       ✓     ✓      ✓       ✓
Payment          ✓     ✓
Inventory        ✓     ✓              ✓
```

------------------------------------------------------------------------

# 58. SCREEN 47 --- WORKSHOP SETTINGS

Route:

``` text
/settings/workshop
```

Fields:

``` text
Workshop Name
Address
Phone
Email
Logo
Invoice Footer
```

Preview invoice option.

------------------------------------------------------------------------

# 59. SCREEN 48 --- GENERAL SETTINGS

Route:

``` text
/settings/general
```

Fields:

``` text
Currency
Tax Enabled
Tax Rate
Timezone
Default Warehouse
```

Default:

``` text
Currency: IDR
Timezone: Asia/Jakarta
```

------------------------------------------------------------------------

# 60. SCREEN 49 --- AUDIT LOG

Route:

``` text
/settings/audit-logs
```

Filters: - User - Action - Entity - Date

Table:

``` text
Date
User
Action
Entity
Reference
```

Detail drawer:

``` text
Old Values
New Values
IP
User Agent
```

------------------------------------------------------------------------

# 61. GLOBAL SEARCH

Desktop shortcut:

``` text
Ctrl + K
```

Search across:

``` text
Customer
Vehicle
WO
Invoice
Spare Part
SKU
```

Result grouping:

``` text
CUSTOMERS
Budi Santoso

VEHICLES
B 1234 XYZ

WORK ORDERS
WO-20260914-0001

INVOICES
INV-20260914-0001
```

------------------------------------------------------------------------

# 62. QUICK CREATE

Global quick action:

``` text
+ New
```

Options:

``` text
Work Order
Customer
Vehicle
Service
Spare Part
Purchase
```

Role-based visibility.

------------------------------------------------------------------------

# 63. NOTIFICATION CENTER

Future-ready UI.

V1 notifications: - Low stock - Payment completed - Vehicle ready -
Stock opname pending

Do not implement complex notification rules in V1.

------------------------------------------------------------------------

# 64. MECHANIC MOBILE HOME

Mechanic should not see the full owner dashboard.

Home:

``` text
Halo, Andi

Pekerjaan Saya

┌──────────────────────┐
│ WO-001               │
│ Honda Beat           │
│ B 1234 XYZ           │
│                      │
│ PEMERIKSAAN          │
│                      │
│ [ Buka ]             │
└──────────────────────┘

┌──────────────────────┐
│ WO-004               │
│ Honda Vario          │
│                      │
│ DIKERJAKAN           │
└──────────────────────┘
```

Primary navigation:

``` text
Home
Pekerjaan
Riwayat
Profile
```

------------------------------------------------------------------------

# 65. MECHANIC MOBILE WO

Top:

``` text
WO-001
Honda Beat
B 1234 XYZ

CHECKING
```

Action tabs:

``` text
Pemeriksaan
Jasa
Part
Catatan
```

Bottom action:

``` text
[ Simpan & Lanjut ]
```

------------------------------------------------------------------------

# 66. ADMIN DESKTOP WORKFLOW

Typical flow:

``` text
Customer datang
      ↓
Search customer
      ↓
Select vehicle
      ↓
Create WO
      ↓
Assign mechanic
      ↓
Monitor board
      ↓
Review estimate
      ↓
Invoice
      ↓
Payment
```

------------------------------------------------------------------------

# 67. MECHANIC MOBILE WORKFLOW

``` text
Open WO
   ↓
Read complaint
   ↓
Inspection
   ↓
Add diagnosis
   ↓
Add service
   ↓
Add parts
   ↓
Add recommendation
   ↓
Save
   ↓
Work
   ↓
QC
```

------------------------------------------------------------------------

# 68. WAREHOUSE WORKFLOW

``` text
Purchase
   ↓
Receive
   ↓
Stock IN
   ↓
Part requested
   ↓
Issue
   ↓
Stock OUT
```

------------------------------------------------------------------------

# 69. OWNER WORKFLOW

``` text
Dashboard
   ↓
Workshop Board
   ↓
Revenue
   ↓
Low Stock
   ↓
Mechanic Performance
   ↓
Reports
```

------------------------------------------------------------------------

# 70. MODAL RULES

Use modal for: - quick create - confirmation - simple edit - add
service - add part - payment confirmation

Use full page for: - Work Order - Inspection - Purchase - Stock Opname -
Reports - Settings

Use drawer for: - details - timeline - audit log - quick item selection

------------------------------------------------------------------------

# 71. CONFIRMATION DIALOGS

Critical actions require confirmation.

Examples:

### Cancel WO

``` text
Batalkan Work Order?

WO-001
Honda Beat

Tindakan ini akan menghentikan proses WO.

Alasan
[________________]

[ Kembali ] [ Batalkan WO ]
```

### Stock Adjustment

``` text
Konfirmasi Penyesuaian Stok?

Kampas Rem
-2 PCS

Alasan:
Barang rusak

[ Batal ] [ Konfirmasi ]
```

### Void Invoice

Must show strong warning.

------------------------------------------------------------------------

# 72. TOAST MESSAGES

Success:

``` text
Work Order berhasil dibuat.
```

``` text
Pemeriksaan berhasil disimpan.
```

``` text
Spare part berhasil dikeluarkan.
```

``` text
Pembayaran berhasil diproses.
```

Error:

``` text
Gagal menyimpan data.
```

Specific:

``` text
Stok tidak mencukupi.
```

------------------------------------------------------------------------

# 73. ACCESSIBILITY

Minimum: - keyboard navigation - visible focus - labels for form
controls - aria-label for icon-only buttons - sufficient contrast - no
information conveyed by color alone - confirmation for destructive
actions

------------------------------------------------------------------------

# 74. TABLE UX

Desktop: - sticky header - pagination - sorting - column alignment -
compact row height - row hover - action menu

Mobile: - card layout - essential information first - secondary details
expandable

------------------------------------------------------------------------

# 75. FORM UX

Long forms should be split into sections.

Example Work Order:

``` text
Customer
Vehicle
Work Details
Notes
```

Use progressive disclosure.

------------------------------------------------------------------------

# 76. EMPTY STATES

Customer:

``` text
Belum ada customer.

Tambahkan customer pertama untuk mulai membuat Work Order.

[ + Customer Baru ]
```

Vehicle:

``` text
Customer ini belum memiliki kendaraan.

[ + Tambah Kendaraan ]
```

WO:

``` text
Belum ada Work Order.

[ + Work Order Baru ]
```

Inventory:

``` text
Belum ada data spare part.
```

------------------------------------------------------------------------

# 77. OFFLINE / CONNECTION STATE

PWA should show:

``` text
● Online
```

If connection lost:

``` text
⚠ Koneksi terputus

Perubahan belum dapat disimpan.
```

V1 must not pretend that an unsaved transaction succeeded.

------------------------------------------------------------------------

# 78. UNSAVED CHANGES

If form modified:

``` text
Ada perubahan yang belum disimpan.

Yakin ingin meninggalkan halaman?

[ Tetap di Halaman ]
[ Tinggalkan ]
```

------------------------------------------------------------------------

# 79. PRINT UX

Invoice print preview:

``` text
[ Print ]
[ Download PDF ]
```

Thermal layout: - 58mm - 80mm

A4 optional for reports.

------------------------------------------------------------------------

# 80. DESIGN TOKEN PLACEHOLDERS

Final values are defined in:

`GARAGE_PRO_DESIGN_SYSTEM_V1.md`

UI should consume tokens rather than hard-code styling values throughout
components.

Concept:

``` text
color.primary
color.surface
color.text
color.muted
color.success
color.warning
color.danger

radius.sm
radius.md
radius.lg

spacing.1
spacing.2
spacing.4
spacing.6
```

------------------------------------------------------------------------

# 81. COMPONENT COMPOSITION

Example Work Order page:

``` text
WorkOrderPage
 ├── PageHeader
 ├── WorkOrderStatusBadge
 ├── VehicleSummaryCard
 ├── CustomerSummaryCard
 ├── WorkOrderTabs
 │    ├── OverviewTab
 │    ├── InspectionTab
 │    ├── ServicesTab
 │    ├── PartsTab
 │    ├── RecommendationsTab
 │    ├── TimelineTab
 │    └── InvoiceTab
 └── WorkOrderActionBar
```

------------------------------------------------------------------------

# 82. FRONTEND ROUTE MAP

``` text
/login

/dashboard

/work-orders
/work-orders/new
/work-orders/:id
/work-orders/:id/inspection
/work-orders/:id/qc

/workshop/board

/customers
/customers/new
/customers/:id
/customers/:id/edit

/vehicles
/vehicles/new
/vehicles/:id
/vehicles/:id/edit
/vehicles/:id/history

/services
/services/new
/services/:id/edit
/service-categories

/parts
/parts/new
/parts/:id
/parts/:id/edit
/part-categories

/inventory
/inventory/movements
/inventory/low-stock
/inventory/stock-opname
/inventory/adjustment

/suppliers
/purchases
/purchases/new
/purchases/:id
/purchases/:id/receiving

/invoices
/invoices/:id
/invoices/:id/payment

/payments

/reports/revenue
/reports/work-orders
/reports/services
/reports/parts
/reports/mechanics
/reports/stock

/settings/users
/settings/roles
/settings/workshop
/settings/general
/settings/audit-logs
```

------------------------------------------------------------------------

# 83. SCREEN PERMISSION SUMMARY

  Screen             Owner       Admin     Mechanic   Warehouse
  ---------------- ------- ----------- ------------ -----------
  Dashboard              ✓           ✓      Limited     Limited
  WO List                ✓           ✓     Assigned        Read
  WO Detail              ✓           ✓     Assigned        Read
  Inspection             ✓   Read/Edit            ✓          \-
  QC                     ✓           ✓   Authorized          \-
  Customer               ✓           ✓         Read          \-
  Vehicle                ✓           ✓         Read          \-
  Service Master         ✓           ✓         Read          \-
  Part Master            ✓           ✓         Read           ✓
  Inventory              ✓        View         Read           ✓
  Purchase               ✓           ✓           \-           ✓
  Invoice                ✓           ✓         Read          \-
  Payment                ✓           ✓           \-          \-
  Reports                ✓           ✓           \-   Inventory
  Users                  ✓          \-           \-          \-
  Settings               ✓          \-           \-          \-
  Audit                  ✓     Limited           \-     Limited

------------------------------------------------------------------------

# 84. UX ACCEPTANCE CRITERIA

## Work Order

-   Admin can create WO without unnecessary screens.
-   Existing customer can be selected quickly.
-   Vehicle selection is linked to customer.
-   Mechanic can open assigned WO from mobile.
-   Status is always visible.
-   User cannot execute invalid status transition.

## Inspection

-   Mechanic can complete checklist using touch.
-   Notes can be added.
-   Recommendation can be created.
-   Save state is obvious.

## Spare Part

-   Available stock visible before issue.
-   Insufficient stock clearly indicated.
-   Stock issue requires permission.
-   Double submission is prevented.

## Invoice

-   Total is clearly displayed.
-   Server-calculated amount is displayed.
-   Payment status is obvious.
-   Print action available.

## Mobile

-   No horizontal page overflow.
-   Buttons are touch-friendly.
-   Primary actions remain accessible.
-   No critical information hidden only behind hover.

------------------------------------------------------------------------

# 85. UX QUALITY CHECKLIST

Before releasing a screen:

``` text
[ ] Loading state
[ ] Empty state
[ ] Error state
[ ] Permission state
[ ] Validation
[ ] Success feedback
[ ] Destructive confirmation
[ ] Mobile layout
[ ] Tablet layout
[ ] Desktop layout
[ ] Keyboard navigation
[ ] No console errors
[ ] API error handled
[ ] Double-submit prevented
[ ] Unsaved changes handled
```

------------------------------------------------------------------------

# 86. IMPLEMENTATION ORDER

Frontend should be implemented in this order:

``` text
1. App Shell
2. Design System Components
3. Login
4. Dashboard
5. Customer
6. Vehicle
7. Mechanic
8. Service Master
9. Spare Part
10. Inventory
11. Work Order
12. Inspection
13. Recommendation
14. Workshop Board
15. QC
16. Purchase
17. Invoice
18. Payment
19. Reports
20. Settings
21. Audit
22. PWA polish
```

------------------------------------------------------------------------

# 87. NEXT DOCUMENT

After this Screen Bible, create:

``` text
GARAGE_PRO_DESIGN_SYSTEM_V1.md
```

It should define: - color tokens - typography scale - spacing - radius -
shadows - buttons - inputs - tables - cards - badges - modal - drawer -
navigation - charts - responsive rules - accessibility - Tailwind
implementation tokens

After Design System, proceed to:

``` text
GARAGE_PRO_PROJECT_ARCHITECTURE_V1.md
```

and then actual source-code implementation.

------------------------------------------------------------------------

# END OF DOCUMENT

**GARAGE PRO --- UI/UX SCREEN BIBLE V1**\
**Status: Development Ready**
