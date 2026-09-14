# GARAGE PRO --- DATABASE DESIGN & ERD V1

**Document Type:** Database Design Specification\
**Product:** GARAGE PRO --- Workshop Management System\
**Version:** V1.0\
**Database:** MySQL 8+\
**ORM:** Sequelize\
**Status:** Development Ready\
**Timezone:** Asia/Jakarta\
**Date:** 14 September 2026

------------------------------------------------------------------------

# 1. Purpose

Dokumen ini mendefinisikan struktur database GARAGE PRO V1 sebagai
single source of truth untuk:

-   Database migration
-   Sequelize models
-   Foreign key relationships
-   API development
-   Inventory engine
-   Work Order engine
-   Invoice & payment engine
-   Reporting
-   Audit trail

Database harus menjaga integritas transaksi, histori harga, histori
kendaraan, dan histori stok.

------------------------------------------------------------------------

# 2. Database Principles

1.  Gunakan MySQL 8+.
2.  Gunakan InnoDB.
3.  Gunakan UTF8MB4.
4.  Gunakan `BIGINT UNSIGNED` untuk primary key internal.
5.  Gunakan `DECIMAL`, bukan FLOAT, untuk uang.
6.  Gunakan UTC pada layer database/backend bila memungkinkan dan
    tampilkan menggunakan timezone Asia/Jakarta.
7.  Gunakan soft delete untuk master data yang perlu dipertahankan
    historinya.
8.  Jangan menghapus stock movement.
9.  Jangan mengubah historical transaction price melalui master data.
10. Gunakan database transaction untuk operasi stok, invoice, payment,
    dan purchase receiving.
11. Semua foreign key harus memiliki index.
12. Semua nomor transaksi harus unique.
13. Semua tabel transaksi wajib memiliki timestamp.
14. Semua operasi penting harus dapat ditelusuri melalui audit log.

------------------------------------------------------------------------

# 3. Naming Convention

Gunakan:

-   Table: `snake_case`, plural
-   Column: `snake_case`
-   Primary key: `id`
-   Foreign key: `<table_singular>_id`
-   Boolean: `is_*` atau `has_*`
-   Timestamp: `created_at`, `updated_at`
-   Soft delete: `deleted_at`

Contoh:

``` text
work_orders
work_order_services
customer_id
created_at
deleted_at
```

------------------------------------------------------------------------

# 4. Global Data Types

  Purpose      MySQL Type
  ------------ -----------------
  Primary ID   BIGINT UNSIGNED
  Code         VARCHAR(30-50)
  Name         VARCHAR(150)
  Phone        VARCHAR(30)
  Email        VARCHAR(150)
  Money        DECIMAL(15,2)
  Quantity     DECIMAL(15,3)
  Percentage   DECIMAL(5,2)
  Date         DATE
  DateTime     DATETIME
  Long text    TEXT
  JSON data    JSON
  Status       VARCHAR/ENUM

For extensibility, status fields may use `VARCHAR` with
application-level enums rather than MySQL ENUM where frequent evolution
is expected.

------------------------------------------------------------------------

# 5. ENTITY GROUPS

``` text
AUTH
 ├── users
 ├── roles
 ├── permissions
 └── role_permissions

CUSTOMER
 ├── customers
 └── vehicles

WORKSHOP
 ├── mechanics
 ├── inspection_items
 ├── work_orders
 ├── work_order_inspections
 ├── work_order_services
 ├── work_order_parts
 └── work_order_recommendations

MASTER
 ├── service_categories
 ├── services
 ├── part_categories
 └── spare_parts

WAREHOUSE
 ├── warehouses
 ├── warehouse_locations
 ├── stock_movements
 ├── stock_opnames
 └── stock_opname_items

PURCHASING
 ├── suppliers
 ├── purchases
 └── purchase_items

TRANSACTION
 ├── invoices
 └── payments

SYSTEM
 ├── settings
 └── audit_logs
```

------------------------------------------------------------------------

# 6. AUTHENTICATION TABLES

## 6.1 users

Purpose: user login and application identity.

  Field           Type                Null Key      Notes
  --------------- ----------------- ------ -------- ------------------
  id              BIGINT UNSIGNED       NO PK       Auto increment
  role_id         BIGINT UNSIGNED       NO FK       roles.id
  username        VARCHAR(80)           NO UNIQUE   Login identifier
  email           VARCHAR(150)         YES UNIQUE   Optional
  password_hash   VARCHAR(255)          NO          Hashed
  name            VARCHAR(150)          NO          Display name
  phone           VARCHAR(30)          YES          
  status          VARCHAR(20)           NO INDEX    ACTIVE/INACTIVE
  last_login_at   DATETIME             YES          
  created_at      DATETIME              NO          
  updated_at      DATETIME              NO          
  deleted_at      DATETIME             YES          Soft delete

Indexes:

``` text
UNIQUE(username)
UNIQUE(email)
INDEX(role_id)
INDEX(status)
```

------------------------------------------------------------------------

## 6.2 roles

  Field         Type                Null Key
  ------------- ----------------- ------ --------
  id            BIGINT UNSIGNED       NO PK
  code          VARCHAR(30)           NO UNIQUE
  name          VARCHAR(100)          NO 
  description   TEXT                 YES 
  created_at    DATETIME              NO 
  updated_at    DATETIME              NO 

Seed:

``` text
OWNER
ADMIN
MECHANIC
WAREHOUSE
```

