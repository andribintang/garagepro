# GARAGE PRO --- FRONTEND INTEGRATION V1

## End-to-End Frontend Implementation Specification

**Project:** GARAGE PRO --- Workshop Management System\
**Phase:** 15 --- Frontend Integration\
**Version:** V1.0\
**Status:** Implementation Ready\
**Primary stack:** React + Vite + TypeScript + Tailwind CSS + React
Router + TanStack Query + React Hook Form + Zod + Axios

------------------------------------------------------------------------

# 1. PURPOSE

Phase 15 menyatukan seluruh backend/domain engine yang telah dibuat
menjadi satu aplikasi frontend GARAGE PRO yang dapat digunakan secara
end-to-end.

Target utama:

``` text
LOGIN
  ↓
DASHBOARD
  ↓
CUSTOMER
  ↓
VEHICLE
  ↓
WORK ORDER
  ↓
INSPECTION
  ↓
SERVICE + SPARE PART
  ↓
APPROVAL
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
  ↓
REPORTS
```

Frontend harus menjadi satu cohesive application, bukan kumpulan halaman
yang berdiri sendiri.

------------------------------------------------------------------------

# 2. OBJECTIVES

Frontend V1 harus:

1.  Terhubung ke seluruh API yang sudah tersedia.
2.  Menggunakan authentication/RBAC yang sudah ada.
3.  Menggunakan TanStack Query sebagai server-state layer.
4.  Menggunakan React Hook Form + Zod untuk form validation.
5.  Menggunakan design system GARAGE PRO.
6.  Memiliki responsive desktop/mobile behavior.
7.  Menyediakan workflow berbeda sesuai role.
8.  Menampilkan status Work Order secara real-time setelah mutation
    berhasil.
9.  Mengintegrasikan Inventory, Invoice, Payment, dan Reports.
10. Memiliki loading, empty, error, forbidden, dan success states.
11. Tidak menghitung financial total sebagai source of truth di
    frontend.
12. Meminimalkan klik pada workflow mekanik dan cashier.
13. Siap menjadi PWA.
14. Memiliki UX yang konsisten di seluruh modul.

------------------------------------------------------------------------

# 3. UX PRINCIPLES

## 3.1 Workflow First

UI harus mengikuti cara kerja bengkel:

``` text
Customer → Motorcycle → Inspection → Work → QC → Payment
```

Jangan memaksa user memahami struktur database.

## 3.2 Role First

Setelah login, user melihat fitur yang relevan dengan role.

### OWNER

Fokus:

``` text
Revenue
Unit Entry
Outstanding
Performance
Reports
```

### ADMIN

Fokus:

``` text
Customer
Vehicle
WO
Invoice
Payment
```

### MECHANIC

Fokus:

``` text
My Work Orders
Inspection
Services
Parts
Recommendations
QC
```

### WAREHOUSE

Fokus:

``` text
Stock
Issue
Return
Receiving
Opname
Low Stock
```

------------------------------------------------------------------------

# 4. APPLICATION SHELL

Desktop:

``` text
┌─────────────────────────────────────────────────────────────┐
│ Logo   Search              Notifications   User             │
├──────────────┬──────────────────────────────────────────────┤
│ Dashboard    │                                              │
│ Work Orders  │                  Main Content                 │
│ Customers    │                                              │
│ Vehicles     │                                              │
│ Services     │                                              │
│ Spare Parts  │                                              │
│ Inventory    │                                              │
│ Purchasing   │                                              │
│ Invoices     │                                              │
│ Reports      │                                              │
│ Settings     │                                              │
└──────────────┴──────────────────────────────────────────────┘
```

Mobile:

``` text
┌──────────────────────────┐
│ ☰ GARAGE PRO      🔔     │
├──────────────────────────┤
│                          │
│       Main Content       │
│                          │
├──────────────────────────┤
│ Home │ WO │ + │ Stock │ More │
└──────────────────────────┘
```

Bottom navigation harus role-aware.

------------------------------------------------------------------------

# 5. ROUTE ARCHITECTURE

Recommended route tree:

``` text
/login

/app
/app/dashboard

/app/work-orders
/app/work-orders/new
/app/work-orders/:id
/app/work-orders/:id/inspection
/app/work-orders/:id/services
/app/work-orders/:id/parts
/app/work-orders/:id/recommendations
/app/work-orders/:id/qc

/app/customers
/app/customers/new
/app/customers/:id
/app/customers/:id/edit

/app/vehicles
/app/vehicles/:id
/app/vehicles/:id/history

/app/services
/app/services/categories

/app/parts
/app/parts/new
/app/parts/:id
/app/parts/categories

/app/inventory
/app/inventory/movements
/app/inventory/low-stock
/app/inventory/opname
/app/inventory/adjustment

/app/purchasing
/app/purchasing/suppliers
/app/purchasing/orders
/app/purchasing/receiving

/app/invoices
/app/invoices/:id

/app/payments
/app/payments/:id

/app/reports/revenue
/app/reports/work-orders
/app/reports/services
/app/reports/parts
/app/reports/mechanics
/app/reports/stock

/app/users
/app/roles
/app/settings/workshop
/app/settings/general
/app/audit
```

