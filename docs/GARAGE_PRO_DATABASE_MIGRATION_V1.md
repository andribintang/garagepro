# GARAGE PRO --- DATABASE MIGRATION + SEED DATA V1

**Project:** GARAGE PRO --- Workshop Management System\
**Document:** Database Migration + Seed Data Specification\
**Version:** V1.0\
**Status:** Implementation Ready\
**Database:** MySQL 8+\
**ORM:** Sequelize 6 + sequelize-cli\
**Backend:** Node.js + Express + TypeScript

------------------------------------------------------------------------

## 1. Tujuan

Dokumen ini menerjemahkan Database Design V1 menjadi implementasi
database yang konkret:

-   urutan migration yang aman berdasarkan foreign key;
-   struktur tabel dan kolom;
-   primary key, foreign key, unique key dan index;
-   soft delete dan timestamps;
-   precision DECIMAL untuk uang dan kuantitas;
-   status/status workflow;
-   inventory ledger;
-   invoice dan payment;
-   audit log;
-   seed roles, permissions, settings dan master data;
-   development admin;
-   test database/reset;
-   rollback;
-   acceptance criteria;
-   prompt Claude Code untuk mengimplementasikan fase migration.

Prinsip utama:

> **Migration membentuk struktur database. Seeder mengisi baseline data.
> Business logic tetap berada di service layer, bukan di database
> trigger, kecuali kebutuhan integritas yang benar-benar wajib.**

------------------------------------------------------------------------

# 2. Arsitektur Database

Database utama:

``` text
garage_pro
```

Storage engine:

``` text
InnoDB
```

Character set:

``` text
utf8mb4
```

Collation:

``` text
utf8mb4_unicode_ci
```

Semua tabel transaksi menggunakan InnoDB agar mendukung transaction dan
row locking.

------------------------------------------------------------------------

# 3. Konvensi Database

## 3.1 Primary Key

Gunakan:

``` sql
BIGINT UNSIGNED AUTO_INCREMENT
```

Contoh:

``` sql
id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY
```

Untuk Sequelize:

``` ts
id: {
  type: DataTypes.BIGINT.UNSIGNED,
  autoIncrement: true,
  primaryKey: true,
}
```

## 3.2 Timestamps

Tabel aplikasi menggunakan:

``` text
created_at
updated_at
```

Untuk master data yang mendukung soft delete:

``` text
deleted_at
```

Sequelize:

``` ts
{
  timestamps: true,
  paranoid: true,
  underscored: true,
}
```

Jangan menggunakan `paranoid` pada tabel ledger/transaksi yang harus
immutable.

------------------------------------------------------------------------

# 4. Money dan Quantity

## 4.1 Money

Gunakan:

``` sql
DECIMAL(15,2)
```

Contoh:

``` text
selling_price
purchase_price
subtotal
discount
tax
grand_total
paid_amount
```

Jangan gunakan FLOAT/DOUBLE untuk nilai uang.

## 4.2 Quantity

Gunakan:

``` sql
DECIMAL(15,3)
```

Agar mendukung unit/part quantity yang membutuhkan pecahan.

------------------------------------------------------------------------

# 5. Strategi Migration

Migration harus dijalankan dalam urutan dependency.

Urutan:

``` text
001 users/roles foundation
002 permissions
003 role_permissions
004 users
005 customers
006 vehicles
007 mechanics
008 service_categories
009 services
010 part_categories
011 spare_parts
012 warehouses
013 warehouse_locations
014 suppliers
015 inspection_items
016 work_orders
017 work_order_inspections
018 work_order_services
019 work_order_parts
020 work_order_recommendations
021 stock_movements
022 stock_opnames
023 stock_opname_items
024 purchases
025 purchase_items
026 invoices
027 payments
028 settings
029 audit_logs
```

Catatan:

`users` membutuhkan `roles`, sedangkan `role_permissions` membutuhkan
`roles` dan `permissions`.

Jika implementasi memerlukan `users.created_by`, `audit_logs.user_id`,
atau self-reference lain, FK harus dibuat setelah tabel target tersedia
atau menggunakan migration terpisah.

------------------------------------------------------------------------

# 6. Migration Filename Convention

Gunakan timestamp Sequelize:

``` text
YYYYMMDDHHMMSS-create-roles.ts
YYYYMMDDHHMMSS-create-permissions.ts
...
```

Dalam dokumentasi phase implementation, logical order tetap menggunakan:

``` text
001
002
003
...
```

Contoh:

``` text
20260914000100-create-roles.ts
20260914000200-create-permissions.ts
20260914000300-create-role-permissions.ts
```

Jangan menggunakan filename yang dapat menghasilkan urutan berbeda antar
environment.

------------------------------------------------------------------------

# 7. Tabel AUTH

## 7.1 roles

Kolom:

  Field         Type              Rule
  ------------- ----------------- ---------------
  id            BIGINT UNSIGNED   PK
  code          VARCHAR(50)       UNIQUE
  name          VARCHAR(100)      NOT NULL
  description   VARCHAR(255)      NULL
  is_system     BOOLEAN           DEFAULT false
  created_at    DATETIME          NOT NULL
  updated_at    DATETIME          NOT NULL

Index:

``` text
UNIQUE(code)
```

Seed:

``` text
OWNER
ADMIN
MECHANIC
WAREHOUSE
```

------------------------------------------------------------------------

# 8. permissions

Kolom:

``` text
id
code
name
module
description
created_at
updated_at
```

Unique:

``` text
code
```

Format permission:

``` text
dashboard.view

customers.view
customers.create
customers.update
customers.delete

vehicles.view
vehicles.create
vehicles.update

mechanics.view
mechanics.create
mechanics.update

services.view
services.create
services.update
services.delete

parts.view
parts.create
parts.update
parts.delete

inventory.view
inventory.issue
inventory.return
inventory.adjust
inventory.opname

purchases.view
purchases.create
purchases.update
purchases.receive

work_orders.view
work_orders.create
work_orders.update
work_orders.approve
work_orders.start
work_orders.qc
work_orders.rework
work_orders.ready
work_orders.invoice
work_orders.complete
work_orders.cancel

invoices.view
invoices.create

payments.view
payments.create
payments.refund

reports.view

users.view
users.create
users.update
users.disable

roles.view
roles.update

settings.view
settings.update

audit_logs.view
```

Permission code harus stabil karena dipakai application authorization.

------------------------------------------------------------------------

# 9. role_permissions

Kolom:

``` text
id
role_id
permission_id
created_at
updated_at
```

Constraint:

``` text
FK role_id → roles.id
FK permission_id → permissions.id
UNIQUE(role_id, permission_id)
```

Index:

``` text
INDEX(role_id)
INDEX(permission_id)
```

------------------------------------------------------------------------

# 10. users

Kolom:

``` text
id
role_id
name
username
email
password_hash
phone
avatar_url
is_active
last_login_at
created_at
updated_at
deleted_at
```

Rules:

``` text
username UNIQUE
email UNIQUE
role_id NOT NULL
password_hash NOT NULL
is_active DEFAULT true
```

Index:

``` text
UNIQUE(username)
UNIQUE(email)
INDEX(role_id)
INDEX(is_active)
```

Password tidak boleh disimpan plaintext.

------------------------------------------------------------------------

# 11. CUSTOMERS

## 11.1 customers

Kolom:

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

Unique:

``` text
customer_code
```

Index:

``` text
INDEX(name)
INDEX(phone)
INDEX(email)
```

Nomor customer dibuat server-side.

Format contoh:

``` text
CUS-2026-000001
```

------------------------------------------------------------------------

# 12. vehicles

Kolom:

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

Rules:

``` text
customer_id NOT NULL
plate_number NOT NULL
brand NOT NULL
model NOT NULL
```

Unique yang direkomendasikan:

``` text
UNIQUE(plate_number)
```

Jika bisnis membutuhkan kemungkinan kendaraan berpindah owner, jangan
menggunakan customer_id sebagai bagian unique plate.

Index:

``` text
INDEX(customer_id)
INDEX(plate_number)
INDEX(vin)
INDEX(engine_number)
```

------------------------------------------------------------------------

# 13. MECHANICS

## mechanics

Kolom:

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

Rules:

``` text
employee_code UNIQUE
user_id NULLABLE jika mekanik belum memiliki login
```

Index:

``` text
UNIQUE(employee_code)
INDEX(user_id)
INDEX(name)
INDEX(is_active)
```

------------------------------------------------------------------------

# 14. MASTER SERVICE

## 14.1 service_categories

Kolom:

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

Unique:

``` text
code
```

Index:

``` text
UNIQUE(code)
INDEX(name)
```

## 14.2 services

Kolom:

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

Rules:

``` text
price DECIMAL(15,2)
estimated_duration_minutes INT UNSIGNED
```

Index:

``` text
UNIQUE(code)
INDEX(category_id)
INDEX(name)
INDEX(is_active)
```

------------------------------------------------------------------------

# 15. MASTER SPARE PART

## 15.1 part_categories

Kolom:

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

## 15.2 spare_parts

Kolom:

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

Rules:

``` text
sku UNIQUE
part_number optional but indexed
barcode optional but indexed
minimum_stock >= 0
maximum_stock >= minimum_stock when supplied
purchase_price >= 0
selling_price >= 0
```

Index:

``` text
UNIQUE(sku)
INDEX(part_number)
INDEX(name)
INDEX(category_id)
INDEX(barcode)
INDEX(is_active)
```

------------------------------------------------------------------------

# 16. WAREHOUSE

## 16.1 warehouses

Kolom:

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

Unique:

``` text
code
```

## 16.2 warehouse_locations

Kolom:

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
UNIQUE(warehouse_id, code)
```

------------------------------------------------------------------------

# 17. WORK ORDER

## work_orders

Kolom:

``` text
id
wo_number
customer_id
vehicle_id
mechanic_id
status
complaint
diagnosis
estimated_total
approved_total
final_total
discount
tax
notes
opened_at
approved_at
started_at
qc_at
ready_at
completed_at
cancelled_at
created_by
updated_by
created_at
updated_at
```

Status canonical:

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
REJECTED
```

Recommended implementation: use `VARCHAR(30)` plus application constants
instead of MySQL ENUM, allowing safer future migration.

Unique:

``` text
wo_number
```

Index:

``` text
INDEX(customer_id)
INDEX(vehicle_id)
INDEX(mechanic_id)
INDEX(status)
INDEX(opened_at)
INDEX(created_at)
INDEX(status, created_at)
```

