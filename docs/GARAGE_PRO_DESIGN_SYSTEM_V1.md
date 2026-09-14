# GARAGE PRO --- DESIGN SYSTEM V1

**Document Type:** UI Design System Specification\
**Product:** GARAGE PRO --- Workshop Management System\
**Version:** V1.0\
**Status:** Development Ready\
**Date:** 14 September 2026

------------------------------------------------------------------------

# 1. DESIGN SYSTEM OBJECTIVE

GARAGE PRO menggunakan visual identity:

> **Modern Automotive SaaS --- Clean, Strong, Fast, Practical**

Design harus terasa: - profesional - modern - terpercaya - cepat
digunakan - tidak terlalu ramai - cocok untuk lingkungan bengkel -
nyaman digunakan di desktop dan smartphone

Inspirasi visual secara prinsip: - modern SaaS dashboard - automotive
service application - enterprise utility software

Bukan: - UI game - dashboard penuh ornament - glassmorphism berlebihan -
gradient berlebihan - interface yang terlalu dekoratif

------------------------------------------------------------------------

# 2. DESIGN PRINCIPLES

## 2.1 Clarity

Informasi paling penting harus paling mudah ditemukan.

Prioritas:

``` text
Status
↓
Primary Action
↓
Important Data
↓
Secondary Data
```

## 2.2 Hierarchy

Gunakan ukuran, weight, spacing dan surface untuk membedakan informasi.

## 2.3 Consistency

Komponen yang sama harus memiliki perilaku yang sama di seluruh
aplikasi.

## 2.4 Density

Desktop boleh padat untuk pekerjaan admin.

Mobile harus lebih lapang dan touch-friendly.

## 2.5 Action Oriented

Setiap halaman harus memiliki satu primary action yang jelas.

------------------------------------------------------------------------

# 3. COLOR SYSTEM

Gunakan semantic tokens.

## 3.1 Primary

Primary color digunakan untuk: - primary button - active navigation -
selected state - important links - focus indicator

Suggested base:

``` text
Primary 500: #2563EB
Primary 600: #1D4ED8
Primary 700: #1E40AF
```

Primary 500 adalah default interactive color.

## 3.2 Neutral

``` text
Neutral 50:  #F8FAFC
Neutral 100: #F1F5F9
Neutral 200: #E2E8F0
Neutral 300: #CBD5E1
Neutral 400: #94A3B8
Neutral 500: #64748B
Neutral 600: #475569
Neutral 700: #334155
Neutral 800: #1E293B
Neutral 900: #0F172A
```

## 3.3 Semantic

Success:

``` text
#16A34A
```

Warning:

``` text
#D97706
```

Danger:

``` text
#DC2626
```

Info:

``` text
#0284C7
```

## 3.4 Surface

Light:

``` text
Background: #F8FAFC
Surface:    #FFFFFF
Elevated:   #FFFFFF
Border:     #E2E8F0
```

Dark mode:

``` text
Background: #0F172A
Surface:    #111827
Elevated:   #1E293B
Border:     #334155
```

------------------------------------------------------------------------

# 4. COLOR SEMANTIC TOKENS

Never hard-code meaning-specific colors inside business components.

Use:

``` text
--color-primary
--color-primary-hover
--color-primary-active

--color-success
--color-success-bg
--color-success-text

--color-warning
--color-warning-bg
--color-warning-text

--color-danger
--color-danger-bg
--color-danger-text

--color-info
--color-info-bg
--color-info-text

--color-background
--color-surface
--color-surface-elevated
--color-border

--color-text
--color-text-secondary
--color-text-muted
```

------------------------------------------------------------------------

# 5. STATUS COLORS

Status must not rely only on color.

Use color + icon/text.

Recommended mapping:

  Status                 Semantic
  ---------------------- ----------
  Baru                   Neutral
  Pemeriksaan            Info
  Estimasi               Neutral
  Menunggu Persetujuan   Warning
  Disetujui              Success
  Dikerjakan             Primary
  QC                     Info
  Rework                 Danger
  Siap Diambil           Success
  Ditagihkan             Warning
  Lunas                  Success
  Selesai                Success
  Dibatalkan             Neutral