------------------------------------------------------------------------

# 6. AUTHENTICATION INTEGRATION

Frontend auth state:

``` ts
interface AuthUser {
  id: number;
  name: string;
  username: string;
  role: string;
  permissions: string[];
}
```

Application startup:

``` text
App Start
 ↓
Check auth state
 ↓
GET /auth/me
 ↓
Success → App
Failure → Login
```

Use existing authentication architecture.

Do not duplicate authentication logic in each page.

------------------------------------------------------------------------

# 7. AXIOS CLIENT

Create centralized API client:

``` text
src/lib/api/client.ts
```

Responsibilities:

-   base URL
-   authorization
-   request ID
-   response handling
-   error normalization
-   refresh token behavior if supported

Example:

``` ts
api.get("/work-orders");
api.post("/work-orders", payload);
```

Components must not directly use raw `fetch()`.

------------------------------------------------------------------------

# 8. API ERROR NORMALIZATION

Normalize backend errors:

``` ts
interface ApiError {
  code: string;
  message: string;
  requestId?: string;
  fields?: Record<string, string>;
}
```

UI behavior:

``` text
VALIDATION_ERROR
→ show field errors

401
→ refresh/login

403
→ forbidden state

404
→ not found

409
→ business conflict toast/dialog

422
→ business validation message

500
→ generic server error
```

Never expose stack traces.

------------------------------------------------------------------------

# 9. TANSTACK QUERY ARCHITECTURE

Each feature owns:

``` text
api.ts
queries.ts
mutations.ts
keys.ts
types.ts
```

Example:

``` text
features/work-orders/
├── api/
│   ├── workOrderApi.ts
│   ├── workOrderQueries.ts
│   └── workOrderMutations.ts
├── components/
├── hooks/
├── pages/
├── schemas/
├── types/
└── utils/
```

------------------------------------------------------------------------

# 10. QUERY KEY STANDARD

Examples:

``` ts
customerKeys
vehicleKeys
workOrderKeys
serviceKeys
partKeys
inventoryKeys
invoiceKeys
paymentKeys
reportKeys
```

Never create arbitrary query keys inside components.

------------------------------------------------------------------------

# 11. MUTATION STANDARD

Mutation flow:

``` text
User Action
 ↓
Form validation
 ↓
Mutation
 ↓
API
 ↓
Success
 ↓
Invalidate relevant queries
 ↓
Update navigation/status
 ↓
Toast
```

Example payment:

``` text
payment mutation success
 ↓
invalidate invoice detail
invalidate invoice list
invalidate payment history
invalidate WO detail
invalidate dashboard
invalidate revenue report
```

------------------------------------------------------------------------

# 12. FORM ARCHITECTURE

Use:

``` text
React Hook Form
+
Zod
```

Rules:

-   client validation for UX
-   server validation remains authoritative
-   map API field errors back to form
-   disable submit during mutation
-   prevent accidental duplicate submit

Never rely on frontend validation alone.

------------------------------------------------------------------------

# 13. GLOBAL UI STATES

Every data page must support:

## Loading

Skeleton preferred for content pages.

## Empty

Explain what user should do next.

## Error

Show retry.

## Forbidden

``` text
Anda tidak memiliki akses ke halaman ini.
```

## Not Found

``` text
Data tidak ditemukan.
```

## Mutation Success

Use toast + state refresh.

------------------------------------------------------------------------

# 14. DASHBOARD INTEGRATION

Dashboard should consume:

``` http
GET /api/v1/dashboard/summary
```

Widgets:

``` text
Today's Work Orders
Today's Revenue
Today's Collection
Outstanding
Vehicles Served
Low Stock
```

Role-specific:

### OWNER

``` text
Revenue
Collection
Outstanding
WO Volume
Top Services
Top Mechanics
```

### ADMIN

``` text
WO Today
Ready for Invoice
Unpaid
Payment Collection
```

### MECHANIC

``` text
My New WO
In Progress
QC
Completed Today
```

### WAREHOUSE

``` text
Low Stock
Pending Issue
Receiving
Stock Value
```

------------------------------------------------------------------------

# 15. CUSTOMER INTEGRATION

Customer list:

``` text
GET /customers
```

Features:

-   search
-   pagination
-   filter
-   create
-   edit
-   detail
-   vehicle count
-   service history

Quick action:

``` text
+ Customer
```

------------------------------------------------------------------------

# 16. VEHICLE INTEGRATION

Vehicle must always connect to customer.

Create flow:

``` text
Select Customer
 ↓
Plate Number
 ↓
Brand
 ↓
Model
 ↓
Year
 ↓
VIN/Frame optional
 ↓
Save
```

Duplicate plate handling must use backend response.

Vehicle detail:

``` text
Vehicle Info
Service History
Open Work Orders
Last Service
Mileage
```

------------------------------------------------------------------------

# 17. WORK ORDER LIST

Important columns:

``` text
WO Number
Date
Customer
Vehicle
Plate
Mechanic
Status
Total
Actions
```

Filters:

