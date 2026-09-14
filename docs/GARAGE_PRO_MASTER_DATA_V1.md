# GARAGE PRO --- MASTER DATA V1

**Project:** GARAGE PRO --- Workshop Management System\
**Document:** Master Data Specification\
**Version:** V1.0\
**Status:** Implementation Ready\
**Backend:** Node.js + Express + TypeScript\
**Frontend:** React + Vite + TypeScript\
**Database:** MySQL 8+\
**ORM:** Sequelize\
**Validation:** Zod\
**Data Fetching:** TanStack Query

------------------------------------------------------------------------

# 1. Tujuan

Phase 11 membangun seluruh master data yang menjadi fondasi transaksi
GARAGE PRO.

``` text
Customer
   ↓
Vehicle
   ↓
Work Order
   ↓
Service + Spare Part
   ↓
Warehouse / Inventory
   ↓
Invoice
```

Master data:

``` text
1. Customer
2. Vehicle
3. Mechanic
4. Service Category
5. Service
6. Part Category
7. Spare Part
8. Supplier
9. Warehouse
10. Warehouse Location
11. Inspection Item
```

Prinsip:

> Master data harus bersih, searchable, memiliki validation yang
> konsisten, mendukung soft delete, dan aman digunakan oleh transaksi.

------------------------------------------------------------------------

# 2. Master Data Dependency

``` text
Service Category
      ↓
Service

Part Category
      ↓
Spare Part
      ↓
Warehouse
      ↓
Warehouse Location

Customer
      ↓
Vehicle

User
      ↓
Mechanic

Supplier

Inspection Item
```

Untuk transaksi:

``` text
Customer + Vehicle + Mechanic
          ↓
       Work Order
          ↓
Service + Spare Part
          ↓
Warehouse
```

------------------------------------------------------------------------

# 3. Global Master Data Rules

Semua master:

-   menggunakan `id` BIGINT UNSIGNED;
-   menggunakan timestamps;
-   menggunakan `deleted_at` jika mendukung soft delete;
-   memiliki `is_active` jika perlu status operasional;
-   tidak boleh dihapus jika sudah direferensikan transaksi;
-   pencarian tidak case-sensitive;
-   pagination server-side;
-   sorting whitelist;
-   filtering server-side;
-   validation backend wajib;
-   frontend validation hanya sebagai UX;
-   duplicate business code harus ditolak.

------------------------------------------------------------------------

# 4. Global CRUD Pattern

Untuk entity master:

``` text
GET    /resource
POST   /resource
GET    /resource/:id
PUT    /resource/:id
DELETE /resource/:id
```

Delete secara default:

``` text
soft delete
```

Jika data sudah dipakai transaksi:

``` text
do not hard delete
```

Response standard:

``` json
{
  "success": true,
  "data": {}
}
```

List:

``` json
{
  "success": true,
  "data": [],
  "meta": {
    "page": 1,
    "pageSize": 20,
    "total": 100,
    "totalPages": 5
  }
}
```

------------------------------------------------------------------------

# 5. Pagination

Default:

``` text
page = 1
pageSize = 20
```

Maximum:

``` text
pageSize = 100
```

Query:

``` text
?page=1&pageSize=20
```

Backend wajib membatasi page size.

------------------------------------------------------------------------

# 6. Search

Gunakan:

``` text
?search=keyword
```

Contoh Customer:

``` text
GET /api/v1/customers?search=budi
```

Search field:

``` text
customer_code
name
phone
email
```

Vehicle:

``` text
plate_number
brand
model
vin
engine_number
```

Part:

``` text
sku
part_number
name
barcode
brand
```

------------------------------------------------------------------------

# 7. Sorting

Format:

``` text
?sortBy=name&sortOrder=asc
```

Backend hanya boleh menerima whitelist field.

Jangan langsung memasukkan parameter user ke SQL ORDER BY.

------------------------------------------------------------------------

# 8. CUSTOMER

## 8.1 Purpose

Customer adalah pemilik kendaraan dan sumber service history.

------------------------------------------------------------------------

## 8.2 Fields

``` text
id
customer_code
name
phone
email
address
city
notes
is_active
created_at
updated_at
deleted_at
```

Required:

``` text
name
```

Recommended:

``` text
phone
```

------------------------------------------------------------------------

# 9. Customer Validation

Rules:

``` text
name:
  required
  2–150 characters

phone:
  optional
  max 30 chars

email:
  optional
  valid email

address:
  optional
  max 500 chars

city:
  optional
  max 100 chars

notes:
  optional
  max 1000 chars
```

Normalize:

``` text
trim whitespace
collapse repeated spaces where appropriate
```

Phone should be normalized consistently.

------------------------------------------------------------------------

# 10. Customer Code

Format:

``` text
CUS-YYYY-XXXXXX
```