------------------------------------------------------------------------

# 6. TYPOGRAPHY

Recommended font:

``` text
Inter
```

Fallback:

``` text
ui-sans-serif, system-ui, sans-serif
```

Font weights:

``` text
400 Regular
500 Medium
600 Semibold
700 Bold
```

Avoid excessive 800/900.

------------------------------------------------------------------------

# 7. TYPE SCALE

``` text
Display:  36px / 44px / 700
H1:       30px / 38px / 700
H2:       24px / 32px / 700
H3:       20px / 28px / 600
H4:       18px / 26px / 600

Body LG:  16px / 24px / 400
Body:     14px / 22px / 400
Body SM:  13px / 20px / 400

Label:    13px / 18px / 500
Caption:  12px / 18px / 400
```

Default application body:

``` text
14px
```

Mobile input text:

``` text
16px
```

This reduces accidental browser zoom on mobile Safari.

------------------------------------------------------------------------

# 8. SPACING SYSTEM

Use 4px base unit.

``` text
0   = 0
1   = 4px
2   = 8px
3   = 12px
4   = 16px
5   = 20px
6   = 24px
8   = 32px
10  = 40px
12  = 48px
16  = 64px
20  = 80px
24  = 96px
```

Common usage:

``` text
Card padding: 20–24px
Form gap: 16px
Section gap: 24–32px
Page padding desktop: 24–32px
Page padding mobile: 16px
```

------------------------------------------------------------------------

# 9. BORDER RADIUS

``` text
sm: 6px
md: 8px
lg: 12px
xl: 16px
full: 9999px
```

Recommended:

``` text
Button: 8px
Input: 8px
Card: 12px
Modal: 16px
Badge: full
Avatar: full
```

------------------------------------------------------------------------

# 10. SHADOW SYSTEM

Keep shadows subtle.

``` text
shadow-sm
shadow-md
shadow-lg
```

Default cards should generally use:

``` text
border + subtle shadow
```

rather than heavy shadow.

------------------------------------------------------------------------

# 11. ICON SYSTEM

Use:

``` text
Lucide React
```

Rules: - default size: 20px - small: 16px - large: 24px - stroke width:
2 - icon-only buttons require tooltip/aria-label

Examples:

``` text
Dashboard       LayoutDashboard
Work Order      ClipboardList
Customer        Users
Vehicle         Bike
Service         Wrench
Parts           Package
Inventory       Warehouse
Purchase        ShoppingCart
Invoice         Receipt
Payment         CreditCard
Reports         BarChart3
Settings        Settings
Search          Search
Add             Plus
Edit            Pencil
Delete          Trash2
```

------------------------------------------------------------------------

# 12. BUTTON SYSTEM

Variants:

``` text
Primary
Secondary
Outline
Ghost
Danger
Link
```

## Primary

Use for the main page action.

Example:

``` text
[ + Work Order Baru ]
```

## Secondary

Use for supporting action.

## Outline

Use for alternative action.

## Ghost

Use for low emphasis action.

## Danger

Only destructive actions.

------------------------------------------------------------------------

# 13. BUTTON SIZES

``` text
sm: 32px
md: 40px
lg: 48px
```

Desktop default:

``` text
40px
```

Mobile primary action:

``` text
44–48px
```

Minimum recommended touch target:

``` text
44 × 44px
```

------------------------------------------------------------------------

# 14. BUTTON STATES

Every button supports:

``` text
default
hover
active
focus
disabled
loading
```

Loading:

``` text
[ ◌ Menyimpan... ]
```

Never allow duplicate submission.

------------------------------------------------------------------------

# 15. INPUT SYSTEM

Input height:

``` text
Desktop: 40px
Mobile: 48px
```

Structure:

``` text
Label
Input
Helper / Error
```

Example:

``` text
Nomor HP *
[ 08123456789          ]

Format nomor HP tidak valid.
```

------------------------------------------------------------------------