``` text
Status
Date
Mechanic
Plate
Customer
WO Number
```

Mobile card:

``` text
WO-202609-00127
Honda Beat • F 1234 ABC
Budi
IN PROGRESS
Rp 450.000
```

------------------------------------------------------------------------

# 18. CREATE WORK ORDER

Fast flow:

``` text
Customer
 ↓
Vehicle
 ↓
Complaint
 ↓
Mileage
 ↓
Mechanic
 ↓
Create
```

Support:

``` text
Existing Customer
Existing Vehicle
Quick Create Customer
Quick Create Vehicle
```

After creation:

``` text
→ Work Order Detail
```

------------------------------------------------------------------------

# 19. WORK ORDER DETAIL

This is the most important frontend screen.

Header:

``` text
WO-202609-00127
IN PROGRESS
Customer
Vehicle
Mechanic
```

Tabs:

``` text
Overview
Inspection
Services
Parts
Recommendations
Timeline
```

Sticky action area:

``` text
[Next Action]
```

Action changes by status.

Examples:

``` text
NEW
[Start Inspection]

WAITING_APPROVAL
[Approve]

APPROVED
[Start Work]

IN_PROGRESS
[Send to QC]

QC
[Pass QC]

REWORK
[Resume Work]

READY
[Create Invoice]

INVOICED
[Receive Payment]

PAID
[Complete]
```

Never show irrelevant actions.

------------------------------------------------------------------------

# 20. INSPECTION UI

Mechanic mobile screen:

``` text
┌────────────────────────────┐
│ Inspection                 │
├────────────────────────────┤
│ Engine Oil       ✓ OK      │
│ Brake Front      ⚠ Check   │
│ Brake Rear       ✓ OK      │
│ Tire Front       ⚠ Replace │
│ Tire Rear        ✓ OK      │
│ Battery          ✓ OK      │
├────────────────────────────┤
│ Notes                      │
│ [____________________]     │
├────────────────────────────┤
│ [Save Inspection]          │
└────────────────────────────┘
```

Statuses:

``` text
OK
CHECK
REPLACE
NA
```

Allow recommendation creation from inspection.

------------------------------------------------------------------------

# 21. SERVICE SELECTION

Use searchable combobox.

Display:

``` text
Service Name
Code
Price
Category
```

After select:

``` text
Quantity
Unit Price
Discount
```

Price is loaded from master but server remains authoritative.

------------------------------------------------------------------------

# 22. PART SELECTION

Display:

``` text
Part Name
SKU
Available Stock
Selling Price
Warehouse
```

If insufficient stock:

``` text
Stok tidak mencukupi.
```

Do not allow issue without backend validation.

Part line should clearly distinguish:

``` text
Reserved/Estimated
Issued/Consumed
Returned
```

------------------------------------------------------------------------

# 23. RECOMMENDATIONS

Mechanic can create:

``` text
Recommendation
Reason
Priority
Estimated Cost
```

Statuses:

``` text
PENDING
APPROVED
DECLINED
CONVERTED
```

Admin/customer approval flow must be visually clear.

------------------------------------------------------------------------

# 24. WORKSHOP BOARD

Desktop Kanban:

``` text
NEW
CHECKING
WAITING APPROVAL
APPROVED
IN PROGRESS
QC
REWORK
READY
```

Each card:

``` text
WO
Customer
Plate
Mechanic
Age
Amount
```

Do not rely on drag-and-drop for state transitions unless backend
transition API is explicitly invoked.

Recommended V1:

``` text
Open WO
→ execute transition action
```

------------------------------------------------------------------------

# 25. QC SCREEN

QC should show:

``` text
WO
Customer
Vehicle
Services
Parts
Inspection findings
Mechanic notes
```

Checklist:

``` text
Engine
Brake
Electrical
Tire
Test Ride
Final Cleanliness
```

Actions:

``` text
PASS
REWORK
```

If REWORK:

``` text
Reason required
```

------------------------------------------------------------------------

# 26. INVENTORY INTEGRATION

Inventory screens must distinguish:

``` text
Stock Balance
Stock Movement
Stock Issue
Stock Return
Stock Adjustment
Stock Opname
Low Stock
```

Stock balance is read-only projection.

Mutations must use inventory API.

------------------------------------------------------------------------

# 27. PART ISSUE UX

From WO Parts:

``` text
Part
Qty
Warehouse
Available
```

CTA:

``` text
Issue Part
```

After success:

``` text
Issued Qty
Remaining Stock
Movement Number
```

Never reduce stock locally.

Refetch/invalidate inventory and WO queries.

------------------------------------------------------------------------

# 28. PURCHASE / RECEIVING

Receiving workflow:

``` text
Purchase Order
 ↓
Receiving
 ↓
Confirm Received Qty
 ↓
Stock Movement IN
 ↓
Stock Updated
```

Frontend should show expected vs received.

------------------------------------------------------------------------

# 29. INVOICE INTEGRATION

From READY WO:

``` text
[Create Invoice]
```

Then:

``` text
Invoice Draft
 ↓
Review
 ↓
Issue
```

Invoice screen displays:

``` text
Customer
Vehicle
WO
Services
Parts
Discount
Tax
Grand Total
```

Financial totals come from server.

------------------------------------------------------------------------

# 30. PAYMENT INTEGRATION

Payment CTA only for:

``` text
ISSUED
PARTIAL
```

Payment modal:

``` text
Grand Total
Paid
Outstanding

Amount

Cash
Transfer
QRIS
Debit Card
Credit Card
Other

Reference
Notes
```

Confirm:

``` text
[Confirm Payment]
```

After success:

``` text
Payment Successful
Invoice PAID/PARTIAL
WO status updated
```

Offer:

``` text
[Print Receipt]
```

------------------------------------------------------------------------

# 31. PAYMENT SAFETY UX

During submission:

``` text
button disabled
loading indicator
```

After timeout:

Do not automatically resubmit payment.

Instead:

``` text
Periksa status pembayaran
```

Use idempotency key.

This prevents duplicate financial transactions.

------------------------------------------------------------------------

# 32. INVOICE STATUS UI

Status badges:

``` text
DRAFT
ISSUED
PARTIAL
PAID
VOID
```

Use semantic design-system tokens.

Never represent status only by color.

Always include text.

------------------------------------------------------------------------

# 33. SERVICE HISTORY

Vehicle history should show:

``` text
Date
WO
Services
Parts
Mechanic
Total
Mileage
```

Click:

``` text
View WO
```

History is derived from actual transactions.

Do not create duplicate manual history records.

------------------------------------------------------------------------

# 34. REPORTING FRONTEND

Reports:

``` text
Revenue
Work Orders
Services
Parts
Mechanics
Stock
```

Global filter:

``` text
Date From
Date To
```

Optional:

``` text
Mechanic
Service
Category
Payment Method
Warehouse
```

Export:

``` text
CSV
```

PDF export can follow later if backend supports it.

------------------------------------------------------------------------

# 35. REVENUE REPORT

Separate:

``` text
Invoice Revenue
Cash Collection
Outstanding
```

Example dashboard:

``` text
Revenue
Rp 125.000.000

Collection
Rp 108.500.000

Outstanding
Rp 16.500.000
```

Do not label collection as revenue.

------------------------------------------------------------------------

# 36. DESIGN SYSTEM INTEGRATION

Use existing GARAGE PRO Design System.

Core components:

``` text
Button
Input
Select
Combobox
DatePicker
Modal
Drawer
Card
Table
Badge
Tabs
Toast
Alert
Dropdown
Pagination
Skeleton
EmptyState
ErrorState
ConfirmDialog
```

No page should create ad-hoc versions of these components.

------------------------------------------------------------------------

# 37. COMPONENT LAYERS

Recommended:

``` text
components/
├── ui/
├── layout/
├── forms/
├── data-display/
├── feedback/
└── domain/
```

Domain components may include:

``` text
WorkOrderStatusBadge
InvoiceStatusBadge
PaymentMethodSelector
StockStatusBadge
CustomerCombobox
VehicleCombobox
ServiceCombobox
PartCombobox
```

------------------------------------------------------------------------

# 38. MOBILE-FIRST MECHANIC UX

Mechanic should be able to operate with one hand.

Requirements:

-   large touch target
-   bottom CTA
-   minimal typing
-   search first
-   checklist UI
-   sticky action bar
-   status always visible
-   no dense desktop table

Priority:

``` text
My Work Orders
 ↓
WO Detail
 ↓
Inspection
 ↓
Services
 ↓
Parts
 ↓
Recommendation
 ↓
QC
```

------------------------------------------------------------------------

# 39. DESKTOP ADMIN UX

Admin requires higher information density:

-   table
-   filters
-   multi-column layouts
-   keyboard-friendly inputs
-   quick actions
-   bulk operations where safe

Primary workflow:

``` text
Customer
WO
Invoice
Payment
```

------------------------------------------------------------------------

# 40. WAREHOUSE UX

Warehouse desktop/mobile hybrid:

``` text
Stock Search
 ↓
Select Part
 ↓
Select Warehouse
 ↓
Issue / Receive / Adjust
```

Show:

``` text
Available
Minimum Stock
Location
Last Movement
```

------------------------------------------------------------------------

# 41. OWNER UX

Owner should not need operational detail to see business condition.

Dashboard:

``` text
Revenue
Collection
Outstanding
WO
Average Ticket
Top Services
Top Mechanics
Low Stock
```

Clicking KPI should navigate to filtered report.

------------------------------------------------------------------------

# 42. PERMISSION GATING

Use:

``` tsx
<PermissionGate permission="payment.create">
  <ReceivePaymentButton />
</PermissionGate>
```

Also enforce route-level protection.

Important:

Frontend permission gate is UX protection.

Backend RBAC remains security authority.

------------------------------------------------------------------------

# 43. ROLE-BASED NAVIGATION

OWNER:

``` text
Dashboard
Work Orders
Customers
Vehicles
Invoices
Reports
Users
Settings
```

ADMIN:

``` text
Dashboard
Work Orders
Customers
Vehicles
Invoices
Payments
Reports
```