WO number example:

``` text
WO-2026-000001
```

Totals are server-calculated.

------------------------------------------------------------------------

# 18. work_order_inspections

Kolom:

``` text
id
work_order_id
inspection_item_id
result
note
severity
created_by
created_at
updated_at
```

Unique:

``` text
UNIQUE(work_order_id, inspection_item_id)
```

Result example:

``` text
GOOD
CHECK
REPLACE
NOT_APPLICABLE
```

------------------------------------------------------------------------

# 19. inspection_items

Kolom:

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

Unique:

``` text
code
```

Index:

``` text
INDEX(category)
INDEX(sequence)
```

Seed examples:

``` text
Engine Oil
Brake Front
Brake Rear
Tire Front
Tire Rear
Battery
Chain
Sprocket
Lights
Horn
Coolant
Drive Belt
Air Filter
Spark Plug
Suspension
```

------------------------------------------------------------------------

# 20. work_order_services

Kolom:

``` text
id
work_order_id
service_id
service_code_snapshot
service_name_snapshot
qty
unit_price
discount
subtotal
mechanic_id
notes
created_at
updated_at
```

Rules:

``` text
qty > 0
unit_price >= 0
discount >= 0
subtotal = server calculated
```

Historical snapshot fields are mandatory so changes to master service
data do not alter old transactions.

Index:

``` text
INDEX(work_order_id)
INDEX(service_id)
INDEX(mechanic_id)
```

------------------------------------------------------------------------

# 21. work_order_parts

Kolom:

``` text
id
work_order_id
spare_part_id
warehouse_id
sku_snapshot
part_number_snapshot
part_name_snapshot
qty
issued_qty
returned_qty
unit_cost
unit_price
discount
subtotal
issue_status
issued_at
returned_at
created_at
updated_at
```

Issue status:

``` text
PENDING
PARTIAL
ISSUED
RETURNED
```

Rules:

``` text
qty > 0
issued_qty >= 0
returned_qty >= 0
issued_qty <= qty
returned_qty <= issued_qty
```

Index:

``` text
INDEX(work_order_id)
INDEX(spare_part_id)
INDEX(warehouse_id)
INDEX(issue_status)
```

------------------------------------------------------------------------

# 22. work_order_recommendations

Kolom:

``` text
id
work_order_id
type
description
service_id
spare_part_id
estimated_price
status
approved_at
approved_by
declined_at
declined_by
converted_at
created_at
updated_at
```

Type:

``` text
SERVICE
PART
GENERAL
```

Status:

``` text
PENDING
APPROVED
DECLINED
CONVERTED
```

Index:

``` text
INDEX(work_order_id)
INDEX(status)
```

------------------------------------------------------------------------

# 23. INVENTORY LEDGER

## stock_movements

Ini adalah sumber kebenaran inventory.

Kolom:

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

Movement type:

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

Reference type:

``` text
PURCHASE
WORK_ORDER
STOCK_OPNAME
ADJUSTMENT
TRANSFER
```

Quantity rule:

``` text
positive quantity = stock in
negative quantity = stock out
```

Jangan update ledger row yang sudah dibuat.

Index:

``` text
INDEX(spare_part_id, warehouse_id, created_at)
INDEX(reference_type, reference_id)
INDEX(movement_type)
INDEX(created_at)
```

Penting:

> `balance_after` adalah audit convenience, sedangkan stock
> authoritative dihitung dari ledger atau maintained stock projection
> yang selalu dapat direkonsiliasi.

------------------------------------------------------------------------

# 24. STOCK OPNAME

## stock_opnames

Kolom:

``` text
id
opname_number
warehouse_id
status
counted_at
completed_at
created_by
completed_by
notes
created_at
updated_at
```

Status:

``` text
DRAFT
COUNTING
REVIEW
COMPLETED
CANCELLED
```

Unique:

``` text
opname_number
```

## stock_opname_items

Kolom:

``` text
id
stock_opname_id
spare_part_id
system_qty
counted_qty
difference_qty
unit_cost
notes
created_at
updated_at
```

Rules:

``` text
difference_qty = counted_qty - system_qty
```

Final adjustment harus menghasilkan stock movement.

------------------------------------------------------------------------

# 25. SUPPLIER

## suppliers

Kolom:

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

Unique:

``` text
supplier_code
```

Index:

``` text
INDEX(name)
INDEX(phone)
INDEX(email)
```

------------------------------------------------------------------------

# 26. PURCHASING

## purchases

Kolom:

``` text
id
purchase_number
supplier_id
warehouse_id
status
purchase_date
subtotal
discount
tax
grand_total
notes
created_by
received_by
received_at
created_at
updated_at
```

Status:

``` text
DRAFT
ORDERED
PARTIAL_RECEIVED
RECEIVED
CANCELLED
```

Unique:

``` text
purchase_number
```

## purchase_items

Kolom:

``` text
id
purchase_id
spare_part_id
qty
received_qty
unit_cost
discount
subtotal
created_at
updated_at
```

Rules:

``` text
received_qty <= qty
```

Receiving creates `PURCHASE_RECEIPT` stock movements.

------------------------------------------------------------------------

# 27. INVOICE

## invoices

Kolom:

``` text
id
invoice_number
work_order_id
customer_id
invoice_date
status
subtotal
discount
tax
grand_total
paid_amount
balance_due
notes
created_by
created_at
updated_at
```

Status:

``` text
DRAFT
ISSUED
PARTIAL
PAID
VOID
```

Unique:

``` text
invoice_number
work_order_id
```

Index:

``` text
INDEX(customer_id)
INDEX(invoice_date)
INDEX(status)
```

Invoice values are snapshots.

------------------------------------------------------------------------

# 28. PAYMENTS

## payments

Kolom:

``` text
id
payment_number
invoice_id
payment_date
method
amount
reference_number
status
notes
received_by
created_at
updated_at
```

Payment method:

``` text
CASH
TRANSFER
QRIS
DEBIT_CARD
CREDIT_CARD
OTHER
```

Status:

``` text
SUCCESS
VOID
REFUNDED
```

Unique:

``` text
payment_number
```

Index:

``` text
INDEX(invoice_id)
INDEX(payment_date)
INDEX(method)
INDEX(status)
INDEX(reference_number)
```

Payment amount harus positif.

Server menghitung:

``` text
paid_amount = SUM(successful payments)
balance_due = grand_total - paid_amount
```

------------------------------------------------------------------------

# 29. SETTINGS

## settings

Kolom:

``` text
id
key
value
type
description
is_public
created_at
updated_at
```

Unique:

``` text
key
```

Type:

``` text
STRING
NUMBER
BOOLEAN
JSON
```

Seed default:

``` text
workshop.name
workshop.address
workshop.phone
workshop.email
workshop.logo_url

currency.code = IDR
currency.decimal_places = 0

wo.number_prefix = WO
invoice.number_prefix = INV
customer.number_prefix = CUS
purchase.number_prefix = PO
payment.number_prefix = PAY
opname.number_prefix = OP

tax.enabled = false
tax.rate = 0

inventory.allow_negative_stock = false

work_order.require_approval = true
work_order.auto_invoice_on_ready = false
```

------------------------------------------------------------------------

# 30. AUDIT LOG

## audit_logs

Kolom:

``` text
id
user_id
action
module
entity_type
entity_id
old_values
new_values
ip_address
user_agent
request_id
created_at
```

`old_values` dan `new_values`:

``` sql
JSON
```

Index:

``` text
INDEX(user_id)
INDEX(module)
INDEX(entity_type, entity_id)
INDEX(action)
INDEX(created_at)
INDEX(request_id)
```

Audit log bersifat append-only.

Jangan memberikan endpoint untuk mengubah audit log.

------------------------------------------------------------------------

# 31. FOREIGN KEY POLICY

Default:

``` text
ON UPDATE CASCADE
```

Untuk transaksi yang memiliki histori penting, gunakan:

``` text
ON DELETE RESTRICT
```

Contoh:

``` text
customers → vehicles
customers → work_orders
vehicles → work_orders
work_orders → invoices
invoices → payments
spare_parts → stock_movements
warehouses → stock_movements
```

Master data sebaiknya menggunakan soft delete sehingga FK tidak perlu
menghapus histori.

Jangan menggunakan:

``` text
ON DELETE CASCADE
```

pada transaksi utama secara sembarangan.

------------------------------------------------------------------------

# 32. Sequelize Model Defaults

Base model convention:

``` ts
export const modelDefaults = {
  timestamps: true,
  underscored: true,
  freezeTableName: true,
};
```

Master model:

``` ts
{
  ...modelDefaults,
  paranoid: true,
}
```

Transaction/ledger model:

``` ts
{
  ...modelDefaults,
  paranoid: false,
}
```

------------------------------------------------------------------------

# 33. Contoh Migration Sequelize

Contoh:

``` ts
import { QueryInterface, DataTypes } from 'sequelize';

export async function up(queryInterface: QueryInterface) {
  await queryInterface.createTable('roles', {
    id: {
      type: DataTypes.BIGINT.UNSIGNED,
      autoIncrement: true,
      primaryKey: true,
      allowNull: false,
    },

    code: {
      type: DataTypes.STRING(50),
      allowNull: false,
      unique: true,
    },

    name: {
      type: DataTypes.STRING(100),
      allowNull: false,
    },

    description: {
      type: DataTypes.STRING(255),
      allowNull: true,
    },

    is_system: {
      type: DataTypes.BOOLEAN,
      allowNull: false,
      defaultValue: false,
    },

    created_at: {
      type: DataTypes.DATE,
      allowNull: false,
    },

    updated_at: {
      type: DataTypes.DATE,
      allowNull: false,
    },
  });
}

export async function down(queryInterface: QueryInterface) {
  await queryInterface.dropTable('roles');
}
```

Migration harus deterministic dan dapat dijalankan dari database kosong.

------------------------------------------------------------------------

# 34. Index Strategy

Prioritas index:

### Search

``` text
customers.name
customers.phone
vehicles.plate_number
spare_parts.sku
spare_parts.part_number
spare_parts.barcode
services.name
```

### Relationship

Semua FK harus mempunyai index yang sesuai.

### Workflow

``` text
work_orders.status
work_orders.mechanic_id
work_orders.created_at
```

### Reports

``` text
invoices.invoice_date
invoices.status
payments.payment_date
stock_movements.created_at
stock_movements.movement_type
```