Example:

``` text
CUS-2026-000001
```

Generated server-side.

Frontend tidak boleh menentukan final customer code.

------------------------------------------------------------------------

# 11. Customer API

### List

``` http
GET /api/v1/customers
```

Permission:

``` text
customers.view
```

### Create

``` http
POST /api/v1/customers
```

Permission:

``` text
customers.create
```

### Detail

``` http
GET /api/v1/customers/:id
```

Permission:

``` text
customers.view
```

### Update

``` http
PUT /api/v1/customers/:id
```

Permission:

``` text
customers.update
```

### Delete

``` http
DELETE /api/v1/customers/:id
```

Permission:

``` text
customers.delete
```

------------------------------------------------------------------------

# 12. Customer Create Example

``` json
{
  "name": "Budi Santoso",
  "phone": "081234567890",
  "email": "budi@example.com",
  "address": "Jl. Raya Bogor",
  "city": "Bogor",
  "notes": "Pelanggan rutin"
}
```

Response:

``` json
{
  "success": true,
  "data": {
    "id": 1,
    "customerCode": "CUS-2026-000001",
    "name": "Budi Santoso"
  }
}
```

------------------------------------------------------------------------

# 13. Customer Detail

Detail page harus menampilkan:

``` text
Customer Profile
├── Customer Code
├── Name
├── Phone
├── Email
├── Address
└── Notes

Vehicles
Service History
Outstanding Invoices
Recent Work Orders
```

Quick action:

``` text
+ Tambah Kendaraan
+ Buat Work Order
```

------------------------------------------------------------------------

# 14. Customer Delete Rules

Soft delete.

Reject deletion jika:

``` text
customer memiliki active WO
```

atau business policy mengharuskan record tetap aktif.

Jika sudah memiliki histori:

``` text
soft delete only
```

Error:

``` text
CUSTOMER_HAS_ACTIVE_TRANSACTION
```

------------------------------------------------------------------------

# 15. VEHICLE

## Fields

``` text
id
customer_id
plate_number
brand
model
variant
year
color
vin
engine_number
odometer
notes
created_at
updated_at
deleted_at
```

Required:

``` text
customer_id
plate_number
brand
model
```

------------------------------------------------------------------------

# 16. Vehicle Validation

``` text
plate_number:
  required
  2–20 chars

brand:
  required
  max 100

model:
  required
  max 100

variant:
  optional
  max 100

year:
  optional
  1950–currentYear+1

odometer:
  >= 0

vin:
  optional
  max 50

engine_number:
  optional
  max 100
```

Plate normalization:

``` text
uppercase
trim
```

Example:

``` text
B 1234 XYZ
```

stored consistently according to workshop convention.

------------------------------------------------------------------------

# 17. Vehicle Ownership

Vehicle wajib memiliki customer.

``` text
Customer 1 ─── N Vehicle
```

Saat membuat vehicle:

``` text
validate customer exists
validate customer active
```

------------------------------------------------------------------------

# 18. Vehicle Duplicate

Default unique:

``` text
plate_number
```

Jika duplicate:

``` text
VEHICLE_PLATE_ALREADY_EXISTS
```

Jika kendaraan berpindah owner, gunakan ownership workflow future.

Jangan membuat duplicate vehicle hanya karena owner berubah.

------------------------------------------------------------------------

# 19. Vehicle API

``` text
GET    /api/v1/vehicles
POST   /api/v1/vehicles
GET    /api/v1/vehicles/:id
PUT    /api/v1/vehicles/:id
DELETE /api/v1/vehicles/:id
```

Permissions:

``` text
vehicles.view
vehicles.create
vehicles.update
```

Delete menggunakan controlled soft-delete policy.

------------------------------------------------------------------------

# 20. Vehicle Detail

Tampilkan:

``` text
Vehicle
├── Plate
├── Brand
├── Model
├── Variant
├── Year
├── Color
├── VIN
├── Engine Number
└── Odometer

Owner
Service History
Work Orders
```

Quick action:

``` text
Buat Work Order
Lihat Riwayat Service
```

------------------------------------------------------------------------

# 21. MECHANIC

Mechanic adalah master teknisi workshop.

Fields:

``` text
id
user_id
employee_code
name
phone
specialization
is_active
created_at
updated_at
deleted_at
```

------------------------------------------------------------------------

# 22. Mechanic Rules

`employee_code` unique.

`user_id` optional.

Jika mechanic memiliki login:

``` text
mechanics.user_id → users.id
```

Jika tidak:

``` text
user_id = NULL
```

Mechanic inactive:

``` text
cannot be assigned to new WO
```

Existing historical WO tetap menampilkan mechanic tersebut.

------------------------------------------------------------------------

# 23. Mechanic API