MECHANIC:

``` text
Dashboard
My Work Orders
Customers
Vehicles
```

WAREHOUSE:

``` text
Dashboard
Parts
Inventory
Purchasing
```

------------------------------------------------------------------------

# 44. GLOBAL SEARCH

Desktop header search should support:

``` text
WO number
Invoice number
Customer name
Phone
Plate number
Part SKU
```

Results grouped:

``` text
Customers
Vehicles
Work Orders
Invoices
Parts
```

Search should route to the correct detail page.

------------------------------------------------------------------------

# 45. NOTIFICATIONS

V1 notification center may expose:

``` text
Low Stock
WO waiting approval
WO ready for QC
WO ready for invoice
Unpaid invoices
```

Notifications should link to actionable screen.

------------------------------------------------------------------------

# 46. OPTIMISTIC UI POLICY

Do NOT optimistically update financial or inventory source-of-truth
data.

Avoid optimistic updates for:

``` text
Payment
Stock issue
Stock adjustment
Invoice status
WO transition
```

Use server response + refetch.

Optimistic UI is acceptable for low-risk preferences such as local UI
state.

------------------------------------------------------------------------

# 47. CACHE INVALIDATION MAP

## Customer mutation

Invalidate:

``` text
customer list
customer detail
search
```

## Vehicle mutation

Invalidate:

``` text
vehicle list
vehicle detail
customer detail
```

## WO mutation

Invalidate:

``` text
WO list
WO detail
board
vehicle history
dashboard
```

## Inventory mutation

Invalidate:

``` text
stock
movement
low stock
WO parts
```

## Invoice mutation

Invalidate:

``` text
invoice
WO
dashboard
revenue
```

## Payment mutation

Invalidate:

``` text
payment
invoice
WO
dashboard
revenue
outstanding
```

------------------------------------------------------------------------

# 48. URL STATE

Filters should be encoded in URL where useful.

Example:

``` text
/work-orders?status=IN_PROGRESS&mechanic=12
```

Benefits:

-   shareable
-   browser back works
-   refresh safe
-   report navigation

Do not put sensitive data into URL.

------------------------------------------------------------------------

# 49. TABLE UX

Tables require:

-   loading skeleton
-   empty state
-   pagination
-   sortable columns where supported
-   row actions
-   responsive behavior

Mobile:

Convert table into cards when necessary.

Do not force horizontal scrolling for every screen.

------------------------------------------------------------------------

# 50. FORM UX

Rules:

``` text
Required fields marked *
Inline validation
Submit disabled during mutation
Unsaved changes warning where necessary
Success feedback
Error preserved
```

For destructive actions:

``` text
ConfirmDialog
```

Reason required for:

``` text
Void
Refund
Rework
Adjustment
```

------------------------------------------------------------------------

# 51. DATE/TIME

Frontend should display workshop-local time.

Recommended:

``` text
Asia/Jakarta
```

API should use ISO 8601 timestamps.

Date display:

``` text
14 Sep 2026
```

Date/time:

``` text
14 Sep 2026, 18:45
```

Do not manipulate timezone manually in multiple components.

Create one date/time utility.

------------------------------------------------------------------------

# 52. MONEY DISPLAY

Create:

``` ts
formatCurrency()
```

Example:

``` text
Rp 1.250.000
```

Input may display formatted currency while form stores normalized value.

API monetary strings must remain precise.

------------------------------------------------------------------------

# 53. STATUS HELPERS

Create centralized:

``` ts
getWorkOrderStatusMeta()
getInvoiceStatusMeta()
getPaymentStatusMeta()
getStockStatusMeta()
```

Each returns:

``` ts
{
  label,
  description,
  semantic,
  icon
}
```

This prevents inconsistent status presentation.

------------------------------------------------------------------------

# 54. ROUTE GUARDS

Structure:

``` tsx
<ProtectedRoute>
  <PermissionRoute permission="invoice.view">
    <InvoicePage />
  </PermissionRoute>
</ProtectedRoute>
```

Role is not enough.

Use permission-level checks.

------------------------------------------------------------------------

# 55. UNSAVED CHANGES

Forms such as:

``` text
Customer
Vehicle
Service
Part
Inspection
WO
```

should warn before leaving when dirty.

Do not interrupt simple search/filter screens.

------------------------------------------------------------------------

# 56. ACCESSIBILITY

Target:

``` text
WCAG 2.1 AA
```

Requirements:

-   keyboard navigation
-   visible focus
-   semantic HTML
-   labels
-   aria where necessary
-   contrast
-   non-color status indication
-   touch target minimum approximately 44px

------------------------------------------------------------------------

# 57. PWA

Prepare:

``` text
manifest
service worker
offline shell
installability
```

V1 offline behavior:

``` text
App shell available
Read-only cached data where safe
```

Do NOT allow offline financial mutations in V1.

No offline:

``` text
Payment
Stock adjustment
Stock issue
Invoice issue
WO transition
```

unless a future offline transaction queue is implemented.

------------------------------------------------------------------------

# 58. FRONTEND PROJECT STRUCTURE

Recommended:

``` text
frontend/src/
├── app/
│   ├── App.tsx
│   ├── router.tsx
│   ├── providers/
│   └── guards/
│
├── components/
│   ├── ui/
│   ├── layout/
│   ├── forms/
│   ├── feedback/
│   └── data-display/
│
├── features/
│   ├── auth/
│   ├── dashboard/
│   ├── customers/
│   ├── vehicles/
│   ├── mechanics/
│   ├── work-orders/
│   ├── services/
│   ├── parts/
│   ├── inventory/
│   ├── purchasing/
│   ├── invoices/
│   ├── payments/
│   └── reports/
│
├── lib/
│   ├── api/
│   ├── auth/
│   ├── query/
│   ├── format/
│   ├── date/
│   └── permissions/
│
├── hooks/
├── styles/
├── types/
└── main.tsx
```

------------------------------------------------------------------------

# 59. FRONTEND ENVIRONMENT

Example:

``` env
VITE_API_BASE_URL=https://api.example.com/api/v1
VITE_APP_NAME=GARAGE PRO
```

Development:

``` env
VITE_API_BASE_URL=http://localhost:3000/api/v1
```

Never hard-code production API URLs inside components.

------------------------------------------------------------------------

# 60. FRONTEND INTEGRATION SEQUENCE

Implement in this order:

``` text
01 App shell
02 Auth integration
03 Route guards
04 Global UI components
05 API client
06 Query provider
07 Dashboard
08 Customer
09 Vehicle
10 Work Order
11 Inspection
12 Service
13 Parts
14 Inventory
15 Purchasing
16 Invoice
17 Payment
18 Reports
19 Settings
20 Audit
```

------------------------------------------------------------------------

# 61. END-TO-END DEMO SCENARIO

Create test/demo data:

``` text
Customer:
Budi Santoso

Vehicle:
Honda Beat
Plate:
F 1234 ABC

Mechanic:
Andi

Service:
Ganti Oli

Part:
Oli Mesin

Warehouse:
Gudang Utama
```

Flow:

``` text
Login as ADMIN
 ↓
Create/open customer
 ↓
Create vehicle
 ↓
Create WO
 ↓
Assign mechanic
 ↓
Inspection
 ↓
Add service
 ↓
Add part
 ↓
Issue part
 ↓
Start work
 ↓
QC
 ↓
READY
 ↓
Create invoice
 ↓
Issue invoice
 ↓
Receive payment
 ↓
Invoice PAID
 ↓
WO COMPLETED
 ↓
View vehicle history
 ↓
View revenue report
```

This scenario must work without manually editing the database.

------------------------------------------------------------------------

# 62. E2E TEST SCENARIO

Recommended Playwright/Cypress flow:

``` text
login
create customer
create vehicle
create work order
add inspection
add service
add part
issue part
approve
start
QC pass
create invoice
issue invoice
payment
verify PAID
verify WO COMPLETED
verify history
verify report
```

Assertions:

``` text
URL correct
status correct
totals correct
stock reduced
invoice generated
payment recorded
receipt visible
report updated
```

------------------------------------------------------------------------

# 63. ERROR RECOVERY

If mutation fails:

``` text
Keep user on current screen
Show actionable error
Do not clear form
Do not pretend success
```

If network timeout occurs during payment:

``` text
Do not automatically retry POST
Query payment/invoice status
```

If stale WO:

``` text
Refetch detail
Explain:
"Data Work Order berubah. Halaman telah diperbarui."
```

------------------------------------------------------------------------

# 64. FRONTEND SECURITY

Never trust:

``` text
role from localStorage
permissions from UI
price from browser
total from browser
stock from browser
```

Frontend values are for display only.

Backend validates every mutation.

Avoid storing sensitive access tokens in insecure long-lived local
storage where the existing auth architecture provides a safer mechanism.

------------------------------------------------------------------------

# 65. PERFORMANCE

Use:

``` text
code splitting
lazy routes
pagination
debounced search
query caching
skeletons
virtualization only when necessary
```

Do not load:

``` text
all customers
all vehicles
all parts
all invoices
```

on application startup.

------------------------------------------------------------------------

# 66. OBSERVABILITY

Frontend should attach:

``` text
requestId
```

to errors where available.

Error reporting should capture:

``` text
route
user role
action
API endpoint
error code
requestId
timestamp
```

Never capture passwords/payment secrets.

------------------------------------------------------------------------

# 67. TESTING LAYERS

## Unit

-   formatters
-   status helpers
-   validation schemas
-   UI utility functions

## Component

-   forms
-   modals
-   status badges
-   payment modal
-   inspection checklist

## Integration

-   query/mutation hooks
-   route guards
-   permission gates

## E2E

Full workshop flow.

------------------------------------------------------------------------

# 68. FRONTEND ACCEPTANCE CRITERIA

### Application

-   [ ] Login works.
-   [ ] Logout works.
-   [ ] Role/permission routes work.
-   [ ] Navigation is role-aware.
-   [ ] Responsive shell works.

### Customer

-   [ ] CRUD works.
-   [ ] Search works.
-   [ ] Vehicle relation works.