# 16. INPUT STATES

``` text
default
hover
focus
disabled
readonly
error
success
```

Focus should have visible outline/ring.

------------------------------------------------------------------------

# 17. SEARCH INPUT

Use prominent search.

Example:

``` text
🔍 Cari WO, customer, plat nomor...
```

Keyboard shortcut:

``` text
Ctrl + K
```

Mobile: - search may occupy full row - clear button visible when text
exists

------------------------------------------------------------------------

# 18. SELECT / COMBOBOX

For small static options: - Select

For searchable data: - Combobox

Use Combobox for: - customer - vehicle - mechanic - service - spare
part - supplier

Do not force users to scroll through hundreds of records.

------------------------------------------------------------------------

# 19. NUMBER INPUT

For: - quantity - KM - price - stock

Support: - keyboard numeric input - min/max - decimal rules where needed

For quantity, provide optional stepper on mobile:

``` text
[ − ]  2  [ + ]
```

------------------------------------------------------------------------

# 20. CURRENCY INPUT

Display:

``` text
Rp 75.000
```

Internally:

``` text
75000
```

Avoid storing formatted strings.

------------------------------------------------------------------------

# 21. TEXTAREA

Default:

``` text
min-height: 96px
```

Used for: - complaint - diagnosis - notes - recommendation reason -
rejection reason

------------------------------------------------------------------------

# 22. CHECKBOX

Use for: - checklist - multi-select - permissions

Touch area should include label.

------------------------------------------------------------------------

# 23. RADIO

Use for mutually exclusive values.

Examples: - payment method - inspection result - adjustment type

------------------------------------------------------------------------

# 24. SWITCH

Use for binary settings:

``` text
Active
Tax Enabled
Notifications
```

Do not use switch for actions that should require confirmation.

------------------------------------------------------------------------

# 25. BADGE

Variants:

``` text
neutral
primary
success
warning
danger
info
```

Sizes:

``` text
sm
md
```

Example:

``` text
[ SIAP DIAMBIL ]
```

------------------------------------------------------------------------

# 26. CARD

Card structure:

``` text
Card
 ├── Header
 ├── Content
 └── Footer
```

Default:

``` text
background: surface
border: 1px
radius: lg
padding: 20–24px
```

Do not put every small UI element in a separate card.

------------------------------------------------------------------------

# 27. STAT CARD

Structure:

``` text
Label
Value
Trend / Context
Icon
```

Example:

``` text
┌───────────────────────┐
│ Omzet Hari Ini     ◈  │
│ Rp 4.850.000          │
│ +12,4% vs kemarin     │
└───────────────────────┘
```

------------------------------------------------------------------------

# 28. TABLE

Desktop: - header 40--44px - row 48--56px - compact where appropriate

Rules: - numeric values right aligned - dates consistent - status
badge - actions right aligned - long text truncated with tooltip

------------------------------------------------------------------------

# 29. DATA TABLE

Required features where applicable:

``` text
Sorting
Filtering
Pagination
Column visibility
Row actions
Loading
Empty
Error
```

Do not add every feature to every table automatically.

------------------------------------------------------------------------

# 30. MOBILE TABLE

Convert to card/list when table becomes unreadable.

Example:

``` text
WO-001
Honda Beat
B 1234 XYZ

Rp 190.000
[ LUNAS ]

14 Sep 2026

[ Lihat ]
```

------------------------------------------------------------------------

# 31. MODAL

Modal sizes:

``` text
sm: 400px
md: 520px
lg: 720px
xl: 960px
```

Use: - simple forms - quick actions - confirmation

Avoid very long forms inside modal.

------------------------------------------------------------------------

# 32. DRAWER

Use for: - detail preview - search selection - timeline - audit log -
contextual information

Desktop: drawer from right.

Mobile: bottom sheet or full-height sheet.

------------------------------------------------------------------------

# 33. CONFIRMATION DIALOG

Destructive actions must explain: - what happens - affected record -
irreversible nature if applicable

Primary button should use danger variant for destructive action.