``` text
GET    /api/v1/mechanics
POST   /api/v1/mechanics
GET    /api/v1/mechanics/:id
PUT    /api/v1/mechanics/:id
DELETE /api/v1/mechanics/:id
```

Permissions:

``` text
mechanics.view
mechanics.create
mechanics.update
```

------------------------------------------------------------------------

# 24. SERVICE CATEGORY

Fields:

``` text
id
code
name
description
is_active
created_at
updated_at
deleted_at
```

Example:

``` text
ENGINE
BRAKE
ELECTRICAL
TRANSMISSION
SUSPENSION
TUNEUP
GENERAL
```

------------------------------------------------------------------------

# 25. Service Category Validation

``` text
code:
  required
  uppercase
  max 50
  unique

name:
  required
  max 100

description:
  optional
  max 500
```

------------------------------------------------------------------------

# 26. Service Category API

``` text
GET    /api/v1/service-categories
POST   /api/v1/service-categories
GET    /api/v1/service-categories/:id
PUT    /api/v1/service-categories/:id
DELETE /api/v1/service-categories/:id
```

Permissions:

``` text
services.view
services.create
services.update
services.delete
```

------------------------------------------------------------------------

# 27. SERVICE

Fields:

``` text
id
category_id
code
name
description
estimated_duration_minutes
price
is_active
created_at
updated_at
deleted_at
```

------------------------------------------------------------------------

# 28. Service Validation

``` text
category_id:
  required
  must exist

code:
  required
  unique
  max 50

name:
  required
  max 150

estimated_duration_minutes:
  integer >= 0

price:
  decimal >= 0
```

------------------------------------------------------------------------

# 29. Service Price Rule

Master price:

``` text
services.price
```

Transaction price:

``` text
work_order_services.unit_price
```

Saat service ditambahkan ke WO:

``` text
master price
      ↓
snapshot
      ↓
WO unit_price
```

Perubahan harga master tidak mengubah histori.

------------------------------------------------------------------------

# 30. Service API

``` text
GET    /api/v1/services
POST   /api/v1/services
GET    /api/v1/services/:id
PUT    /api/v1/services/:id
DELETE /api/v1/services/:id
```

Query:

``` text
?categoryId=
?isActive=
?search=
```

------------------------------------------------------------------------

# 31. PART CATEGORY

Fields:

``` text
id
code
name
description
is_active
created_at
updated_at
deleted_at
```

Example:

``` text
ENGINE
BRAKE
ELECTRICAL
TIRE
OIL
FILTER
BATTERY
CHAIN
SUSPENSION
ACCESSORIES
```

------------------------------------------------------------------------

# 32. SPARE PART

Fields:

``` text
id
category_id
sku
part_number
name
brand
unit
purchase_price
selling_price
minimum_stock
maximum_stock
barcode
location_hint
is_active
created_at
updated_at
deleted_at
```

------------------------------------------------------------------------

# 33. Spare Part Validation

``` text
category_id:
  required

sku:
  required
  unique
  max 80

part_number:
  optional
  max 100

name:
  required
  max 200

brand:
  optional
  max 100

unit:
  required
  max 30

purchase_price:
  >= 0

selling_price:
  >= 0

minimum_stock:
  >= 0

maximum_stock:
  >= minimum_stock when provided

barcode:
  optional
  unique when provided
```

------------------------------------------------------------------------

# 34. SKU Rules

SKU harus:

``` text
unique
stable
human-readable
```

Contoh:

``` text
OIL-10W40-001
BRK-PAD-F-001
BAT-MF-001
```

SKU tidak boleh berubah tanpa controlled master-data action.

Historical WO menyimpan snapshot.

------------------------------------------------------------------------

# 35. Spare Part API

``` text
GET    /api/v1/parts
POST   /api/v1/parts
GET    /api/v1/parts/:id
PUT    /api/v1/parts/:id
DELETE /api/v1/parts/:id
```

Filters:

``` text
?categoryId=
?brand=
?isActive=
?lowStock=true
?search=
```

------------------------------------------------------------------------

# 36. Spare Part Detail

Tampilkan:

``` text
Part Profile
Current Stock
Available Stock
Minimum Stock
Warehouse Distribution
Recent Stock Movements
Purchase History
Usage History
```

Quick action:

``` text
Stock Movement
Issue
Receiving
Adjustment
```

Permission harus diperiksa sesuai action.

------------------------------------------------------------------------

# 37. SUPPLIER

Fields:

``` text
id
supplier_code
name
phone
email
address
contact_person
notes
is_active
created_at
updated_at
deleted_at
```

Validation:

``` text
supplier_code required + unique
name required
email optional valid
phone optional
```

------------------------------------------------------------------------

# 38. Supplier API

``` text
GET    /api/v1/suppliers
POST   /api/v1/suppliers
GET    /api/v1/suppliers/:id
PUT    /api/v1/suppliers/:id
DELETE /api/v1/suppliers/:id
```