------------------------------------------------------------------------

## 6.3 permissions

  Field        Type                Null Key
  ------------ ----------------- ------ --------
  id           BIGINT UNSIGNED       NO PK
  code         VARCHAR(100)          NO UNIQUE
  name         VARCHAR(150)          NO 
  module       VARCHAR(50)           NO INDEX
  action       VARCHAR(30)           NO 
  created_at   DATETIME              NO 
  updated_at   DATETIME              NO 

Examples:

``` text
work_order.view
work_order.create
work_order.update
work_order.approve
work_order.issue_part
work_order.qc
work_order.complete
inventory.view
inventory.adjust
invoice.create
payment.create
```

------------------------------------------------------------------------

## 6.4 role_permissions

  Field           Type                Null Key
  --------------- ----------------- ------ -------
  role_id         BIGINT UNSIGNED       NO PK/FK
  permission_id   BIGINT UNSIGNED       NO PK/FK

Composite primary key:

``` text
(role_id, permission_id)
```

------------------------------------------------------------------------

# 7. CUSTOMER DOMAIN

## 7.1 customers

  Field           Type                Null Key
  --------------- ----------------- ------ --------
  id              BIGINT UNSIGNED       NO PK
  customer_code   VARCHAR(30)           NO UNIQUE
  name            VARCHAR(150)          NO INDEX
  phone           VARCHAR(30)           NO INDEX
  email           VARCHAR(150)         YES 
  address         TEXT                 YES 
  notes           TEXT                 YES 
  status          VARCHAR(20)           NO INDEX
  created_at      DATETIME              NO 
  updated_at      DATETIME              NO 
  deleted_at      DATETIME             YES 

Rules:

-   `customer_code` unique.
-   Phone should be normalized.
-   Customer cannot be physically deleted if transaction history exists.

------------------------------------------------------------------------

## 7.2 vehicles

  Field            Type                  Null Key
  ---------------- ------------------- ------ ----------
  id               BIGINT UNSIGNED         NO PK
  customer_id      BIGINT UNSIGNED         NO FK/INDEX
  vehicle_code     VARCHAR(30)             NO UNIQUE
  plate_number     VARCHAR(20)             NO INDEX
  brand            VARCHAR(80)             NO INDEX
  model            VARCHAR(100)            NO INDEX
  variant          VARCHAR(100)           YES 
  year             SMALLINT UNSIGNED      YES 
  color            VARCHAR(50)            YES 
  engine_number    VARCHAR(100)           YES INDEX
  chassis_number   VARCHAR(100)           YES INDEX
  current_km       DECIMAL(10,1)           NO 
  notes            TEXT                   YES 
  status           VARCHAR(20)             NO INDEX
  created_at       DATETIME                NO 
  updated_at       DATETIME                NO 
  deleted_at       DATETIME               YES 

Recommended:

``` text
INDEX(customer_id)
INDEX(plate_number)
INDEX(engine_number)
INDEX(chassis_number)
```

Plate number should be stored normalized.

------------------------------------------------------------------------

# 8. WORKSHOP DOMAIN

## 8.1 mechanics

  Field            Type                Null Key
  ---------------- ----------------- ------ -----------
  id               BIGINT UNSIGNED       NO PK
  user_id          BIGINT UNSIGNED      YES FK/UNIQUE
  employee_code    VARCHAR(30)           NO UNIQUE
  name             VARCHAR(150)          NO INDEX
  phone            VARCHAR(30)          YES 
  specialization   VARCHAR(100)         YES 
  status           VARCHAR(20)           NO INDEX
  created_at       DATETIME              NO 
  updated_at       DATETIME              NO 

A mechanic may optionally have a login user.

------------------------------------------------------------------------

## 8.2 inspection_items

Master checklist.

  Field         Type                Null Key
  ------------- ----------------- ------ --------
  id            BIGINT UNSIGNED       NO PK
  code          VARCHAR(50)           NO UNIQUE
  category      VARCHAR(50)           NO INDEX
  name          VARCHAR(150)          NO 
  description   TEXT                 YES 
  sort_order    INT                   NO 
  is_active     BOOLEAN               NO 
  created_at    DATETIME              NO 
  updated_at    DATETIME              NO 

Seed examples:

``` text
ENGINE
ENGINE_OIL
BRAKE_FRONT
BRAKE_REAR
TIRE_FRONT
TIRE_REAR
BATTERY
CVT
SUSPENSION
ELECTRICAL
LIGHTING
CHAIN
COOLING
BODY
```

------------------------------------------------------------------------

# 9. WORK ORDERS

## 9.1 work_orders

This is the central operational table.

  Field               Type                Null Key
  ------------------- ----------------- ------ ----------
  id                  BIGINT UNSIGNED       NO PK
  work_order_number   VARCHAR(40)           NO UNIQUE
  customer_id         BIGINT UNSIGNED       NO FK/INDEX
  vehicle_id          BIGINT UNSIGNED       NO FK/INDEX
  mechanic_id         BIGINT UNSIGNED       NO FK/INDEX
  warehouse_id        BIGINT UNSIGNED       NO FK/INDEX
  opened_at           DATETIME              NO INDEX
  started_at          DATETIME             YES 
  completed_at        DATETIME             YES 
  current_km          DECIMAL(10,1)         NO 
  complaint           TEXT                  NO 
  diagnosis           TEXT                 YES 
  customer_notes      TEXT                 YES 
  internal_notes      TEXT                 YES 
  status              VARCHAR(30)           NO INDEX
  approval_status     VARCHAR(20)           NO INDEX
  approval_at         DATETIME             YES 
  approved_by         BIGINT UNSIGNED      YES FK
  created_by          BIGINT UNSIGNED       NO FK
  updated_by          BIGINT UNSIGNED      YES FK
  created_at          DATETIME              NO 
  updated_at          DATETIME              NO 