------------------------------------------------------------------------

# 34. TOAST

Position desktop:

``` text
top-right
```

Mobile:

``` text
top-center
```

Duration:

``` text
Success: 3–4 sec
Info: 4 sec
Warning/Error: 5–7 sec
```

Critical errors may remain until dismissed.

------------------------------------------------------------------------

# 35. ALERT

Use inline alert when information is relevant to the current page.

Examples:

``` text
⚠ Stok spare part berada di bawah minimum.
```

``` text
ℹ WO ini belum mendapat persetujuan customer.
```

------------------------------------------------------------------------

# 36. EMPTY STATE

Components:

``` text
Icon
Title
Description
Primary Action
```

Avoid oversized illustrations.

------------------------------------------------------------------------

# 37. LOADING

Prefer skeleton for page content.

For actions: - button spinner - disabled while request active

For tables: - skeleton rows

------------------------------------------------------------------------

# 38. ERROR STATE

Error page:

``` text
Tidak dapat memuat data

Terjadi masalah saat mengambil data.

[ Coba Lagi ]
```

Technical details should not be exposed by default.

------------------------------------------------------------------------

# 39. PAGINATION

Desktop:

``` text
← Sebelumnya
1 2 3 4
Berikutnya →
```

Also display:

``` text
Menampilkan 1–20 dari 156 data
```

Mobile: - Previous / Next - optional page selector

------------------------------------------------------------------------

# 40. NAVIGATION

Desktop sidebar:

``` text
GARAGE PRO

Dashboard

WORKSHOP
  Work Orders
  Workshop Board

MASTER
  Customers
  Vehicles
  Services
  Spare Parts

INVENTORY
  Stock
  Stock Movement
  Low Stock
  Stock Opname

PURCHASING
  Suppliers
  Purchases
  Receiving

TRANSACTIONS
  Invoices
  Payments

REPORTS

SETTINGS
```

Active menu: - primary background tint - primary text - icon emphasized

------------------------------------------------------------------------

# 41. MOBILE NAVIGATION

Primary bottom navigation:

``` text
Home
Work
Search
More
```

For mechanic:

``` text
Home
Pekerjaan
Riwayat
Profile
```

Other modules accessible through More/menu.

------------------------------------------------------------------------

# 42. PAGE HEADER

Standard:

``` text
Title
Description
Actions
```

Example:

``` text
Work Orders
Kelola seluruh pekerjaan bengkel

[ + Work Order Baru ]
```

Mobile:

``` text
Work Orders
[ + ]
```

Secondary description may collapse.

------------------------------------------------------------------------

# 43. BREADCRUMB

Use on desktop for deep pages.

Example:

``` text
Workshop / Work Orders / WO-20260914-0001
```

Avoid breadcrumb clutter on mobile.

------------------------------------------------------------------------

# 44. TABS

Use for related information.

Example:

``` text
Overview | Inspection | Services | Parts | Timeline
```

Mobile: - horizontal scroll - active tab clearly indicated

------------------------------------------------------------------------

# 45. TIMELINE

Used for: - WO history - service history - audit log

Example:

``` text
14:30
Pembayaran diterima
Rp190.000

14:10
Invoice dibuat

13:30
QC selesai

12:20
Pekerjaan selesai
```

------------------------------------------------------------------------

# 46. KANBAN

Used for Workshop Board.

Column: - title - count - cards

Card: - WO number - vehicle - customer - mechanic - key service - status

Drag-and-drop is optional.

V1 should prioritize explicit status transitions rather than
unrestricted drag-and-drop.

------------------------------------------------------------------------

# 47. DASHBOARD CHARTS

Chart types:

``` text
Revenue → Line / Area
WO volume → Bar
Service mix → Bar
Payment mix → Donut
Stock status → Bar
```

Rules: - avoid 3D charts - avoid excessive colors - always show
values/context - responsive - accessible labels

------------------------------------------------------------------------

# 48. DATA VISUALIZATION

Charts should answer a business question.

Bad:

``` text
Decorative chart
```

Good:

``` text
Revenue 7 Hari Terakhir
```

Good:

``` text
Top 10 Jasa
```

Good:

``` text
Spare Part di Bawah Minimum
```

------------------------------------------------------------------------

# 49. AVATAR

Use: - initials - profile image if available

Example:

``` text
AY
```

Sizes:

``` text
24
32
40
48
```

------------------------------------------------------------------------

# 50. TOOLTIP

Use only when: - icon meaning is not obvious - truncated data needs
explanation - keyboard shortcut

Do not use tooltips as the only way to expose critical information.

------------------------------------------------------------------------

# 51. DATE / TIME

Display Indonesian format:

``` text
14 Sep 2026
14 Sep 2026, 14:30
```

Full date where needed:

``` text
14 September 2026
```

Timezone:

``` text
Asia/Jakarta
```

------------------------------------------------------------------------

# 52. MOBILE TOUCH DESIGN

Minimum touch target:

``` text
44 × 44px
```

For mechanic critical controls:

``` text
48–56px
```

Avoid controls closer than 8px.

------------------------------------------------------------------------

# 53. MECHANIC UI SPECIAL RULES

Mechanic UI should prioritize:

``` text
WO
Vehicle
Complaint
Inspection
Services
Parts
Recommendation
QC
```

Avoid: - complex reports - accounting controls - admin settings - dense
tables

------------------------------------------------------------------------

# 54. MECHANIC ACTION BAR

For mobile:

``` text
┌────────────────────────────┐
│ [ Simpan ] [ Lanjutkan ]  │
└────────────────────────────┘
```

Sticky at bottom where appropriate.

Must not cover content.

Add safe-area padding on supported devices.

------------------------------------------------------------------------

# 55. ADMIN UI SPECIAL RULES

Admin UI prioritizes: - tables - filters - search - bulk actions where
safe - keyboard workflow - quick create

Desktop page max content width:

``` text
1440px
```

Use full available width for tables.

------------------------------------------------------------------------

# 56. OWNER UI SPECIAL RULES

Owner UI prioritizes: - KPI - trend - exceptions - revenue - workshop
performance - low stock

Owner should be able to understand workshop condition in under 30
seconds.

------------------------------------------------------------------------

# 57. WAREHOUSE UI SPECIAL RULES

Warehouse UI prioritizes: - stock - receiving - issue - movement -
opname - low stock

Show stock quantity prominently.

------------------------------------------------------------------------

# 58. DARK MODE

Dark mode is supported.

Do not simply invert colors.

Use semantic surface tokens.

Avoid pure black:

``` text
#000000
```

Preferred background:

``` text
#0F172A
```

Cards:

``` text
#111827 / #1E293B
```

Text remains high contrast.

------------------------------------------------------------------------

# 59. RESPONSIVE GRID

Desktop:

``` text
4-column KPI
12-column content grid
```

Tablet:

``` text
2-column KPI
```

Mobile:

``` text
1-column
```

Example:

``` text
grid-cols-1
sm:grid-cols-2
lg:grid-cols-4
```

------------------------------------------------------------------------

# 60. PAGE CONTAINER

Recommended:

``` text
width: 100%
max-width: 1440px
margin: auto
padding-inline: 24px
```

Mobile:

``` text
padding-inline: 16px
```

------------------------------------------------------------------------

# 61. DESIGN TOKENS --- CSS

Recommended token structure:

``` css
:root {
  --color-primary: #2563EB;
  --color-primary-hover: #1D4ED8;
  --color-primary-active: #1E40AF;

  --color-background: #F8FAFC;
  --color-surface: #FFFFFF;
  --color-border: #E2E8F0;

  --color-text: #0F172A;
  --color-text-secondary: #475569;
  --color-text-muted: #64748B;

  --color-success: #16A34A;
  --color-warning: #D97706;
  --color-danger: #DC2626;
  --color-info: #0284C7;

  --radius-sm: 6px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --radius-xl: 16px;

  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-5: 20px;
  --space-6: 24px;
  --space-8: 32px;
}
```