Permissions:

``` text
purchases.view
purchases.create
purchases.update
```

------------------------------------------------------------------------

# 39. SUPPLIER DETAIL

Tampilkan:

``` text
Supplier Profile
Purchase Orders
Receiving History
Purchased Parts
Total Purchase Value
```

Future:

``` text
Supplier performance
lead time
price comparison
```

------------------------------------------------------------------------

# 40. WAREHOUSE

Fields:

``` text
id
code
name
address
is_main
is_active
created_at
updated_at
deleted_at
```

Rules:

``` text
code unique
name required
only one MAIN warehouse if business rule requires
```

------------------------------------------------------------------------

# 41. Warehouse API

``` text
GET    /api/v1/warehouses
POST   /api/v1/warehouses
GET    /api/v1/warehouses/:id
PUT    /api/v1/warehouses/:id
DELETE /api/v1/warehouses/:id
```

Permissions:

``` text
inventory.view
inventory.adjust
```

Creation/update should be restricted to authorized administrative users.

------------------------------------------------------------------------

# 42. WAREHOUSE LOCATION

Fields:

``` text
id
warehouse_id
code
name
description
is_active
created_at
updated_at
deleted_at
```

Unique:

``` text
warehouse_id + code
```

Example:

``` text
MAIN / RACK-A
MAIN / RACK-B
MAIN / RACK-C
```

------------------------------------------------------------------------

# 43. Inspection Item

Fields:

``` text
id
code
name
category
sequence
is_active
created_at
updated_at
deleted_at
```

Example categories:

``` text
ENGINE
BRAKE
TIRE
ELECTRICAL
BODY
SUSPENSION
TRANSMISSION
```

------------------------------------------------------------------------

# 44. Inspection Item API

``` text
GET    /api/v1/inspection-items
POST   /api/v1/inspection-items
GET    /api/v1/inspection-items/:id
PUT    /api/v1/inspection-items/:id
DELETE /api/v1/inspection-items/:id
```

Permissions:

``` text
work_orders.view
work_orders.update
```

Administrative editing can later use a dedicated permission.

------------------------------------------------------------------------

# 45. Inspection Result

Canonical:

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

These are business constants, not free text.

------------------------------------------------------------------------

# 46. Master Data API Architecture

``` text
HTTP Request
     ↓
Auth Middleware
     ↓
Permission Middleware
     ↓
Zod Validator
     ↓
Controller
     ↓
Master Data Service
     ↓
Repository
     ↓
Sequelize
     ↓
MySQL
```

Controller tidak boleh langsung query Sequelize.

------------------------------------------------------------------------

# 47. Generic CRUD Service Pattern

``` ts
class CustomerService {
  async create(input: CreateCustomerInput, actor: AuthContext) {
    validateBusinessRules(input);

    const customerCode =
      await documentNumberService.generate('CUS');

    const customer =
      await customerRepository.create({
        ...input,
        customerCode,
      });

    await auditService.log({
      action: 'CUSTOMER_CREATED',
      userId: actor.userId,
      entityType: 'CUSTOMER',
      entityId: customer.id,
    });

    return customer;
  }
}
```

------------------------------------------------------------------------

# 48. Duplicate Handling

Database unique constraint tetap wajib.

Application:

``` text
pre-check duplicate
+
database unique constraint
```

Jika race condition:

``` text
catch unique constraint error
→ convert to business error
```

Example:

``` text
CUSTOMER_CODE_ALREADY_EXISTS
VEHICLE_PLATE_ALREADY_EXISTS
SERVICE_CODE_ALREADY_EXISTS
PART_SKU_ALREADY_EXISTS
SUPPLIER_CODE_ALREADY_EXISTS
```

------------------------------------------------------------------------

# 49. Soft Delete

Default:

``` text
DELETE /resource/:id
```

berarti:

``` text
deleted_at = NOW()
```

Data tidak dihapus fisik.

List default:

``` text
deleted_at IS NULL
```

Optional admin filter:

``` text
?includeDeleted=true
```

harus memiliki permission khusus jika diimplementasikan.

------------------------------------------------------------------------

# 50. Reference Protection

Jika master sudah digunakan:

``` text
Service → WO
Part → WO
Part → Stock Movement
Customer → WO
Vehicle → WO
Supplier → Purchase
Warehouse → Stock Movement
```

jangan hard delete.

Return:

``` text
MASTER_DATA_IN_USE
```

Recommended UX:

``` text
Deactivate
```

daripada delete.

------------------------------------------------------------------------

# 51. Active/Inactive Rules

Master inactive tidak boleh digunakan untuk transaksi baru.

Contoh:

``` text
service.is_active = false
```

Maka:

``` text
cannot add to new WO
```

Namun historical WO tetap dapat menampilkan service tersebut.

Sama untuk:

``` text
part
mechanic
supplier
warehouse
```

------------------------------------------------------------------------

# 52. Quick Create

Pada Work Order:

``` text
Customer Combobox
      ↓
[+ Customer Baru]
```

Vehicle:

``` text
Vehicle Combobox
      ↓
[+ Kendaraan Baru]
```

Service/Part:

``` text
Search
      ↓
[+ Master Baru]
```

Quick create harus membuka modal/drawer.

Setelah berhasil:

``` text
create
↓
invalidate query
↓
select new record
```

------------------------------------------------------------------------

# 53. Combobox Rules

Combobox harus mendukung:

``` text
search
keyboard navigation
loading
empty state
create new
clear
selected state
```

Minimum touch target:

``` text
44px
```

Mechanic mobile:

``` text
large touch target
```

------------------------------------------------------------------------

# 54. Master Data Table

Desktop:

``` text
Checkbox
Code
Name
Category
Status
Updated
Actions
```

Actions:

``` text
View
Edit
Deactivate
Delete
```

Delete hanya jika valid.

Mobile:

``` text
Card/list
Primary information
Status
More menu
```

Jangan memaksa horizontal table pada layar kecil.

------------------------------------------------------------------------

# 55. Form UX

Form:

``` text
Label
Input
Helper text
Error
```

Required:

``` text
*
```

Submit:

``` text
Simpan
```

Loading:

``` text
Menyimpan...
```

Success:

``` text
Data berhasil disimpan.
```

Error:

``` text
Data gagal disimpan.
```

------------------------------------------------------------------------

# 56. Customer Form

Sections:

``` text
Identitas
Kontak
Alamat
Catatan
```

Fields:

``` text
Nama *
No. HP
Email
Alamat
Kota
Catatan
Status
```

------------------------------------------------------------------------

# 57. Vehicle Form

Sections:

``` text
Pemilik
Identitas Kendaraan
Nomor Identifikasi
Kilometer
Catatan
```

Fields:

``` text
Customer *
Plat Nomor *
Merk *
Model *
Varian
Tahun
Warna
VIN
Nomor Mesin
Odometer
Catatan
```

------------------------------------------------------------------------

# 58. Service Form

``` text
Kategori *
Kode *
Nama *
Deskripsi
Durasi Estimasi
Harga *
Status
```

Price:

``` text
Rp
```

Input numeric harus mencegah invalid characters.

------------------------------------------------------------------------

# 59. Spare Part Form

``` text
Kategori *
SKU *
Part Number
Nama *
Brand
Unit *
Harga Beli
Harga Jual
Minimum Stock
Maximum Stock
Barcode
Lokasi Default
Status
```

------------------------------------------------------------------------

# 60. Mechanic Form

``` text
Employee Code *
Nama *
No. HP
Spesialisasi
User Login
Status
```

Jika `user login` dipilih:

``` text
validate selected user
```

------------------------------------------------------------------------

# 61. Supplier Form

``` text
Supplier Code *
Nama *
Contact Person
Phone
Email
Address
Notes
Status
```

------------------------------------------------------------------------

# 62. Warehouse Form

``` text
Code *
Name *
Address
Main Warehouse
Status
```

Jika `is_main = true`:

``` text
validate main warehouse rule
```

------------------------------------------------------------------------

# 63. Inspection Item Form

``` text
Code *
Name *
Category
Sequence
Status
```

Sorting:

``` text
sequence ASC
```

------------------------------------------------------------------------

# 64. API Query Standards

Semua list endpoint harus mendukung jika relevan:

``` text
page
pageSize
search
sortBy
sortOrder
isActive
```

Entity-specific:

``` text
categoryId
customerId
warehouseId
brand
lowStock
```

Backend harus mengabaikan atau menolak parameter yang tidak dikenal
sesuai API policy.

------------------------------------------------------------------------

# 65. API Validation Standard

Gunakan Zod:

``` ts
const createCustomerSchema = z.object({
  name: z.string().trim().min(2).max(150),
  phone: z.string().trim().max(30).optional(),
  email: z.string().email().optional(),
  address: z.string().max(500).optional(),
  city: z.string().max(100).optional(),
  notes: z.string().max(1000).optional(),
});
```

Input dari request tidak boleh dipercaya.

------------------------------------------------------------------------

# 66. Master Data Error Codes