Indexes:

``` text
UNIQUE(work_order_number)
INDEX(customer_id)
INDEX(vehicle_id)
INDEX(mechanic_id)
INDEX(warehouse_id)
INDEX(status)
INDEX(opened_at)
INDEX(status, opened_at)
```

------------------------------------------------------------------------

# 10. WORK ORDER INSPECTION

## 10.1 work_order_inspections

  Field                Type                Null Key
  -------------------- ----------------- ------ -----
  id                   BIGINT UNSIGNED       NO PK
  work_order_id        BIGINT UNSIGNED       NO FK
  inspection_item_id   BIGINT UNSIGNED       NO FK
  result               VARCHAR(30)           NO 
  measurement          VARCHAR(100)         YES 
  notes                TEXT                 YES 
  recommendation       TEXT                 YES 
  inspected_by         BIGINT UNSIGNED       NO FK
  inspected_at         DATETIME              NO 
  created_at           DATETIME              NO 
  updated_at           DATETIME              NO 

Unique:

``` text
UNIQUE(work_order_id, inspection_item_id)
```

------------------------------------------------------------------------

# 11. SERVICE MASTER

## 11.1 service_categories

  Field         Type                Null Key
  ------------- ----------------- ------ --------
  id            BIGINT UNSIGNED       NO PK
  code          VARCHAR(30)           NO UNIQUE
  name          VARCHAR(100)          NO 
  description   TEXT                 YES 
  status        VARCHAR(20)           NO 
  created_at    DATETIME              NO 
  updated_at    DATETIME              NO 
  deleted_at    DATETIME             YES 

------------------------------------------------------------------------

## 11.2 services

  Field                        Type                Null Key
  ---------------------------- ----------------- ------ ----------
  id                           BIGINT UNSIGNED       NO PK
  service_code                 VARCHAR(30)           NO UNIQUE
  category_id                  BIGINT UNSIGNED       NO FK/INDEX
  name                         VARCHAR(150)          NO INDEX
  description                  TEXT                 YES 
  price                        DECIMAL(15,2)         NO 
  estimated_duration_minutes   INT UNSIGNED         YES 
  status                       VARCHAR(20)           NO INDEX
  created_at                   DATETIME              NO 
  updated_at                   DATETIME              NO 
  deleted_at                   DATETIME             YES 

------------------------------------------------------------------------

# 12. WORK ORDER SERVICES

## 12.1 work_order_services

  Field           Type                Null Key
  --------------- ----------------- ------ ----------
  id              BIGINT UNSIGNED       NO PK
  work_order_id   BIGINT UNSIGNED       NO FK/INDEX
  service_id      BIGINT UNSIGNED       NO FK
  description     VARCHAR(255)         YES 
  qty             DECIMAL(15,3)         NO 
  unit_price      DECIMAL(15,2)         NO 
  discount        DECIMAL(15,2)         NO 
  subtotal        DECIMAL(15,2)         NO 
  created_at      DATETIME              NO 
  updated_at      DATETIME              NO 

Formula:

``` text
subtotal = (qty × unit_price) - discount
```

Historical price is frozen here.

------------------------------------------------------------------------

# 13. PART MASTER

## 13.1 part_categories

  Field         Type                Null Key
  ------------- ----------------- ------ --------
  id            BIGINT UNSIGNED       NO PK
  code          VARCHAR(30)           NO UNIQUE
  name          VARCHAR(100)          NO 
  description   TEXT                 YES 
  status        VARCHAR(20)           NO 
  created_at    DATETIME              NO 
  updated_at    DATETIME              NO 
  deleted_at    DATETIME             YES 

------------------------------------------------------------------------

## 13.2 spare_parts

  Field            Type                Null Key
  ---------------- ----------------- ------ ----------
  id               BIGINT UNSIGNED       NO PK
  sku              VARCHAR(50)           NO UNIQUE
  barcode          VARCHAR(100)         YES UNIQUE
  name             VARCHAR(150)          NO INDEX
  category_id      BIGINT UNSIGNED       NO FK/INDEX
  brand            VARCHAR(80)          YES INDEX
  unit             VARCHAR(20)           NO 
  purchase_price   DECIMAL(15,2)         NO 
  selling_price    DECIMAL(15,2)         NO 
  minimum_stock    DECIMAL(15,3)         NO 
  maximum_stock    DECIMAL(15,3)        YES 
  status           VARCHAR(20)           NO INDEX
  created_at       DATETIME              NO 
  updated_at       DATETIME              NO 
  deleted_at       DATETIME             YES 

Barcode can be nullable but, when present, must be unique.

------------------------------------------------------------------------

# 14. WORK ORDER PARTS