### Vehicle

-   [ ] CRUD works.
-   [ ] History works.

### Work Order

-   [ ] Create.
-   [ ] Detail.
-   [ ] Inspection.
-   [ ] Services.
-   [ ] Parts.
-   [ ] Recommendations.
-   [ ] State transitions.
-   [ ] Board.
-   [ ] QC.

### Inventory

-   [ ] Stock overview.
-   [ ] Issue.
-   [ ] Return.
-   [ ] Adjustment.
-   [ ] Opname.
-   [ ] Low stock.

### Invoice

-   [ ] Create from READY.
-   [ ] Review.
-   [ ] Issue.
-   [ ] View.
-   [ ] Void where permitted.

### Payment

-   [ ] Full payment.
-   [ ] Partial payment.
-   [ ] Multiple payments.
-   [ ] Method selection.
-   [ ] Receipt.
-   [ ] Error handling.
-   [ ] Duplicate submit protection.

### Reports

-   [ ] Revenue.
-   [ ] Collection.
-   [ ] Outstanding.
-   [ ] WO.
-   [ ] Service.
-   [ ] Parts.
-   [ ] Mechanics.
-   [ ] Stock.

------------------------------------------------------------------------

# 69. DEFINITION OF DONE

Phase 15 is complete only when:

``` text
Frontend builds successfully
        ↓
Backend API integration works
        ↓
All protected routes work
        ↓
All major CRUD workflows work
        ↓
WO lifecycle works
        ↓
Inventory integration works
        ↓
Invoice integration works
        ↓
Payment integration works
        ↓
Reports show data
        ↓
E2E scenario passes
        ↓
Mobile UX passes
        ↓
Desktop UX passes
```

Commands:

``` bash
npm run build
npm run lint
npm run test
npm run test:e2e
```

No critical errors in browser console.

------------------------------------------------------------------------

# 70. CLAUDE CODE MASTER PROMPT

``` text
You are implementing Phase 15 — FRONTEND INTEGRATION V1 of GARAGE PRO.

IMPORTANT:
This is an existing project.
Do NOT rebuild the project from scratch.
First inspect the existing frontend, backend API contracts, authentication, RBAC, database/domain services, Work Order Engine, Inventory Engine, and Invoice + Payment Engine.

GOAL:
Turn the existing domain engines into one cohesive production-ready frontend.

STACK:
- React
- Vite
- TypeScript
- Tailwind CSS
- React Router
- TanStack Query
- React Hook Form
- Zod
- Axios
- Existing GARAGE PRO Design System

CORE FLOW:

LOGIN
→ DASHBOARD
→ CUSTOMER
→ VEHICLE
→ WORK ORDER
→ INSPECTION
→ SERVICE
→ PART
→ APPROVAL
→ WORK
→ QC
→ READY
→ INVOICE
→ PAYMENT
→ COMPLETED
→ HISTORY
→ REPORTS

RULES:

1. READ EXISTING CODE FIRST.
2. Reuse existing components.
3. Reuse existing API contracts.
4. Reuse existing auth/RBAC.
5. Reuse existing Work Order state machine.
6. Reuse existing Inventory Engine.
7. Reuse existing Invoice + Payment Engine.
8. Do not duplicate business logic in frontend.
9. Do not calculate financial source-of-truth totals in frontend.
10. Do not mutate stock locally.
11. Do not mutate WO status directly.
12. Do not use optimistic updates for financial/inventory mutations.
13. Use TanStack Query.
14. Use React Hook Form + Zod.
15. Use centralized Axios client.
16. Use role and permission guards.
17. Use the existing design system.
18. Keep mobile mechanic UX simple.
19. Keep admin desktop UX information-dense.
20. Never bypass backend authorization.

IMPLEMENT IN THIS ORDER:

1. Inspect project
2. Verify app bootstrap
3. Verify API client
4. Verify AuthProvider
5. Verify route guards
6. Verify design system
7. Build application shell
8. Build role-aware navigation
9. Integrate dashboard
10. Integrate customers
11. Integrate vehicles
12. Integrate work orders
13. Integrate inspection
14. Integrate services
15. Integrate parts
16. Integrate inventory
17. Integrate purchasing
18. Integrate invoices
19. Integrate payments
20. Integrate reports
21. Integrate settings
22. Integrate audit
23. Add loading/error/empty states
24. Add responsive behavior
25. Add E2E tests
26. Build/lint/test

ROUTES:

/login
/app/dashboard
/app/work-orders
/app/work-orders/new
/app/work-orders/:id
/app/work-orders/:id/inspection
/app/work-orders/:id/services
/app/work-orders/:id/parts
/app/work-orders/:id/recommendations
/app/work-orders/:id/qc
/app/customers
/app/customers/new
/app/customers/:id
/app/customers/:id/edit
/app/vehicles
/app/vehicles/:id
/app/vehicles/:id/history
/app/services
/app/parts
/app/inventory
/app/purchasing
/app/invoices
/app/invoices/:id
/app/payments
/app/payments/:id
/app/reports/*
/app/settings/*
/app/audit

WORK ORDER UI:

Show contextual next action.

NEW:
Start Inspection

WAITING_APPROVAL:
Approve

APPROVED:
Start Work

IN_PROGRESS:
Send to QC

QC:
Pass / Rework

REWORK:
Resume Work

READY:
Create Invoice

ISSUED/PARTIAL INVOICE:
Receive Payment

PAID:
Complete

Do not expose invalid transitions.

PAYMENT UI:

Show:
- Grand total
- Paid
- Outstanding
- Amount
- Payment method
- Reference
- Notes

Use Idempotency-Key.

On payment timeout:
DO NOT automatically resubmit.
Refresh/query payment status.

INVENTORY UI:

Stock is server truth.
After issue/return/adjustment:
invalidate relevant queries.

REPORTING:

Keep separate:
- invoice revenue
- cash collection
- outstanding

MOBILE:

Mechanic:
- large touch targets
- sticky bottom CTA
- checklist
- minimal typing

ADMIN:

- tables
- filters
- quick actions
- dense information

SECURITY:

Frontend permission gates are UX only.
Backend remains authority.

Do not expose:
- secrets
- tokens
- passwords
- card credentials

TEST:

Implement E2E:
login
customer
vehicle
WO
inspection
service
part
issue
approval
start
QC
READY
invoice
issue
payment
PAID
COMPLETED
history
report

FINAL VALIDATION:

Run:
npm run build
npm run lint
npm run test
npm run test:e2e

Then report:
- files changed
- routes added
- components added
- API hooks added
- tests added
- build result
- lint result
- test result
- unresolved issues

Do not claim success if a command failed.
```