``` text
MASTER_NOT_FOUND
MASTER_DATA_IN_USE
MASTER_DATA_INACTIVE

CUSTOMER_NOT_FOUND
CUSTOMER_CODE_ALREADY_EXISTS

VEHICLE_NOT_FOUND
VEHICLE_PLATE_ALREADY_EXISTS

MECHANIC_NOT_FOUND
MECHANIC_CODE_ALREADY_EXISTS

SERVICE_CATEGORY_NOT_FOUND
SERVICE_CATEGORY_CODE_ALREADY_EXISTS

SERVICE_NOT_FOUND
SERVICE_CODE_ALREADY_EXISTS

PART_CATEGORY_NOT_FOUND
PART_CATEGORY_CODE_ALREADY_EXISTS

PART_NOT_FOUND
PART_SKU_ALREADY_EXISTS
PART_BARCODE_ALREADY_EXISTS

SUPPLIER_NOT_FOUND
SUPPLIER_CODE_ALREADY_EXISTS

WAREHOUSE_NOT_FOUND
WAREHOUSE_CODE_ALREADY_EXISTS

WAREHOUSE_LOCATION_NOT_FOUND
WAREHOUSE_LOCATION_CODE_ALREADY_EXISTS

INSPECTION_ITEM_NOT_FOUND
INSPECTION_ITEM_CODE_ALREADY_EXISTS
```

------------------------------------------------------------------------

# 67. React Query Key Convention

Customer:

``` ts
['customers']
['customers', id]
['customers', { search, page }]
```

Vehicle:

``` ts
['vehicles']
['vehicles', id]
```

Service:

``` ts
['services']
['services', id]
```

Part:

``` ts
['parts']
['parts', id]
```

Warehouse:

``` ts
['warehouses']
['warehouses', id]
```

Setelah mutation:

``` text
invalidate relevant query
```

------------------------------------------------------------------------

# 68. Frontend Feature Structure

``` text
frontend/src/features/
├── customers/
├── vehicles/
├── mechanics/
├── services/
├── parts/
├── suppliers/
├── warehouses/
└── inspection-items/
```

Contoh:

``` text
customers/
├── api/
│   └── customersApi.ts
├── components/
│   ├── CustomerForm.tsx
│   ├── CustomerTable.tsx
│   └── CustomerCombobox.tsx
├── hooks/
│   └── useCustomers.ts
├── pages/
│   ├── CustomerListPage.tsx
│   ├── CustomerCreatePage.tsx
│   └── CustomerDetailPage.tsx
├── schemas/
│   └── customer.schema.ts
└── types/
    └── customer.types.ts
```

------------------------------------------------------------------------

# 69. Route Map

``` text
/customers
/customers/new
/customers/:id
/customers/:id/edit

/vehicles
/vehicles/new
/vehicles/:id
/vehicles/:id/edit

/mechanics
/mechanics/new
/mechanics/:id/edit

/services
/services/new
/services/:id/edit

/service-categories

/parts
/parts/new
/parts/:id
/parts/:id/edit

/part-categories

/suppliers
/suppliers/new
/suppliers/:id/edit

/warehouses
/warehouses/new
/warehouses/:id/edit

/inspection-items
```

------------------------------------------------------------------------

# 70. Permission-to-Route

Customer:

``` text
customers.view → list/detail
customers.create → create
customers.update → edit
```

Vehicle:

``` text
vehicles.view
vehicles.create
vehicles.update
```

Service:

``` text
services.view
services.create
services.update
services.delete
```

Part:

``` text
parts.view
parts.create
parts.update
parts.delete
```

Supplier:

``` text
purchases.view
purchases.create
purchases.update
```

Warehouse:

``` text
inventory.view
inventory.adjust
```

------------------------------------------------------------------------

# 71. Data Import Readiness

V1 master data sebaiknya dirancang agar future CSV import mudah.

Import-ready fields:

``` text
Customer:
customer_code,name,phone,email,address,city

Vehicle:
plate_number,brand,model,variant,year,color,vin,engine_number

Service:
code,category_code,name,price,estimated_duration_minutes

Part:
sku,part_number,name,brand,unit,purchase_price,selling_price,
minimum_stock,maximum_stock,barcode,category_code
```

Import belum wajib di Phase 11 kecuali diprioritaskan.

------------------------------------------------------------------------

# 72. Bulk Import Future Contract

Future endpoint:

``` text
POST /api/v1/import/customers
POST /api/v1/import/vehicles
POST /api/v1/import/services
POST /api/v1/import/parts
```

Response:

``` text
total
success
failed
errors[]
```

Jangan membuat import besar synchronous jika data sangat besar.

------------------------------------------------------------------------

# 73. Master Data Dashboard Cards

Optional summary:

``` text
Total Customers
Active Vehicles
Active Mechanics
Active Services
Active Parts
Low Stock Parts
Active Suppliers
Warehouses
```

Dashboard tetap menggunakan query service khusus, bukan mengambil
seluruh tabel.

------------------------------------------------------------------------

# 74. Master Data Audit

Audit minimal:

``` text
CUSTOMER_CREATED
CUSTOMER_UPDATED
CUSTOMER_DEACTIVATED

VEHICLE_CREATED
VEHICLE_UPDATED

MECHANIC_CREATED
MECHANIC_UPDATED
MECHANIC_DEACTIVATED

SERVICE_CREATED
SERVICE_UPDATED
SERVICE_DEACTIVATED

PART_CREATED
PART_UPDATED
PART_DEACTIVATED

SUPPLIER_CREATED
SUPPLIER_UPDATED

WAREHOUSE_CREATED
WAREHOUSE_UPDATED

INSPECTION_ITEM_CREATED
INSPECTION_ITEM_UPDATED
```

Audit harus menyimpan actor.

------------------------------------------------------------------------

# 75. Master Data Service Rules

Service layer bertanggung jawab:

``` text
existence check
active check
duplicate check
reference protection
normalization
business validation
audit
transaction boundary
```

Repository hanya:

``` text
CRUD/query database
```

------------------------------------------------------------------------

# 76. Transaction Boundary

Create simple master:

``` text
validation
→ insert
→ audit
```

Jika audit wajib atomic:

``` text
BEGIN
insert
audit
COMMIT
```

Update:

``` text
BEGIN
read old
update
audit old/new
COMMIT
```

Delete/deactivate:

``` text
BEGIN
validate references
deactivate
audit
COMMIT
```

------------------------------------------------------------------------

# 77. Concurrency

Untuk master data:

``` text
unique constraint
+
transaction when needed
```

Contoh dua user membuat SKU sama:

``` text
User A → SKU ABC
User B → SKU ABC

Database unique constraint
→ one succeeds
→ one receives duplicate error
```

------------------------------------------------------------------------

# 78. Master Data Acceptance

## Customer

-   [ ] CRUD works
-   [ ] search works
-   [ ] pagination works
-   [ ] validation works
-   [ ] duplicate code rejected
-   [ ] soft delete works

## Vehicle

-   [ ] customer relationship works
-   [ ] plate uniqueness works
-   [ ] search works
-   [ ] odometer validation works

## Mechanic

-   [ ] employee code unique
-   [ ] optional user relationship
-   [ ] inactive mechanic cannot be assigned to new WO

## Service

-   [ ] category relationship
-   [ ] price validation
-   [ ] historical price unaffected

## Part

-   [ ] SKU unique
-   [ ] barcode unique
-   [ ] category relationship
-   [ ] stock thresholds valid

## Supplier

-   [ ] supplier code unique
-   [ ] CRUD works

## Warehouse

-   [ ] code unique
-   [ ] main warehouse rule
-   [ ] location relationship

## Inspection

-   [ ] code unique
-   [ ] sequence sorting
-   [ ] active filtering

------------------------------------------------------------------------

# 79. Integration Acceptance

Master Data phase dianggap complete jika:

``` text
Customer
   ↓
Vehicle
   ↓
WO selection
```

berhasil.

``` text
Service
   ↓
WO service selection
```

berhasil.

``` text
Part
   ↓
Warehouse
   ↓
WO part selection
```

berhasil.

``` text
Mechanic
   ↓
WO assignment
```

berhasil.

``` text
Supplier
   ↓
Purchase
```

berhasil.

------------------------------------------------------------------------

# 80. Definition of Done

``` text
DATABASE
  ↓
MODELS
  ↓
REPOSITORIES
  ↓
SERVICES
  ↓
VALIDATORS
  ↓
CONTROLLERS
  ↓
ROUTES
  ↓
PERMISSIONS
  ↓
API TESTS
  ↓
FRONTEND PAGES
  ↓
FORMS
  ↓
TABLES
  ↓
SEARCH/FILTER
  ↓
RESPONSIVE UX
  ↓
INTEGRATION
```

Phase 11 tidak dianggap selesai hanya karena CRUD backend tersedia.

Frontend dan permission harus terintegrasi.

------------------------------------------------------------------------

# 81. Claude Code Master Prompt --- MASTER DATA PHASE