Jangan membuat index berlebihan.

Setiap index harus memiliki alasan query.

------------------------------------------------------------------------

# 35. Seed Data Strategy

Seeder dibagi menjadi:

``` text
01-roles
02-permissions
03-role-permissions
04-settings
05-inspection-items
06-service-categories
07-services
08-part-categories
09-warehouses
10-warehouse-locations
11-admin
```

Seeder harus idempotent.

Artinya menjalankan seed dua kali tidak boleh membuat duplicate baseline
records.

Gunakan:

``` ts
findOrCreate
```

atau upsert sesuai kebutuhan.

------------------------------------------------------------------------

# 36. Role Seed

## OWNER

Full access.

## ADMIN

Operational access:

``` text
dashboard
customers
vehicles
mechanics
services
parts
inventory view
work orders
invoice
payment
reports
```

## MECHANIC

Access:

``` text
dashboard limited
work_orders.view
work_orders.create
work_orders.update
work_orders.start
work_orders.qc
work_orders.rework
work_orders.ready
```

Tidak boleh:

``` text
payment
user management
role management
inventory adjustment
financial configuration
```

## WAREHOUSE

Access:

``` text
parts.view
inventory.view
inventory.issue
inventory.return
inventory.adjust
inventory.opname
purchases.view
purchases.receive
```

------------------------------------------------------------------------

# 37. Default Admin Seed

Development only.

Environment:

``` text
NODE_ENV=development
```

Seed admin menggunakan environment variable:

``` text
SEED_ADMIN_NAME
SEED_ADMIN_USERNAME
SEED_ADMIN_EMAIL
SEED_ADMIN_PASSWORD
```

Jangan hard-code production password.

Contoh `.env.example`:

``` env
SEED_ADMIN_NAME=Garage Pro Admin
SEED_ADMIN_USERNAME=admin
SEED_ADMIN_EMAIL=admin@example.local
SEED_ADMIN_PASSWORD=ChangeMeImmediately
```

Production:

> Admin pertama harus dibuat melalui secure deployment/bootstrap process
> dan password wajib diganti.

Password harus di-hash dengan bcrypt atau Argon2 sebelum disimpan.

------------------------------------------------------------------------

# 38. Seed Master Service Categories

Contoh:

``` text
SERVICE_GENERAL
SERVICE_ENGINE
SERVICE_BRAKE
SERVICE_ELECTRICAL
SERVICE_TRANSMISSION
SERVICE_SUSPENSION
SERVICE_TUNEUP
```

Contoh service:

``` text
Oil Change
Periodic Service
Brake Inspection
Brake Pad Replacement
Battery Inspection
Chain Adjustment
Tire Replacement
Spark Plug Replacement
Air Filter Replacement
CVT Service
```

Harga seed hanya untuk development/demo.

Jangan menganggap seed price sebagai harga operasional final.

------------------------------------------------------------------------

# 39. Seed Part Categories

Contoh:

``` text
ENGINE
BRAKE
ELECTRICAL
TRANSMISSION
SUSPENSION
TIRE
OIL
FILTER
CHAIN
BATTERY
ACCESSORIES
```

------------------------------------------------------------------------

# 40. Seed Warehouse

Default:

``` text
MAIN
Workshop Main Warehouse
```

Location:

``` text
MAIN-RACK-A
MAIN-RACK-B
MAIN-RACK-C
```

------------------------------------------------------------------------

# 41. Test Database

Nama:

``` text
garage_pro_test
```

Environment:

``` env
DB_DATABASE=garage_pro_test
NODE_ENV=test
```

Test flow:

``` text
drop test database
create test database
run migrations
run test seed
execute tests
rollback/reset
```

Jangan pernah menjalankan test reset terhadap database production.

------------------------------------------------------------------------

# 42. Migration Commands

Install:

``` bash
npm install sequelize sequelize-cli mysql2
npm install -D ts-node
```

Migration:

``` bash
npx sequelize-cli db:migrate
```

Rollback terakhir:

``` bash
npx sequelize-cli db:migrate:undo
```

Rollback semua:

``` bash
npx sequelize-cli db:migrate:undo:all
```

Seeder:

``` bash
npx sequelize-cli db:seed:all
```

Undo seed:

``` bash
npx sequelize-cli db:seed:undo:all
```

Status:

``` bash
npx sequelize-cli db:migrate:status
```

------------------------------------------------------------------------

# 43. Recommended package scripts

`backend/package.json`:

``` json
{
  "scripts": {
    "db:migrate": "sequelize-cli db:migrate",
    "db:migrate:undo": "sequelize-cli db:migrate:undo",
    "db:migrate:reset": "sequelize-cli db:migrate:undo:all && sequelize-cli db:migrate",
    "db:seed": "sequelize-cli db:seed:all",
    "db:seed:undo": "sequelize-cli db:seed:undo:all",
    "db:reset": "sequelize-cli db:migrate:undo:all && sequelize-cli db:migrate && sequelize-cli db:seed:all",
    "db:status": "sequelize-cli db:migrate:status"
  }
}
```

------------------------------------------------------------------------

# 44. Environment Configuration

`.env.example`:

``` env
NODE_ENV=development

PORT=4000

DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=garage_pro
DB_USERNAME=root
DB_PASSWORD=

DB_LOGGING=false

JWT_SECRET=change-this-in-real-environment
JWT_EXPIRES_IN=8h

SEED_ADMIN_NAME=Garage Pro Admin
SEED_ADMIN_USERNAME=admin
SEED_ADMIN_EMAIL=admin@example.local
SEED_ADMIN_PASSWORD=ChangeMeImmediately
```

Production secret harus berasal dari secure secret
management/environment.

Jangan commit `.env`.

------------------------------------------------------------------------

# 45. Transaction Rules During Migration

Migration schema changes umumnya dijalankan satu migration per logical
change.

Untuk MySQL, DDL transaction behavior memiliki keterbatasan. Karena itu:

-   jangan mengasumsikan seluruh rangkaian DDL dapat di-rollback sebagai
    satu atomic transaction;
-   buat migration kecil dan deterministic;
-   pastikan `down()` benar-benar menghapus objek yang dibuat migration;
-   jangan mencampurkan data migration kompleks dengan schema migration
    jika tidak perlu;
-   gunakan backup sebelum migration production;
-   test `up → down → up` pada database kosong dan database clone.

------------------------------------------------------------------------

# 46. Production Migration Procedure

Urutan:

``` text
1. Backup database
2. Enable maintenance mode jika diperlukan
3. Verify application version
4. Verify migration status
5. Run migration
6. Verify schema
7. Run smoke test
8. Start/restart application
9. Monitor logs
10. Verify critical transactions
```

Critical smoke test:

``` text
login
create customer
create vehicle
create WO
add service
add part
issue part
create invoice
record payment
complete WO
```

------------------------------------------------------------------------

# 47. Rollback Strategy

Rollback schema hanya boleh dilakukan setelah:

-   memahami migration yang gagal;
-   memastikan data yang terdampak;
-   mengambil backup/snapshot;
-   menghentikan application write jika diperlukan.

Jangan melakukan:

``` bash
db:migrate:undo:all
```

pada production sebagai prosedur rutin.

Production rollback ideal:

``` text
fix-forward migration
```

Jika migration:

``` text
014-add-x
```

bermasalah, prefer membuat:

``` text
015-fix-x
```

daripada menghapus histori migration.

------------------------------------------------------------------------

# 48. Data Integrity Rules

Database harus memastikan:

``` text
quantity >= 0
price >= 0
discount >= 0
payment.amount > 0
minimum_stock >= 0
maximum_stock >= minimum_stock
```

Tetapi business workflow seperti:

``` text
NEW → CHECKING
CHECKING → ESTIMATE
APPROVED → IN_PROGRESS
QC → REWORK
```

harus divalidasi oleh service layer.

Jangan mengandalkan frontend.

------------------------------------------------------------------------

# 49. Inventory Integrity

Flow issue:

``` text
Work Order Part
      ↓
validate stock
      ↓
database transaction
      ↓
row lock / stock verification
      ↓
create stock movement
      ↓
update issue quantity
      ↓
commit
```

Flow return:

``` text
Return request
      ↓
validate previously issued qty
      ↓
transaction
      ↓
stock movement +
      ↓
update returned qty
      ↓
commit
```

Tidak boleh ada perubahan stock tanpa stock movement.

------------------------------------------------------------------------

# 50. Invoice Integrity

Saat invoice dibuat:

``` text
subtotal
- discount
+ tax
= grand_total
```

Semua dihitung backend.

Payment:

``` text
successful payments
= paid_amount
```

Balance:

``` text
grand_total - paid_amount
```

Status:

``` text
0 paid      → ISSUED
partial     → PARTIAL
fully paid  → PAID
```

Tidak boleh:

``` text
payment > grand_total
```

kecuali business rule secara eksplisit mendukung overpayment.

------------------------------------------------------------------------

# 51. Historical Data Integrity

Ketika master berubah:

``` text
Service price changes
Part selling price changes
Part name changes
SKU changes
```

data transaksi lama tidak boleh berubah.

Karena itu:

``` text
work_order_services.service_name_snapshot
work_order_services.unit_price

work_order_parts.sku_snapshot
work_order_parts.part_name_snapshot
work_order_parts.unit_price
work_order_parts.unit_cost
```

harus disimpan.

------------------------------------------------------------------------

# 52. Number Generator

Nomor dokumen tidak boleh dibuat hanya dari frontend.

Backend menggunakan service:

``` ts
documentNumberService.generate('WO');
documentNumberService.generate('INV');
documentNumberService.generate('PAY');
documentNumberService.generate('PO');
```

Untuk concurrency tinggi, gunakan strategy yang aman terhadap race
condition.

Minimal:

``` text
transaction
+
unique constraint
+
retry on duplicate
```

------------------------------------------------------------------------

# 53. Recommended Migration Folder

``` text
backend/
└── src/
    └── database/
        ├── migrations/
        │   ├── 20260914000100-create-roles.ts
        │   ├── 20260914000200-create-permissions.ts
        │   ├── 20260914000300-create-role-permissions.ts
        │   ├── 20260914000400-create-users.ts
        │   ├── ...
        │   └── 20260914002900-create-audit-logs.ts
        │
        ├── seeders/
        │   ├── 01-roles.ts
        │   ├── 02-permissions.ts
        │   ├── 03-role-permissions.ts
        │   ├── 04-settings.ts
        │   ├── 05-inspection-items.ts
        │   ├── 06-service-categories.ts
        │   ├── 07-services.ts
        │   ├── 08-part-categories.ts
        │   ├── 09-warehouses.ts
        │   ├── 10-warehouse-locations.ts
        │   └── 11-admin.ts
        │
        └── models/
```