------------------------------------------------------------------------

# 62. TAILWIND MAPPING

Suggested mapping:

``` text
primary → blue
success → green
warning → amber
danger → red
info → sky
neutral → slate
```

Use semantic component classes where practical rather than scattering
raw color classes.

Example:

``` text
bg-primary
text-primary
border-default
text-muted
```

------------------------------------------------------------------------

# 63. COMPONENT NAMING

React components:

``` text
Button
Input
Select
Combobox
Card
Badge
Modal
Drawer
DataTable
Pagination
StatCard
PageHeader
EmptyState
ErrorState
Skeleton
Toast
ConfirmDialog
```

Domain components:

``` text
WorkOrderCard
WorkOrderStatus
VehicleCard
CustomerCard
ServiceItem
PartItem
InspectionChecklist
StockBadge
PaymentSummary
InvoiceSummary
```

------------------------------------------------------------------------

# 64. COMPONENT FOLDER STRUCTURE

Recommended:

``` text
src/
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
│   ├── customers/
│   ├── vehicles/
│   ├── work-orders/
│   ├── services/
│   ├── parts/
│   ├── inventory/
│   ├── purchasing/
│   ├── invoices/
│   ├── payments/
│   ├── reports/
│   └── settings/
│
└── layouts/
```

------------------------------------------------------------------------

# 65. FORM VALIDATION VISUAL

Invalid:

``` text
Nomor HP *
[ 08123                 ]

⚠ Nomor HP minimal 10 digit.
```

Valid:

``` text
Nomor HP *
[ 081234567890          ] ✓
```

Do not show success indicators excessively.

------------------------------------------------------------------------

# 66. DESTRUCTIVE UX

Danger actions: - delete - cancel WO - stock adjustment - deactivate
user - void invoice

Rules: 1. explain consequence 2. confirm 3. require reason when
business-critical 4. disable duplicate submission 5. audit the action

------------------------------------------------------------------------

# 67. SUCCESS UX

After successful mutation: - toast - update UI state - navigate only
when useful - preserve context

Example:

``` text
✓ Work Order berhasil dibuat.
```

------------------------------------------------------------------------

# 68. ERROR UX

Map technical errors to business language.

Instead of:

``` text
SQLSTATE[23000]
```

Show:

``` text
Data tidak dapat disimpan karena nomor WO sudah digunakan.
```

Log technical error internally.

------------------------------------------------------------------------

# 69. ACCESSIBILITY

Target:

``` text
WCAG 2.1 AA
```

Requirements: - semantic HTML - keyboard navigation - focus states -
labels - aria-label - accessible dialogs - accessible tables -
sufficient contrast - screen-reader friendly status - no color-only
meaning

------------------------------------------------------------------------

# 70. FOCUS MANAGEMENT

Modal: - focus first interactive element - trap focus - restore focus on
close

Drawer: - same principle

After form submit: - announce success - focus relevant result where
appropriate

------------------------------------------------------------------------

# 71. KEYBOARD SHORTCUTS

V1:

``` text
Ctrl + K      Global Search
Esc           Close Modal/Drawer
Enter         Submit focused simple form
```

Future:

``` text
N             New Work Order
P             Payment
```

Do not conflict with browser behavior.

------------------------------------------------------------------------

# 72. NOTIFICATION ICON

Header:

``` text
🔔
```

Badge only when unread count \> 0.

Do not use notification count for ordinary informational events unless
actionable.

------------------------------------------------------------------------

# 73. LOGO USAGE

Primary application logo:

``` text
GARAGE PRO
```

Sidebar collapsed: - icon/monogram version

Login: - full logo

Favicon: - GP monogram

------------------------------------------------------------------------

# 74. EMPTY / ERROR ILLUSTRATIONS

Use minimal iconography rather than large illustrations.

Example:

``` text
PackageOpen
ClipboardX
WifiOff
SearchX
```

This keeps the application professional and lightweight.

------------------------------------------------------------------------

# 75. DESIGN ANTI-PATTERNS

Do NOT use:

``` text
❌ excessive gradients
❌ glassmorphism everywhere
❌ giant illustrations
❌ tiny text
❌ low contrast gray text
❌ huge rounded cards
❌ unnecessary animations
❌ auto-playing content
❌ 3D charts
❌ excessive badges
❌ nested modal over modal
❌ important action hidden in three-dot menu
```

------------------------------------------------------------------------

# 76. MOTION

Animations should be subtle.

Recommended: - 150--200ms for micro interactions - 200--300ms for
modal/drawer - respect `prefers-reduced-motion`

Use animation for: - hover - state transition - drawer - toast -
skeleton

Do not animate business data unnecessarily.

------------------------------------------------------------------------

# 77. RESPONSIVE BEHAVIOR MATRIX

  Component    Mobile       Tablet          Desktop
  ------------ ------------ --------------- ----------
  Sidebar      Drawer       Drawer          Fixed
  Table        Cards        Scroll/Card     Full
  Modal        Full/Sheet   Modal           Modal
  Drawer       Full         Drawer          Drawer
  KPI          1 col        2 col           4 col
  Form         1 col        2 col           2--3 col
  Action bar   Sticky       Sticky/normal   Normal
  Kanban       Horizontal   Horizontal      Full

------------------------------------------------------------------------

# 78. UI PERFORMANCE

Avoid: - loading entire datasets - rendering huge tables - unnecessary
re-renders - giant images - excessive chart libraries

Use: - pagination - server-side filtering - lazy loading - code
splitting - image optimization

------------------------------------------------------------------------

# 79. FRONTEND ARCHITECTURE PRINCIPLE

UI components should not directly contain business rules.

Bad:

``` text
Button → mutate stock directly
```

Good:

``` text
Component
   ↓
Feature Hook
   ↓
API Client
   ↓
Service Layer
   ↓
Backend Business Rule
```

------------------------------------------------------------------------

# 80. DESIGN SYSTEM ACCEPTANCE CRITERIA

Design System V1 is complete when:

``` text
[ ] Color tokens defined
[ ] Typography defined
[ ] Spacing defined
[ ] Radius defined
[ ] Shadows defined
[ ] Buttons defined
[ ] Inputs defined
[ ] Select defined
[ ] Combobox defined
[ ] Cards defined
[ ] Badges defined
[ ] Tables defined
[ ] Modal defined
[ ] Drawer defined
[ ] Toast defined
[ ] Alerts defined
[ ] Navigation defined
[ ] Mobile navigation defined
[ ] Dashboard patterns defined
[ ] Kanban defined
[ ] Charts defined
[ ] Accessibility defined
[ ] Dark mode defined
[ ] Responsive behavior defined
[ ] Tailwind mapping defined
```

------------------------------------------------------------------------

# 81. IMPLEMENTATION PRIORITY

Build UI system in this order:

``` text
1. Tokens
2. Typography
3. Button
4. Input
5. Select / Combobox
6. Badge
7. Card
8. Table
9. Modal
10. Drawer
11. Toast
12. Alert
13. Navigation
14. Page Header
15. Empty/Error/Loading
16. DataTable
17. Charts
18. Domain components
```

------------------------------------------------------------------------

# 82. NEXT DOCUMENT

Next create:

``` text
GARAGE_PRO_PROJECT_ARCHITECTURE_V1.md
```

That document must define:

``` text
Frontend architecture
Backend architecture
Folder structure
Module boundaries
State management
API client
Authentication
Authorization
Service layer
Database access
Validation
Error handling
Logging
File structure
Environment variables
Docker/deployment structure
Git workflow
Coding conventions
Claude Code implementation instructions
```

After that:

``` text
DATABASE MIGRATION
        ↓
BACKEND IMPLEMENTATION
        ↓
FRONTEND IMPLEMENTATION
        ↓
TESTING
        ↓
DEPLOYMENT
```

------------------------------------------------------------------------

# END OF DOCUMENT

**GARAGE PRO --- DESIGN SYSTEM V1**\
**Status: Development Ready**