``` text
You are the senior full-stack engineer implementing Phase 11 of GARAGE PRO.

PROJECT:
GARAGE PRO — Workshop Management System V1.

REFERENCE:
Read and follow:
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

DO NOT redesign the existing architecture.

STACK:
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
- Lucide

Backend:
- Node.js
- Express
- TypeScript
- Sequelize
- MySQL 8+
- Zod
- bcrypt/Argon2 where applicable

IMPLEMENT THESE MASTER DATA MODULES:

1. Customer
2. Vehicle
3. Mechanic
4. Service Category
5. Service
6. Part Category
7. Spare Part
8. Supplier
9. Warehouse
10. Warehouse Location
11. Inspection Item

BACKEND REQUIREMENTS:

For each module implement:
- model
- repository
- service
- validator
- controller
- routes
- tests

Use:
Controller → Service → Repository → Sequelize.

Do not query Sequelize directly inside controllers.

AUTHORIZATION:

Protect every endpoint using:
authenticate
authorize(permission)

Use the permission catalog from AUTH_RBAC.

CUSTOMER:
Implement:
- list
- create
- detail
- update
- soft delete
- search
- pagination

Customer code must be generated server-side.

VEHICLE:
Implement:
- customer relationship
- plate normalization
- unique plate protection
- search
- pagination
- update
- soft delete

MECHANIC:
Implement:
- employee code unique
- optional user relationship
- active/inactive
- prevent inactive mechanic from new assignment

SERVICE:
Implement:
- category relationship
- price
- duration
- active/inactive
- search
- filters

PART:
Implement:
- category
- SKU
- part number
- barcode
- purchase price
- selling price
- stock thresholds
- active/inactive
- search
- filters

SUPPLIER:
Implement CRUD and purchasing relationship.

WAREHOUSE:
Implement CRUD.
Support main warehouse rule.
Implement warehouse locations.

INSPECTION:
Implement CRUD.
Support category and sequence.

GLOBAL RULES:

- backend validation mandatory
- frontend validation mandatory for UX
- soft delete for master data
- never hard-delete records referenced by transactions
- inactive master data cannot be used in new transactions
- historical transactions remain unchanged
- database unique constraints remain the final duplicate protection
- audit important mutations

API:

Implement REST endpoints under:
`/api/v1`

Support:
- pagination
- search
- filtering
- sorting using whitelist
- consistent response format
- consistent error format

FRONTEND:

Implement feature-oriented structure.

Each module should have:
- list page
- form
- detail where appropriate
- table/card responsive presentation
- loading state
- empty state
- error state
- validation
- toast feedback
- confirmation dialog for destructive actions

MOBILE:
Mechanic and operational screens must remain usable on mobile.
Minimum touch target:
44px.

DESKTOP:
Admin/master-data management should use efficient tables and filters.

COMBOBOX:
Implement reusable search combobox for:
- customer
- vehicle
- mechanic
- service
- part
- supplier
- warehouse
- category

Support quick create where specified.

QUERY INVALIDATION:

After create/update/delete:
invalidate appropriate TanStack Query keys.

TESTS:

Backend:
- create
- validation
- duplicate
- update
- soft delete
- not found
- permission denied
- inactive master
- reference protection

Frontend:
- form validation
- list rendering
- search
- mutation
- permission visibility

INTEGRATION:

Verify:

Customer → Vehicle
Vehicle → Work Order selector
Mechanic → Work Order assignment
Service → Work Order service selector
Part → Work Order part selector
Warehouse → Part inventory selector
Supplier → Purchase selector
Inspection Item → Inspection selector

Do not implement Work Order business logic in this phase beyond the selectors/contracts necessary for integration.

SECURITY:

Never trust frontend.
Never expose sensitive fields.
Never bypass authorization.
Do not use raw SQL unless existing architecture requires it.
Use parameterized queries through Sequelize.

PROCESS:

1. Inspect current repository.
2. Inspect existing migrations/models.
3. Inspect auth/RBAC implementation.
4. Identify reusable components.
5. Implement backend modules.
6. Implement API routes.
7. Implement tests.
8. Implement frontend modules.
9. Integrate permissions.
10. Run typecheck.
11. Run backend tests.
12. Run frontend build.
13. Run integration tests.

Do not stop at planning.
Actually modify the repository.

FINAL REPORT:

Return:
- files created
- files modified
- APIs implemented
- permissions used
- frontend screens implemented
- tests executed
- test results
- known limitations
- next recommended phase
```

------------------------------------------------------------------------

# 82. Phase 11 Output Structure

Target implementation:

``` text
backend/src/
├── features/
│   ├── customers/
│   ├── vehicles/
│   ├── mechanics/
│   ├── services/
│   ├── parts/
│   ├── suppliers/
│   ├── warehouses/
│   └── inspection-items/
│
frontend/src/
└── features/
    ├── customers/
    ├── vehicles/
    ├── mechanics/
    ├── services/
    ├── parts/
    ├── suppliers/
    ├── warehouses/
    └── inspection-items/
```

------------------------------------------------------------------------

# 83. Next Phase

Setelah Master Data selesai:

**GARAGE_PRO_WORK_ORDER_ENGINE_V1.md**

Core flow:

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
Service Recommendation
   ↓
Spare Part Recommendation
   ↓
Estimate
   ↓
Customer Approval
   ↓
IN_PROGRESS
   ↓
QC
   ↓
READY
   ↓
INVOICE
   ↓
PAYMENT
```

Work Order adalah core transaction engine GARAGE PRO dan menjadi titik
integrasi antara Master Data, Inventory, Invoice dan Payment.