## 14.1 work_order_parts

  Field           Type                Null Key
  --------------- ----------------- ------ ----------
  id              BIGINT UNSIGNED       NO PK
  work_order_id   BIGINT UNSIGNED       NO FK/INDEX
  part_id         BIGINT UNSIGNED       NO FK/INDEX
  warehouse_id    BIGINT UNSIGNED       NO FK
  location_id     BIGINT UNSIGNED      YES FK
  qty             DECIMAL(15,3)         NO 
  unit_price      DECIMAL(15,2)         NO 
  cost_price      DECIMAL(15,2)         NO 
  discount        DECIMAL(15,2)         NO 
  subtotal        DECIMAL(15,2)         NO 
  stock_issued    BOOLEAN               NO 
  issued_at       DATETIME             YES 
  issued_by       BIGINT UNSIGNED      YES FK
  created_at      DATETIME              NO 
  updated_at      DATETIME              NO 

Important:

`cost_price` is frozen at transaction level.

------------------------------------------------------------------------

# 15. RECOMMENDATIONS

## 15.1 work_order_recommendations

  Field             Type                Null Key
  ----------------- ----------------- ------ ----------
  id                BIGINT UNSIGNED       NO PK
  work_order_id     BIGINT UNSIGNED       NO FK/INDEX
  service_id        BIGINT UNSIGNED      YES FK
  part_id           BIGINT UNSIGNED      YES FK
  description       VARCHAR(255)          NO 
  estimated_price   DECIMAL(15,2)         NO 
  priority          VARCHAR(20)           NO 
  status            VARCHAR(20)           NO 
  notes             TEXT                 YES 
  created_by        BIGINT UNSIGNED       NO FK
  created_at        DATETIME              NO 
  updated_at        DATETIME              NO 

Exactly one of `service_id` or `part_id` should normally be supplied,
unless recommendation is free-form.

------------------------------------------------------------------------

# 16. WAREHOUSE

## 16.1 warehouses

  Field        Type                Null Key
  ------------ ----------------- ------ --------
  id           BIGINT UNSIGNED       NO PK
  code         VARCHAR(30)           NO UNIQUE
  name         VARCHAR(100)          NO 
  address      TEXT                 YES 
  status       VARCHAR(20)           NO 
  created_at   DATETIME              NO 
  updated_at   DATETIME              NO 

------------------------------------------------------------------------

## 16.2 warehouse_locations

  Field          Type                Null Key
  -------------- ----------------- ------ ----------
  id             BIGINT UNSIGNED       NO PK
  warehouse_id   BIGINT UNSIGNED       NO FK/INDEX
  code           VARCHAR(30)           NO 
  name           VARCHAR(100)          NO 
  rack           VARCHAR(30)          YES 
  shelf          VARCHAR(30)          YES 
  status         VARCHAR(20)           NO 
  created_at     DATETIME              NO 
  updated_at     DATETIME              NO 

Unique:

``` text
UNIQUE(warehouse_id, code)
```

------------------------------------------------------------------------

# 17. INVENTORY LEDGER

## 17.1 stock_movements

This table is the source of truth for inventory history.

  Field            Type                Null Key
  ---------------- ----------------- ------ ----------
  id               BIGINT UNSIGNED       NO PK
  part_id          BIGINT UNSIGNED       NO FK/INDEX
  warehouse_id     BIGINT UNSIGNED       NO FK/INDEX
  location_id      BIGINT UNSIGNED      YES FK
  movement_type    VARCHAR(30)           NO INDEX
  reference_type   VARCHAR(50)          YES INDEX
  reference_id     BIGINT UNSIGNED      YES INDEX
  quantity         DECIMAL(15,3)         NO 
  unit_cost        DECIMAL(15,2)         NO 
  balance_after    DECIMAL(15,3)         NO 
  notes            TEXT                 YES 
  created_by       BIGINT UNSIGNED       NO FK
  created_at       DATETIME              NO 

Indexes:

``` text
INDEX(part_id, warehouse_id, created_at)
INDEX(reference_type, reference_id)
INDEX(movement_type)
```

Stock movement is immutable.

------------------------------------------------------------------------

# 18. STOCK OPNAME

## 18.1 stock_opnames

  Field           Type                Null Key
  --------------- ----------------- ------ --------
  id              BIGINT UNSIGNED       NO PK
  opname_number   VARCHAR(40)           NO UNIQUE
  warehouse_id    BIGINT UNSIGNED       NO FK
  opname_date     DATETIME              NO 
  status          VARCHAR(20)           NO INDEX
  notes           TEXT                 YES 
  created_by      BIGINT UNSIGNED       NO FK
  posted_by       BIGINT UNSIGNED      YES FK
  posted_at       DATETIME             YES 
  created_at      DATETIME              NO 
  updated_at      DATETIME              NO 

Statuses:

``` text
DRAFT
COUNTING
REVIEW
POSTED
CANCELLED
```

------------------------------------------------------------------------

## 18.2 stock_opname_items

  Field             Type                Null Key
  ----------------- ----------------- ------ -----
  id                BIGINT UNSIGNED       NO PK
  stock_opname_id   BIGINT UNSIGNED       NO FK
  part_id           BIGINT UNSIGNED       NO FK
  location_id       BIGINT UNSIGNED      YES FK
  system_qty        DECIMAL(15,3)         NO 
  counted_qty       DECIMAL(15,3)         NO 
  variance_qty      DECIMAL(15,3)         NO 
  notes             TEXT                 YES 
  created_at        DATETIME              NO 
  updated_at        DATETIME              NO 

Formula:

``` text
variance_qty = counted_qty - system_qty
```

------------------------------------------------------------------------

# 19. SUPPLIER