------------------------------------------------------------------------

# 54. Database Config

Recommended:

``` ts
import { Sequelize } from 'sequelize';

export const sequelize = new Sequelize(
  process.env.DB_DATABASE!,
  process.env.DB_USERNAME!,
  process.env.DB_PASSWORD!,
  {
    host: process.env.DB_HOST,
    port: Number(process.env.DB_PORT || 3306),
    dialect: 'mysql',
    logging: process.env.DB_LOGGING === 'true' ? console.log : false,
    define: {
      underscored: true,
      freezeTableName: true,
      timestamps: true,
    },
    pool: {
      max: 10,
      min: 0,
      acquire: 30000,
      idle: 10000,
    },
  }
);
```

------------------------------------------------------------------------

# 55. Migration Acceptance Checklist

Migration phase dianggap selesai jika:

## Schema

-   [ ] Semua migration dapat dijalankan dari database kosong.
-   [ ] Semua FK valid.
-   [ ] Semua required unique constraint tersedia.
-   [ ] Semua critical index tersedia.
-   [ ] Semua money menggunakan DECIMAL.
-   [ ] Semua quantity menggunakan DECIMAL.
-   [ ] Semua timestamp konsisten.
-   [ ] Master data memakai soft delete.
-   [ ] Ledger transaksi immutable secara application policy.
-   [ ] Audit log append-only.

## Seeder

-   [ ] Roles tersedia.
-   [ ] Permissions tersedia.
-   [ ] Role-permissions tersedia.
-   [ ] Default settings tersedia.
-   [ ] Inspection items tersedia.
-   [ ] Service categories tersedia.
-   [ ] Services demo tersedia.
-   [ ] Part categories tersedia.
-   [ ] Main warehouse tersedia.
-   [ ] Admin development tersedia.

## Safety

-   [ ] Production password tidak hard-coded.
-   [ ] `.env` tidak masuk Git.
-   [ ] Test DB terpisah.
-   [ ] Migration status dapat diverifikasi.
-   [ ] Rollback telah diuji.
-   [ ] Backup procedure terdokumentasi.

## Integration

-   [ ] Backend dapat connect MySQL.
-   [ ] `/health` database check berhasil.
-   [ ] Model dapat query.
-   [ ] Authentication dapat menemukan admin seed.
-   [ ] Work Order dapat menggunakan customer, vehicle dan mechanic.
-   [ ] Inventory dapat menggunakan warehouse dan spare part.

------------------------------------------------------------------------

# 56. Migration Test Matrix

  Test                         Expected
  ---------------------------- ----------------------------
  Fresh DB → migrate           PASS
  Fresh DB → seed              PASS
  migrate → undo last          PASS
  migrate → undo all           PASS
  migrate twice                no duplicate schema
  seed twice                   no duplicate baseline data
  invalid FK                   rejected
  duplicate SKU                rejected
  duplicate WO number          rejected
  duplicate invoice number     rejected
  negative price               rejected
  invalid payment              rejected
  stock issue without stock    rejected by service
  payment after void invoice   rejected by service

------------------------------------------------------------------------

# 57. Claude Code Master Prompt --- DATABASE MIGRATION PHASE

Gunakan prompt berikut pada Claude Code setelah project scaffold
tersedia.