------------------------------------------------------------------------

# 71. IMPLEMENTATION CHECKLIST FOR CLAUDE CODE

``` text
[ ] Inspect existing repo
[ ] Confirm API base URL
[ ] Confirm auth contract
[ ] Confirm permission contract
[ ] Confirm query conventions
[ ] Confirm design system
[ ] App shell
[ ] Sidebar
[ ] Header
[ ] Mobile navigation
[ ] Dashboard
[ ] Customer
[ ] Vehicle
[ ] Work Order
[ ] Inspection
[ ] Services
[ ] Parts
[ ] Inventory
[ ] Purchasing
[ ] Invoice
[ ] Payment
[ ] Receipt
[ ] Reports
[ ] Settings
[ ] Audit
[ ] Error states
[ ] Loading states
[ ] Permission states
[ ] Mobile responsive
[ ] E2E
[ ] Build
[ ] Lint
[ ] Test
```

------------------------------------------------------------------------

# 72. INTEGRATION ARCHITECTURE

``` text
                    ┌─────────────────────┐
                    │      USER           │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │   GARAGE PRO UI     │
                    │ React + Tailwind    │
                    └──────────┬──────────┘
                               │
                    ┌──────────▼──────────┐
                    │ TanStack Query      │
                    │ Server State        │
                    └──────────┬──────────┘
                               │
                    ┌──────────▼──────────┐
                    │ Axios API Client    │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Express REST API    │
                    └──────────┬──────────┘
                               │
              ┌────────────────┼─────────────────┐
              ▼                ▼                 ▼
        Work Order        Inventory         Finance
          Engine            Engine       Invoice/Payment
              │                │                 │
              └────────────────┼─────────────────┘
                               ▼
                         MySQL Database
```

------------------------------------------------------------------------

# 73. FINAL END-TO-END TARGET

The finished GARAGE PRO application must allow a real workshop to
execute:

``` text
CUSTOMER ARRIVES
       ↓
ADMIN FINDS / CREATES CUSTOMER
       ↓
SELECT / CREATE MOTORCYCLE
       ↓
CREATE WORK ORDER
       ↓
MECHANIC INSPECTS
       ↓
MECHANIC ADDS SERVICE
       ↓
MECHANIC ADDS SPARE PART
       ↓
PART IS ISSUED FROM STOCK
       ↓
ADDITIONAL WORK IF NEEDED
       ↓
CUSTOMER APPROVAL
       ↓
MECHANIC WORKS
       ↓
QC
       ↓
READY
       ↓
ADMIN CREATES INVOICE
       ↓
INVOICE ISSUED
       ↓
CUSTOMER PAYS
       ↓
PAYMENT RECORDED
       ↓
INVOICE PAID
       ↓
WO COMPLETED
       ↓
RECEIPT
       ↓
VEHICLE SERVICE HISTORY
       ↓
REVENUE / COLLECTION REPORT
```

This is the primary acceptance journey for GARAGE PRO V1.

------------------------------------------------------------------------

# 74. NEXT PHASE

After Frontend Integration:

``` text
14 ✅ Invoice + Payment Engine
15 ✅ Frontend Integration Specification
16 🔜 Testing / QA
17    Deployment
```

Phase 16 should focus on systematic QA:

``` text
Unit Testing
Integration Testing
API Testing
E2E Testing
Security Testing
Concurrency Testing
Inventory Integrity
Financial Integrity
Responsive Testing
UAT
```

The objective is to prove that the complete GARAGE PRO workflow is
reliable before production deployment.