## 19.1 suppliers

  Field           Type                Null Key
  --------------- ----------------- ------ --------
  id              BIGINT UNSIGNED       NO PK
  supplier_code   VARCHAR(30)           NO UNIQUE
  name            VARCHAR(150)          NO INDEX
  phone           VARCHAR(30)          YES 
  email           VARCHAR(150)         YES 
  address         TEXT                 YES 
  notes           TEXT                 YES 
  status          VARCHAR(20)           NO 
  created_at      DATETIME              NO 
  updated_at      DATETIME              NO 
  deleted_at      DATETIME             YES 

------------------------------------------------------------------------

# 20. PURCHASE

## 20.1 purchases

  Field             Type                Null Key
  ----------------- ----------------- ------ ----------
  id                BIGINT UNSIGNED       NO PK
  purchase_number   VARCHAR(40)           NO UNIQUE
  supplier_id       BIGINT UNSIGNED       NO FK/INDEX
  warehouse_id      BIGINT UNSIGNED       NO FK
  purchase_date     DATETIME              NO 
  subtotal          DECIMAL(15,2)         NO 
  discount          DECIMAL(15,2)         NO 
  total             DECIMAL(15,2)         NO 
  status            VARCHAR(20)           NO INDEX
  notes             TEXT                 YES 
  created_by        BIGINT UNSIGNED       NO FK
  created_at        DATETIME              NO 
  updated_at        DATETIME              NO 

Statuses:

``` text
DRAFT
ORDERED
PARTIAL_RECEIVED
RECEIVED
CANCELLED
```

------------------------------------------------------------------------

## 20.2 purchase_items

  Field          Type                Null Key
  -------------- ----------------- ------ -----
  id             BIGINT UNSIGNED       NO PK
  purchase_id    BIGINT UNSIGNED       NO FK
  part_id        BIGINT UNSIGNED       NO FK
  qty            DECIMAL(15,3)         NO 
  unit_cost      DECIMAL(15,2)         NO 
  subtotal       DECIMAL(15,2)         NO 
  received_qty   DECIMAL(15,3)         NO 
  created_at     DATETIME              NO 
  updated_at     DATETIME              NO 

------------------------------------------------------------------------

# 21. INVOICE

## 21.1 invoices

  Field                Type                Null Key
  -------------------- ----------------- ------ -----------
  id                   BIGINT UNSIGNED       NO PK
  invoice_number       VARCHAR(40)           NO UNIQUE
  work_order_id        BIGINT UNSIGNED       NO FK/UNIQUE
  invoice_date         DATETIME              NO 
  subtotal_service     DECIMAL(15,2)         NO 
  subtotal_part        DECIMAL(15,2)         NO 
  discount             DECIMAL(15,2)         NO 
  tax                  DECIMAL(15,2)         NO 
  grand_total          DECIMAL(15,2)         NO 
  paid_amount          DECIMAL(15,2)         NO 
  outstanding_amount   DECIMAL(15,2)         NO 
  status               VARCHAR(20)           NO INDEX
  created_by           BIGINT UNSIGNED       NO FK
  created_at           DATETIME              NO 
  updated_at           DATETIME              NO 

One completed WO should normally produce one active invoice in V1.

------------------------------------------------------------------------

# 22. PAYMENTS

## 22.1 payments

  Field              Type                Null Key
  ------------------ ----------------- ------ ----------
  id                 BIGINT UNSIGNED       NO PK
  invoice_id         BIGINT UNSIGNED       NO FK/INDEX
  payment_number     VARCHAR(40)           NO UNIQUE
  payment_method     VARCHAR(30)           NO INDEX
  amount             DECIMAL(15,2)         NO 
  paid_at            DATETIME              NO INDEX
  reference_number   VARCHAR(100)         YES 
  notes              TEXT                 YES 
  received_by        BIGINT UNSIGNED       NO FK
  created_at         DATETIME              NO 

Rules:

``` text
amount > 0
SUM(payments.amount) <= invoice.grand_total
```

------------------------------------------------------------------------

# 23. SETTINGS

## 23.1 settings

  Field           Type                Null Key
  --------------- ----------------- ------ --------
  id              BIGINT UNSIGNED       NO PK
  setting_key     VARCHAR(100)          NO UNIQUE
  setting_value   TEXT                 YES 
  setting_type    VARCHAR(20)           NO 
  description     TEXT                 YES 
  updated_by      BIGINT UNSIGNED      YES FK
  created_at      DATETIME              NO 
  updated_at      DATETIME              NO 

Examples:

``` text
workshop.name
workshop.address
workshop.phone
workshop.logo
invoice.footer
currency.code
currency.symbol
tax.enabled
tax.rate
default.warehouse_id
timezone
```

------------------------------------------------------------------------

# 24. AUDIT LOG

## 24.1 audit_logs

  Field         Type                Null Key
  ------------- ----------------- ------ ----------
  id            BIGINT UNSIGNED       NO PK
  user_id       BIGINT UNSIGNED      YES FK/INDEX
  action        VARCHAR(100)          NO INDEX
  entity_type   VARCHAR(100)          NO INDEX
  entity_id     BIGINT UNSIGNED      YES INDEX
  old_values    JSON                 YES 
  new_values    JSON                 YES 
  ip_address    VARCHAR(45)          YES 
  user_agent    TEXT                 YES 
  created_at    DATETIME              NO INDEX

Examples:

``` text
WORK_ORDER_CREATED
WORK_ORDER_APPROVED
WORK_ORDER_STATUS_CHANGED
PART_ISSUED
STOCK_ADJUSTED
STOCK_OPNAME_POSTED
PURCHASE_RECEIVED
INVOICE_CREATED
INVOICE_VOIDED
PAYMENT_CREATED
```

------------------------------------------------------------------------

# 25. FOREIGN KEY MAP

``` text
users.role_id
    -> roles.id

role_permissions.role_id
    -> roles.id

role_permissions.permission_id
    -> permissions.id

mechanics.user_id
    -> users.id

vehicles.customer_id
    -> customers.id

work_orders.customer_id
    -> customers.id

work_orders.vehicle_id
    -> vehicles.id

work_orders.mechanic_id
    -> mechanics.id

work_orders.warehouse_id
    -> warehouses.id

work_orders.approved_by
    -> users.id

work_orders.created_by
    -> users.id

work_order_inspections.work_order_id
    -> work_orders.id

work_order_inspections.inspection_item_id
    -> inspection_items.id

work_order_inspections.inspected_by
    -> users.id

services.category_id
    -> service_categories.id

work_order_services.work_order_id
    -> work_orders.id

work_order_services.service_id
    -> services.id

spare_parts.category_id
    -> part_categories.id

work_order_parts.work_order_id
    -> work_orders.id

work_order_parts.part_id
    -> spare_parts.id

work_order_parts.warehouse_id
    -> warehouses.id

work_order_parts.location_id
    -> warehouse_locations.id

work_order_parts.issued_by
    -> users.id

work_order_recommendations.work_order_id
    -> work_orders.id

work_order_recommendations.service_id
    -> services.id

work_order_recommendations.part_id
    -> spare_parts.id

warehouses
    -> warehouse_locations

stock_movements.part_id
    -> spare_parts.id

stock_movements.warehouse_id
    -> warehouses.id

stock_movements.location_id
    -> warehouse_locations.id

stock_movements.created_by
    -> users.id

stock_opnames.warehouse_id
    -> warehouses.id

stock_opname_items.stock_opname_id
    -> stock_opnames.id

stock_opname_items.part_id
    -> spare_parts.id

stock_opname_items.location_id
    -> warehouse_locations.id

suppliers
    -> purchases

purchases.supplier_id
    -> suppliers.id

purchases.warehouse_id
    -> warehouses.id

purchase_items.purchase_id
    -> purchases.id

purchase_items.part_id
    -> spare_parts.id

invoices.work_order_id
    -> work_orders.id

invoices.created_by
    -> users.id

payments.invoice_id
    -> invoices.id

payments.received_by
    -> users.id

audit_logs.user_id
    -> users.id
```

------------------------------------------------------------------------

# 26. MASTER ERD

``` text
┌──────────────┐
│    ROLES     │
└──────┬───────┘
       │ 1:N
       ▼
┌──────────────┐
│    USERS     │
└──────┬───────┘
       │
       ├───────────────┐
       ▼               ▼
┌──────────────┐   ┌──────────────┐
│  MECHANICS   │   │ AUDIT_LOGS   │
└──────────────┘   └──────────────┘


┌──────────────┐
│  CUSTOMERS   │
└──────┬───────┘
       │ 1:N
       ▼
┌──────────────┐
│   VEHICLES   │
└──────┬───────┘
       │ 1:N
       ▼
┌────────────────────┐
│    WORK_ORDERS     │
└────┬─────┬─────┬───┘
     │     │     │
     │     │     └───────────────┐
     │     │                     │
     ▼     ▼                     ▼
INSPECTION SERVICES             PARTS
     │     │                     │
     │     ▼                     ▼
     │  SERVICES              SPARE_PARTS
     │                            │
     │                            ▼
     │                      STOCK_MOVEMENTS
     │
     ▼
RECOMMENDATIONS


┌──────────────┐
│  WAREHOUSES  │
└──────┬───────┘
       │ 1:N
       ▼
┌────────────────────┐
│ WAREHOUSE_LOCATIONS│
└────────────────────┘


┌──────────────┐
│  SUPPLIERS   │
└──────┬───────┘
       │ 1:N
       ▼
┌──────────────┐
│  PURCHASES   │
└──────┬───────┘
       │ 1:N
       ▼
┌──────────────┐
│PURCHASE_ITEMS│
└──────┬───────┘
       │ N:1
       ▼
 SPARE_PARTS


WORK_ORDERS
     │
     │ 1:1
     ▼
┌──────────────┐
│   INVOICES   │
└──────┬───────┘
       │ 1:N
       ▼
┌──────────────┐
│   PAYMENTS   │
└──────────────┘
```

------------------------------------------------------------------------

# 27. CARDINALITY SUMMARY

  Parent             Child             Relationship
  ------------------ ----------------- --------------
  Role               Users             1:N
  Role               Permissions       N:M
  Customer           Vehicles          1:N
  Vehicle            Work Orders       1:N
  Mechanic           Work Orders       1:N
  Work Order         Inspections       1:N
  Work Order         Services          1:N
  Work Order         Parts             1:N
  Work Order         Recommendations   1:N
  Service Category   Services          1:N
  Part Category      Spare Parts       1:N
  Warehouse          Locations         1:N
  Spare Part         Stock Movements   1:N
  Warehouse          Stock Movements   1:N
  Supplier           Purchases         1:N
  Purchase           Purchase Items    1:N
  Spare Part         Purchase Items    1:N
  Work Order         Invoice           1:1
  Invoice            Payments          1:N
  Stock Opname       Items             1:N