``` text
You are the senior backend/database engineer implementing GARAGE PRO — Workshop Management System V1.

PROJECT:
GARAGE PRO is a motorcycle general workshop management system.

STACK:
- Node.js
- Express
- TypeScript
- Sequelize 6
- MySQL 8+
- sequelize-cli

OBJECTIVE:
Implement the complete database migration and seed-data layer according to:
GARAGE_PRO_DATABASE_DESIGN_V1.md
GARAGE_PRO_BUSINESS_RULES_V1.md
GARAGE_PRO_PROJECT_ARCHITECTURE_V1.md
GARAGE_PRO_PROJECT_SCAFFOLD_V1.md
GARAGE_PRO_API_SPECIFICATION_V1.md

DO NOT redesign the architecture.
DO NOT invent unrelated modules.
Follow the existing project conventions.

DATABASE REQUIREMENTS:

1. MySQL 8+
2. InnoDB
3. utf8mb4
4. BIGINT UNSIGNED AUTO_INCREMENT primary keys
5. DECIMAL(15,2) for money
6. DECIMAL(15,3) for quantities
7. underscored table/column naming
8. timestamps:
   created_at
   updated_at
9. soft delete for master entities using deleted_at
10. transaction and ledger records must not use destructive cascade
11. use foreign keys
12. use unique constraints for business document numbers
13. add indexes based on expected queries
14. use VARCHAR for workflow status rather than MySQL ENUM unless an existing project convention requires ENUM
15. never use FLOAT/DOUBLE for money

IMPLEMENT TABLES IN DEPENDENCY ORDER:

roles
permissions
role_permissions
users

customers
vehicles
mechanics

service_categories
services

part_categories
spare_parts

warehouses
warehouse_locations

suppliers

inspection_items
work_orders
work_order_inspections
work_order_services
work_order_parts
work_order_recommendations

stock_movements
stock_opnames
stock_opname_items

purchases
purchase_items

invoices
payments

settings
audit_logs

REQUIRED BUSINESS NUMBER FIELDS:

customers.customer_code
work_orders.wo_number
suppliers.supplier_code
purchases.purchase_number
invoices.invoice_number
payments.payment_number
stock_opnames.opname_number

All must have unique constraints.

WORK ORDER STATUSES:

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
REJECTED

Do not enforce workflow transitions inside migrations.
Workflow transitions belong in service layer.

INVENTORY:

stock_movements is the immutable inventory ledger.

Movement types:

PURCHASE_RECEIPT
WORK_ORDER_ISSUE
WORK_ORDER_RETURN
ADJUSTMENT_IN
ADJUSTMENT_OUT
STOCK_OPNAME_IN
STOCK_OPNAME_OUT
TRANSFER_IN
TRANSFER_OUT

Do not implement a mutable stock history table that bypasses stock_movements.

TRANSACTIONS:

Work order service and part records must store historical snapshots of important transaction data.

At minimum:

work_order_services:
service_code_snapshot
service_name_snapshot
unit_price
discount
subtotal

work_order_parts:
sku_snapshot
part_number_snapshot
part_name_snapshot
unit_cost
unit_price
discount
subtotal

INVOICES:

Store invoice totals as transaction snapshots.

PAYMENTS:

Store payment records separately.
Payment total must be calculated by service layer.

AUDIT:

audit_logs must be append-only.
Use JSON fields for old_values and new_values.

SEED DATA:

Create idempotent seeders for:

1. OWNER
2. ADMIN
3. MECHANIC
4. WAREHOUSE

Permissions covering:
dashboard
customers
vehicles
mechanics
services
parts
inventory
purchases
work_orders
invoices
payments
reports
users
roles
settings
audit_logs

Create default settings.

Create inspection item baseline.

Create service categories and demo services.

Create part categories.

Create MAIN warehouse and sample locations.

Create development admin using environment variables.

NEVER hard-code a production password.

Use bcrypt or Argon2 for password hashing.

ENVIRONMENT:

Create/update .env.example with:

NODE_ENV
PORT
DB_HOST
DB_PORT
DB_DATABASE
DB_USERNAME
DB_PASSWORD
DB_LOGGING
JWT_SECRET
JWT_EXPIRES_IN
SEED_ADMIN_NAME
SEED_ADMIN_USERNAME
SEED_ADMIN_EMAIL
SEED_ADMIN_PASSWORD

TEST DATABASE:

Support:
garage_pro_test

Create scripts:

db:migrate
db:migrate:undo
db:migrate:reset
db:seed
db:seed:undo
db:reset
db:status

QUALITY REQUIREMENTS:

- TypeScript strict mode.
- No any unless technically unavoidable.
- Migration files must compile.
- Seeders must be deterministic/idempotent.
- FK relationships must be correct.
- down() must remove objects created by up().
- Do not modify unrelated application modules.
- Do not put business logic in migrations.
- Do not create database triggers unless explicitly required.
- Do not silently swallow migration errors.
- Use clear naming.
- Add comments only where useful.

VALIDATION PROCESS:

After implementation:

1. Create empty database.
2. Run migrations.
3. Verify all tables.
4. Verify indexes.
5. Verify foreign keys.
6. Run seeders.
7. Run seeders again to prove idempotency.
8. Run migration status.
9. Test rollback.
10. Re-run migrations.
11. Run TypeScript build.
12. Run tests.

Provide a final implementation report containing:

- files created
- files modified
- migration order
- seed data
- commands executed
- test results
- known limitations
- next recommended phase

Do not stop after generating migration files.
Actually inspect the project structure and integrate with the existing scaffold.
```

------------------------------------------------------------------------

# 58. Phase Completion Gate

Database Migration V1 boleh dinyatakan **DONE** jika:

``` text
SCHEMA
  ↓
MIGRATION
  ↓
SEED
  ↓
ROLLBACK TEST
  ↓
TYPECHECK
  ↓
DATABASE INTEGRITY TEST
  ↓
ACCEPTANCE
```

Setelah phase ini selesai, roadmap berikutnya adalah:

``` text
09 ✅ DATABASE MIGRATION + SEED DATA
10 🔜 AUTHENTICATION + RBAC
11    MASTER DATA
12    WORK ORDER ENGINE
13    INVENTORY ENGINE
14    INVOICE + PAYMENT
15    FRONTEND INTEGRATION
16    TESTING
17    DEPLOYMENT
```

------------------------------------------------------------------------

# 59. Next Phase

Setelah database migration stabil, implementasi berikutnya:

**GARAGE_PRO_AUTH_RBAC_V1.md**

Cakupan:

``` text
Login
↓
Password Hash
↓
JWT
↓
Refresh Session
↓
Current User
↓
Role
↓
Permission
↓
Route Guard
↓
API Authorization
↓
Frontend Permission Guard
↓
Audit Login
↓
Logout
```

Auth/RBAC harus selesai sebelum modul transaksi utama agar seluruh
endpoint sejak awal memiliki authorization boundary yang benar.