------------------------------------------------------------------------

# 28. DELETE POLICY

## Master Data

Use soft delete:

``` text
customers
vehicles
services
service_categories
spare_parts
part_categories
suppliers
```

## Transaction Data

Do not hard delete:

``` text
work_orders
work_order_services
work_order_parts
invoices
payments
stock_movements
purchases
```

Instead use: - CANCEL - VOID - REVERSAL - ADJUSTMENT

------------------------------------------------------------------------

# 29. STOCK SOURCE OF TRUTH

For V1, stock balance is derived from stock movements.

Concept:

``` text
Opening Balance
+
Purchase IN
+
Adjustment IN
+
Return IN
+
Transfer IN
-
Work Order OUT
-
Adjustment OUT
-
Transfer OUT
=
Current Stock
```

`balance_after` is stored for audit and fast display but must be
generated transactionally.

------------------------------------------------------------------------

# 30. STOCK CONSISTENCY RULE

When issuing a part:

``` text
BEGIN

LOCK relevant inventory state

CHECK available quantity

IF available < requested:
    ROLLBACK
    return INSUFFICIENT_STOCK

CREATE work_order_part
CREATE stock_movement OUT
SET balance_after

COMMIT
```

No concurrent transaction may cause stock to become negative.

------------------------------------------------------------------------

# 31. PURCHASE RECEIVING RULE

When receiving:

``` text
BEGIN

Validate purchase status

For each item:
    validate received_qty
    create stock movement PURCHASE
    update received_qty

Update purchase status

COMMIT
```

Status:

``` text
ORDERED
   ↓
PARTIAL_RECEIVED
   ↓
RECEIVED
```

------------------------------------------------------------------------

# 32. STOCK OPNAME RULE

When posted:

``` text
system_qty
counted_qty
variance = counted - system
```

If variance \> 0:

``` text
STOCK_OPNAME_IN
```

If variance \< 0:

``` text
STOCK_OPNAME_OUT
```

Every variance generates a stock movement.

------------------------------------------------------------------------

# 33. INVOICE DATA INTEGRITY

Invoice totals must be calculated server-side.

``` text
service_subtotal
+
part_subtotal
-
discount
+
tax
=
grand_total
```

Do not trust totals sent by frontend.

Backend recalculates.

------------------------------------------------------------------------

# 34. PAYMENT DATA INTEGRITY

``` text
paid_amount =
SUM(valid payments)

outstanding =
grand_total - paid_amount
```

Payment status:

``` text
paid_amount = 0
    → ISSUED

0 < paid_amount < grand_total
    → PARTIAL

paid_amount = grand_total
    → PAID
```

Never allow:

``` text
paid_amount > grand_total
```

------------------------------------------------------------------------

# 35. HISTORICAL PRICE RULE

Master:

``` text
services.price
spare_parts.selling_price
spare_parts.purchase_price
```

Transaction:

``` text
work_order_services.unit_price
work_order_parts.unit_price
work_order_parts.cost_price
purchase_items.unit_cost
```

Master changes must never mutate transaction history.

------------------------------------------------------------------------

# 36. RECOMMENDATION CONVERSION

When customer approves a recommendation:

``` text
RECOMMENDED
     ↓
APPROVED
     ↓
CONVERTED
     ↓
work_order_service
or
work_order_part
```

No duplicate item should be created accidentally.

Conversion should happen inside a database transaction.

------------------------------------------------------------------------

# 37. SERVICE HISTORY

Service history does not require a separate table in V1.

It can be generated from:

``` text
work_orders
work_order_services
work_order_parts
invoices
payments
```

Vehicle history query:

``` text
vehicles.id
    ↓
work_orders.vehicle_id
    ↓
completed work orders
    ↓
services + parts + invoice
```

This avoids duplicate data.

------------------------------------------------------------------------

# 38. INDEX STRATEGY

Critical indexes:

``` text
customers.phone
customers.name

vehicles.plate_number
vehicles.customer_id

work_orders.work_order_number
work_orders.customer_id
work_orders.vehicle_id
work_orders.mechanic_id
work_orders.status
work_orders.opened_at

services.service_code
services.category_id

spare_parts.sku
spare_parts.barcode
spare_parts.name
spare_parts.category_id

stock_movements.part_id
stock_movements.warehouse_id
stock_movements.created_at

invoices.invoice_number
invoices.work_order_id
invoices.status

payments.payment_number
payments.invoice_id
payments.paid_at
```

------------------------------------------------------------------------

# 39. SEQUELIZE MODEL ORDER

Recommended migration/model creation order:

``` text
01_roles
02_permissions
03_role_permissions
04_users

05_customers
06_vehicles
07_mechanics

08_service_categories
09_services

10_part_categories
11_spare_parts

12_warehouses
13_warehouse_locations

14_inspection_items

15_work_orders
16_work_order_inspections
17_work_order_services
18_work_order_parts
19_work_order_recommendations

20_stock_movements
21_stock_opnames
22_stock_opname_items

23_suppliers
24_purchases
25_purchase_items

26_invoices
27_payments

28_settings
29_audit_logs
```

------------------------------------------------------------------------

# 40. SEQUELIZE ASSOCIATIONS

Conceptual associations:

``` javascript
Role.hasMany(User)
User.belongsTo(Role)

Customer.hasMany(Vehicle)
Vehicle.belongsTo(Customer)

User.hasOne(Mechanic)
Mechanic.belongsTo(User)

Mechanic.hasMany(WorkOrder)
WorkOrder.belongsTo(Mechanic)

Customer.hasMany(WorkOrder)
WorkOrder.belongsTo(Customer)

Vehicle.hasMany(WorkOrder)
WorkOrder.belongsTo(Vehicle)

WorkOrder.hasMany(WorkOrderInspection)
WorkOrderInspection.belongsTo(WorkOrder)

InspectionItem.hasMany(WorkOrderInspection)
WorkOrderInspection.belongsTo(InspectionItem)

ServiceCategory.hasMany(Service)
Service.belongsTo(ServiceCategory)

WorkOrder.hasMany(WorkOrderService)
WorkOrderService.belongsTo(WorkOrder)
WorkOrderService.belongsTo(Service)

PartCategory.hasMany(SparePart)
SparePart.belongsTo(PartCategory)

WorkOrder.hasMany(WorkOrderPart)
WorkOrderPart.belongsTo(WorkOrder)
WorkOrderPart.belongsTo(SparePart)

Warehouse.hasMany(WarehouseLocation)
WarehouseLocation.belongsTo(Warehouse)

SparePart.hasMany(StockMovement)
StockMovement.belongsTo(SparePart)

Warehouse.hasMany(StockMovement)
StockMovement.belongsTo(Warehouse)

Supplier.hasMany(Purchase)
Purchase.belongsTo(Supplier)

Purchase.hasMany(PurchaseItem)
PurchaseItem.belongsTo(Purchase)
PurchaseItem.belongsTo(SparePart)

WorkOrder.hasOne(Invoice)
Invoice.belongsTo(WorkOrder)

Invoice.hasMany(Payment)
Payment.belongsTo(Invoice)
```

------------------------------------------------------------------------

# 41. SEED DATA

Required seed:

## Roles

``` text
OWNER
ADMIN
MECHANIC
WAREHOUSE
```

## Default Admin

Use environment variables for credentials.

Never hard-code production passwords.

## Service Categories

``` text
SERVICE
ENGINE
CVT
BRAKE
ELECTRICAL
SUSPENSION
```

## Part Categories

``` text
OIL
BRAKE
ENGINE
CVT
ELECTRICAL
BATTERY
TIRE
```

## Inspection Items

``` text
Engine
Engine Oil
Brake Front
Brake Rear
Tire Front
Tire Rear
Battery
CVT
Suspension
Electrical
Lighting
Chain
Cooling
Body
```

------------------------------------------------------------------------

# 42. DATABASE TRANSACTION REQUIREMENTS

Mandatory transaction:

``` text
Create Work Order
Approve Work Order
Issue Part
Return Part
Stock Adjustment
Stock Opname Posting
Purchase Receiving
Create Invoice
Create Payment
Void Invoice
Recommendation Conversion
```

------------------------------------------------------------------------

# 43. CONCURRENCY CONTROL

Inventory operations must prevent:

``` text
User A sees stock = 1
User B sees stock = 1

A issues 1
B issues 1

Result must NOT become -1
```

Use: - database transaction - row locking where applicable - atomic
validation/update - unique references

------------------------------------------------------------------------

# 44. DATA RETENTION

Operational transaction history should be retained.

Never delete: - Work Order history - Invoice history - Payment history -
Stock movement history

Master data can be deactivated.

------------------------------------------------------------------------

# 45. DATABASE BACKUP

Production recommendation:

``` text
Daily full backup
+
Periodic incremental/binlog backup
```

Minimum:

``` text
automated daily MySQL backup
retention >= 7 days
```

For production, test restoration periodically.

------------------------------------------------------------------------

# 46. DATABASE ACCEPTANCE CRITERIA

Database is considered ready when:

1.  All tables migrate successfully.
2.  All foreign keys work.
3.  Seed data works.
4.  Unique constraints work.
5.  Soft deletes work.
6.  Transaction rollback works.
7.  Stock cannot become negative.
8.  Invoice cannot exceed calculated totals.
9.  Payment cannot exceed invoice.
10. Historical prices remain unchanged after master updates.
11. Audit records are created for important mutations.
12. Service history can be reconstructed from transaction data.
13. Stock history can be reconstructed from ledger.
14. Sequelize associations load correctly.
15. Database backup and restore are verified.

------------------------------------------------------------------------

# 47. DEVELOPMENT OUTPUT

From this document, Claude Code should generate:

``` text
backend/src/models/
backend/src/migrations/
backend/src/seeders/
backend/src/database/
```

Expected artifacts:

``` text
All Sequelize models
All migrations
All associations
Seeders
Database configuration
Transaction utilities
Inventory ledger service
```

------------------------------------------------------------------------

# 48. NEXT DOCUMENT

After this database specification is implemented and reviewed, create:

``` text
GARAGE_PRO_API_SPECIFICATION_V1.md
```

That document should define:

-   Endpoint
-   HTTP method
-   Authentication
-   Permission
-   Request params
-   Request body
-   Validation
-   Business logic
-   Database transaction
-   Response
-   Error codes
-   Pagination
-   Sorting
-   Filtering
-   Example requests
-   Example responses

The API specification must use this database document as its source of
truth.

------------------------------------------------------------------------

# END OF DOCUMENT

**GARAGE PRO --- DATABASE DESIGN & ERD V1**\
**Status: Development Ready**
