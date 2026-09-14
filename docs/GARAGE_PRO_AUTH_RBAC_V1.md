# GARAGE PRO --- AUTHENTICATION + RBAC V1

**Project:** GARAGE PRO --- Workshop Management System\
**Document:** Authentication + Role Based Access Control Specification\
**Version:** V1.0\
**Status:** Implementation Ready\
**Backend:** Node.js + Express + TypeScript\
**Database:** MySQL 8+\
**ORM:** Sequelize\
**Authentication:** JWT + server-side session/refresh token strategy\
**Password:** bcrypt/Argon2

------------------------------------------------------------------------

# 1. Tujuan

Phase 10 membangun fondasi keamanan aplikasi GARAGE PRO:

``` text
Login
↓
Credential Validation
↓
Password Verification
↓
Access Token
↓
Refresh Session
↓
Current User
↓
Role
↓
Permission
↓
API Authorization
↓
Frontend Route Guard
↓
Audit Log
```

Tujuan utama:

-   user dapat login secara aman;
-   password tidak pernah disimpan plaintext;
-   API memiliki authentication middleware;
-   endpoint memiliki permission boundary;
-   role menentukan kumpulan permission;
-   frontend hanya menampilkan fitur yang diizinkan;
-   session dapat diakhiri/logout;
-   login/logout dapat diaudit;
-   akun inactive/disabled tidak dapat login;
-   token invalid/expired ditolak;
-   authorization tidak boleh hanya bergantung pada frontend.

------------------------------------------------------------------------

# 2. Prinsip Keamanan

## 2.1 Authentication ≠ Authorization

Authentication menjawab:

> Siapa user ini?

Authorization menjawab:

> Apa yang boleh dilakukan user ini?

Contoh:

``` text
ADMIN login berhasil
        ↓
role = ADMIN
        ↓
permission:
  customers.view
  customers.create
  work_orders.view
  ...
```

API tetap wajib memeriksa permission.

------------------------------------------------------------------------

# 3. Role V1

GARAGE PRO memiliki 4 role utama:

  Role        Fokus
  ----------- -----------------------------------------
  OWNER       Full business visibility & control
  ADMIN       Front desk & operational administration
  MECHANIC    Inspection, service execution & QC
  WAREHOUSE   Parts & inventory

Role code harus konsisten:

``` text
OWNER
ADMIN
MECHANIC
WAREHOUSE
```

------------------------------------------------------------------------

# 4. Permission Model

Permission menggunakan format:

``` text
module.action
```

Contoh:

``` text
customers.view
customers.create
customers.update
work_orders.view
work_orders.approve
inventory.issue
payments.create
```

Permission code menjadi contract antara:

``` text
Database
Backend
Frontend
Tests
```

Jangan mengubah permission code sembarangan setelah production.

------------------------------------------------------------------------

# 5. Permission Catalog

## Dashboard

``` text
dashboard.view
```

## Customer

``` text
customers.view
customers.create
customers.update
customers.delete
```

## Vehicle

``` text
vehicles.view
vehicles.create
vehicles.update
```

## Mechanic

``` text
mechanics.view
mechanics.create
mechanics.update
```

## Service

``` text
services.view
services.create
services.update
services.delete
```

## Spare Part

``` text
parts.view
parts.create
parts.update
parts.delete
```

## Inventory

``` text
inventory.view
inventory.issue
inventory.return
inventory.adjust
inventory.opname
```

## Purchase

``` text
purchases.view
purchases.create
purchases.update
purchases.receive
```

## Work Order

``` text
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
```

## Invoice

``` text
invoices.view
invoices.create
```

## Payment

``` text
payments.view
payments.create
payments.refund
```

## Reports

``` text
reports.view
```

## Users

``` text
users.view
users.create
users.update
users.disable
```

## Roles

``` text
roles.view
roles.update
```

## Settings

``` text
settings.view
settings.update
```

## Audit

``` text
audit_logs.view
```

------------------------------------------------------------------------

# 6. Role Permission Matrix

## OWNER

OWNER mendapatkan seluruh permission V1.

``` text
ALL
```

Namun endpoint tertentu tetap harus memiliki explicit permission check,
bukan special-case berdasarkan string role di controller.

------------------------------------------------------------------------

## ADMIN

``` text
dashboard.view

customers.view
customers.create
customers.update

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

purchases.view
purchases.create
purchases.update

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

reports.view
```

ADMIN tidak default mendapatkan:

``` text
users.create
roles.update
settings.update
audit_logs.view
inventory.adjust
inventory.opname
payments.refund
```

Permission tersebut dapat diberikan OWNER secara eksplisit jika
dibutuhkan.

------------------------------------------------------------------------

## MECHANIC

``` text
dashboard.view

customers.view
customers.create

vehicles.view
vehicles.create
vehicles.update

mechanics.view

services.view
parts.view

work_orders.view
work_orders.create
work_orders.update
work_orders.start
work_orders.qc
work_orders.rework
work_orders.ready
```

MECHANIC tidak boleh:

``` text
payments.*
users.*
roles.*
settings.*
inventory.adjust
inventory.opname
purchases.*
```

------------------------------------------------------------------------

## WAREHOUSE

``` text
dashboard.view

parts.view
parts.create
parts.update

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
```

WAREHOUSE tidak boleh:

``` text
payments.*
users.*
roles.*
settings.*
work_orders.approve
work_orders.complete
```

------------------------------------------------------------------------

# 7. Authentication Architecture

Recommended:

``` text
Browser
   ↓
POST /api/v1/auth/login
   ↓
AuthController
   ↓
AuthService
   ↓
UserRepository
   ↓
PasswordHasher
   ↓
Session Service
   ↓
Access Token
   ↓
Authenticated API
```

Refresh:

``` text
Refresh Token
↓
Session lookup
↓
Validate session
↓
Rotate refresh token
↓
Issue new access token
```

------------------------------------------------------------------------

# 8. Token Strategy

Recommended V1:

``` text
Access Token
+
Refresh Token / Session
```

Access token:

-   short-lived;
-   contains minimal claims;
-   used on API requests.

Refresh token:

-   longer-lived;
-   associated with server-side session;
-   revocable;
-   rotated when refreshed.

Jangan menyimpan sensitive information dalam JWT.

------------------------------------------------------------------------

# 9. JWT Claims

Minimal access token:

``` json
{
  "sub": "123",
  "role": "ADMIN",
  "sessionId": "session-id",
  "iat": 1726300000,
  "exp": 1726328800
}
```

Jangan menyimpan:

``` text
password
password_hash
full permission list
sensitive customer information
financial information
```

Permission sebaiknya di-resolve server-side atau melalui controlled
cache.

------------------------------------------------------------------------

# 10. Access Token Lifetime

Recommended default:

``` text
15 minutes
```

Dapat dikonfigurasi melalui:

``` env
JWT_ACCESS_EXPIRES_IN=15m
```

Refresh session:

``` text
7 days
```

Environment:

``` env
JWT_REFRESH_EXPIRES_IN=7d
```

Untuk kebutuhan operasional, expiry dapat disesuaikan setelah security
review.

------------------------------------------------------------------------

# 11. Refresh Token Security

Refresh token harus:

-   random;
-   high entropy;
-   tidak predictable;
-   disimpan server-side dalam bentuk hash;
-   memiliki expiry;
-   dapat revoked;
-   dirotasi setelah digunakan.

Database table tambahan yang direkomendasikan:

``` text
auth_sessions
```

Kolom:

``` text
id
user_id
token_hash
expires_at
revoked_at
last_used_at
ip_address
user_agent
created_at
updated_at
```

Index:

``` text
INDEX(user_id)
INDEX(token_hash)
INDEX(expires_at)
INDEX(revoked_at)
```

`token_hash` harus unique.

------------------------------------------------------------------------

# 12. Login Flow

``` text
POST /auth/login
        ↓
Validate username/email + password
        ↓
Find active user
        ↓
Verify password
        ↓
Check account active
        ↓
Create session
        ↓
Generate access token
        ↓
Generate refresh token
        ↓
Audit LOGIN_SUCCESS
        ↓
Return auth response
```

Jika gagal:

``` text
Audit LOGIN_FAILED
```

Jangan mengungkapkan apakah username atau password yang salah.

Response:

``` json
{
  "message": "Invalid credentials"
}
```

------------------------------------------------------------------------

# 13. Login Request

``` http
POST /api/v1/auth/login
Content-Type: application/json
```

Body:

``` json
{
  "username": "admin",
  "password": "********"
}
```

`username` dapat menerima username atau email jika business requirement
mengizinkan.

------------------------------------------------------------------------

# 14. Login Response

``` json
{
  "success": true,
  "data": {
    "accessToken": "eyJ...",
    "refreshToken": "random-token",
    "expiresIn": 900,
    "user": {
      "id": 1,
      "name": "Garage Pro Admin",
      "username": "admin",
      "role": {
        "code": "ADMIN",
        "name": "Administrator"
      }
    }
  }
}
```

------------------------------------------------------------------------

# 15. Current User

Endpoint:

``` http
GET /api/v1/auth/me
Authorization: Bearer <access-token>
```

Response:

``` json
{
  "success": true,
  "data": {
    "id": 1,
    "name": "Garage Pro Admin",
    "username": "admin",
    "email": "admin@example.local",
    "role": {
      "code": "ADMIN",
      "name": "Administrator"
    },
    "permissions": [
      "dashboard.view",
      "customers.view",
      "customers.create"
    ]
  }
}
```

Frontend menggunakan `/auth/me` untuk bootstrap authentication state.

------------------------------------------------------------------------

# 16. Refresh Endpoint

``` http
POST /api/v1/auth/refresh
```

Body:

``` json
{
  "refreshToken": "..."
}
```

Flow:

``` text
receive refresh token
↓
hash token
↓
find session
↓
validate not revoked
↓
validate not expired
↓
validate user active
↓
revoke old session/token
↓
create rotated session
↓
issue new access + refresh token
```

Jika invalid:

``` text
401 AUTH_REFRESH_INVALID
```

------------------------------------------------------------------------

# 17. Logout

Endpoint:

``` http
POST /api/v1/auth/logout
```

Flow:

``` text
authenticate
↓
revoke current session
↓
audit LOGOUT
↓
return success
```

Logout tidak harus menghapus user.

------------------------------------------------------------------------

# 18. Logout All Sessions

OWNER/ADMIN dapat memiliki kebutuhan revoke session user.

Endpoint internal/admin:

``` http
POST /api/v1/users/:id/revoke-sessions
```

Permission:

``` text
users.update
```

Use cases:

-   device hilang;
-   password reset;
-   employee keluar;
-   security incident.

------------------------------------------------------------------------

# 19. Password Policy

Minimum V1:

``` text
minimum 8 characters
```

Recommended production:

``` text
minimum 10–12 characters
```

Password harus diverifikasi server-side.

Jangan menggunakan:

``` text
password == plaintext
```

Hash:

``` text
bcrypt cost >= 12
```

atau Argon2id dengan parameter yang sesuai environment.

------------------------------------------------------------------------

# 20. Password Reset

V1 minimal:

``` text
OWNER/ADMIN can disable user
```

Recommended endpoint future:

``` text
POST /auth/forgot-password
POST /auth/reset-password
POST /auth/change-password
```

Untuk V1, implementasikan minimal:

``` text
POST /auth/change-password
```

User harus memberikan:

``` json
{
  "currentPassword": "...",
  "newPassword": "..."
}
```

Setelah password berubah:

``` text
revoke all sessions
```

User login kembali.

------------------------------------------------------------------------

# 21. Account Status

User:

``` text
is_active = true
```

boleh login.

Jika:

``` text
is_active = false
```

maka:

``` text
login denied
existing sessions revoked
```

Error:

``` text
AUTH_ACCOUNT_DISABLED
```

Jangan menghapus user untuk menonaktifkan akses.

------------------------------------------------------------------------

# 22. Authentication Middleware

File:

``` text
backend/src/middleware/authenticate.ts
```

Responsibilities:

``` text
read Authorization header
↓
validate Bearer format
↓
verify JWT
↓
extract user/session
↓
attach authenticated user to req
```

Request context:

``` ts
req.auth = {
  userId,
  roleCode,
  sessionId,
};
```

Jangan attach password atau sensitive user data.

------------------------------------------------------------------------

# 23. Authorization Middleware

File:

``` text
backend/src/middleware/authorize.ts
```

Usage:

``` ts
router.get(
  '/customers',
  authenticate,
  authorize('customers.view'),
  customerController.list
);
```

Multiple permission:

``` ts
authorizeAny(
  'inventory.issue',
  'inventory.adjust'
)
```

atau:

``` ts
authorizeAll(
  'reports.view'
)
```

Default behavior:

``` text
missing permission → 403
```

------------------------------------------------------------------------

# 24. 401 vs 403

Gunakan:

``` text
401 Unauthorized
```

untuk:

``` text
missing token
invalid token
expired token
invalid session
```

Gunakan:

``` text
403 Forbidden
```

untuk:

``` text
authenticated
BUT
permission denied
```

------------------------------------------------------------------------

# 25. Error Codes

Authentication:

``` text
AUTH_INVALID_CREDENTIALS
AUTH_ACCOUNT_DISABLED
AUTH_TOKEN_MISSING
AUTH_TOKEN_INVALID
AUTH_TOKEN_EXPIRED
AUTH_SESSION_REVOKED
AUTH_REFRESH_INVALID
AUTH_PASSWORD_INVALID
AUTH_PASSWORD_POLICY
```

Authorization:

``` text
AUTH_FORBIDDEN
AUTH_PERMISSION_REQUIRED
```

------------------------------------------------------------------------

# 26. Auth API Specification

## POST /auth/login

Permission:

``` text
PUBLIC
```

## POST /auth/refresh

Permission:

``` text
PUBLIC_WITH_REFRESH_TOKEN
```

## POST /auth/logout

Permission:

``` text
AUTHENTICATED
```

## GET /auth/me

Permission:

``` text
AUTHENTICATED
```

## POST /auth/change-password

Permission:

``` text
AUTHENTICATED
```

------------------------------------------------------------------------

# 27. User API

## GET /users

Permission:

``` text
users.view
```

## POST /users

Permission:

``` text
users.create
```

## GET /users/:id

Permission:

``` text
users.view
```

## PUT /users/:id

Permission:

``` text
users.update
```

## POST /users/:id/disable

Permission:

``` text
users.disable
```

## POST /users/:id/enable

Permission:

``` text
users.update
```

## POST /users/:id/revoke-sessions

Permission:

``` text
users.update
```

------------------------------------------------------------------------

# 28. User Creation Rules

Required:

``` text
name
username
role_id
password
```

Optional:

``` text
email
phone
```

Validation:

``` text
username unique
email unique when provided
role exists
password meets policy
```

Never return:

``` text
password_hash
```

------------------------------------------------------------------------

# 29. User Update Rules

Allowed:

``` text
name
email
phone
role
is_active
```

Password change should have a separate endpoint.

Do not allow ordinary user update endpoint to silently replace password.

------------------------------------------------------------------------

# 30. Last OWNER Protection

System harus mencegah:

``` text
disable last active OWNER
```

dan:

``` text
remove OWNER role from last active OWNER
```

Error:

``` text
AUTH_LAST_OWNER_PROTECTED
```

Minimal harus selalu ada satu active OWNER.

------------------------------------------------------------------------

# 31. Role API

## GET /roles

``` text
roles.view
```

## GET /roles/:id

``` text
roles.view
```

## PUT /roles/:id/permissions

``` text
roles.update
```

OWNER-only business policy dapat diterapkan untuk system roles.

------------------------------------------------------------------------

# 32. System Roles

Default roles:

``` text
OWNER
ADMIN
MECHANIC
WAREHOUSE
```

`is_system = true`.

System roles tidak boleh dihapus.

Nama/description dapat diubah dengan controlled policy, tetapi `code`
tidak boleh diubah.

------------------------------------------------------------------------

# 33. Permission Loading

Recommended:

``` text
login
↓
user role
↓
role permissions
↓
server-side authorization
```

Frontend menerima permission list untuk UI.

Namun:

> Permission di frontend hanya untuk UX. Security enforcement tetap di
> backend.

------------------------------------------------------------------------

# 34. Frontend Auth Architecture

Folder:

``` text
frontend/src/
└── features/
    └── auth/
        ├── api/
        │   └── authApi.ts
        ├── components/
        │   ├── LoginForm.tsx
        │   └── PermissionGate.tsx
        ├── hooks/
        │   ├── useAuth.ts
        │   └── usePermission.ts
        ├── pages/
        │   └── LoginPage.tsx
        ├── types/
        │   └── auth.types.ts
        └── store/
            └── authStore.ts
```

------------------------------------------------------------------------

# 35. Frontend Auth State

State:

``` ts
type AuthState = {
  user: AuthUser | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  initialized: boolean;
};
```

Bootstrap:

``` text
application starts
↓
read auth state
↓
call /auth/me
↓
success → authenticated
401 → logout local state
↓
render application
```

Jangan render dashboard sebelum auth bootstrap selesai.

------------------------------------------------------------------------

# 36. Token Storage

Security recommendation:

### Best production approach

``` text
Access token → memory
Refresh token → HttpOnly Secure SameSite cookie
```

Advantages:

-   refresh token tidak accessible via JavaScript;
-   mengurangi XSS exposure.

Jika V1 menggunakan localStorage karena keterbatasan arsitektur:

``` text
document risk
XSS protection becomes critical
```

Prefer HttpOnly cookie untuk refresh token.

------------------------------------------------------------------------

# 37. Axios/API Client

Structure:

``` text
apiClient
↓
attach access token
↓
request
↓
401
↓
attempt refresh once
↓
retry original request
↓
if refresh fails
logout
```

Hindari infinite refresh loop.

Flag:

``` text
_retry
```

atau equivalent request state.

------------------------------------------------------------------------

# 38. PermissionGate

Usage:

``` tsx
<PermissionGate permission="customers.create">
  <Button>Tambah Customer</Button>
</PermissionGate>
```

Jika tidak punya permission:

``` text
render nothing
```

atau:

``` text
disabled state
```

sesuai UX.

------------------------------------------------------------------------

# 39. Route Guard

Routes:

``` text
/login
```

public.

Protected:

``` text
/
/dashboard
/work-orders
/customers
/vehicles
/services
/parts
/inventory
/purchases
/invoices
/payments
/reports
/users
/roles
/settings
/audit-logs
```

Route guard:

``` text
not authenticated
→ /login
```

Permission guard:

``` text
authenticated
but no permission
→ /403
```

------------------------------------------------------------------------

# 40. UI Role Experience

## OWNER

Dashboard:

``` text
Revenue
WO
Unit Entry
Outstanding Payment
Stock Value
Mechanic Productivity
```

Full navigation.

## ADMIN

Navigation:

``` text
Dashboard
Work Orders
Customers
Vehicles
Services
Spare Parts
Purchasing
Invoice
Payment
Reports
```

## MECHANIC

Mobile-first:

``` text
My Work Orders
Inspection
Services
Parts
Recommendations
QC
```

## WAREHOUSE

Mobile/desktop:

``` text
Stock
Issue Part
Return Part
Receiving
Opname
Low Stock
```

------------------------------------------------------------------------

# 41. Security Headers

Use:

``` text
helmet
```

Recommended:

``` text
Content-Security-Policy
X-Content-Type-Options
Referrer-Policy
X-Frame-Options
Strict-Transport-Security
```

Production HTTPS wajib.

------------------------------------------------------------------------

# 42. Rate Limiting

Login harus memiliki rate limiting.

Contoh:

``` text
5 failed attempts / 15 minutes / IP
```

dan/atau account-aware throttling.

Jangan membuat lockout permanen hanya karena percobaan gagal.

Response tetap generic:

``` text
Invalid credentials
```

------------------------------------------------------------------------

# 43. Brute Force Protection

Implement:

``` text
rate limiter
+
failed login audit
+
progressive delay
```

Jangan menggunakan security question sebagai pengganti password.

------------------------------------------------------------------------

# 44. Audit Events

Minimal:

``` text
LOGIN_SUCCESS
LOGIN_FAILED
LOGOUT
TOKEN_REFRESH
PASSWORD_CHANGED
USER_CREATED
USER_UPDATED
USER_DISABLED
USER_ENABLED
ROLE_CHANGED
SESSIONS_REVOKED
```

Audit payload jangan menyimpan:

``` text
password
refresh token
access token
password hash
```

------------------------------------------------------------------------

# 45. Login Audit Example

``` json
{
  "action": "LOGIN_SUCCESS",
  "module": "AUTH",
  "entity_type": "USER",
  "entity_id": 1,
  "new_values": {
    "sessionId": "..."
  }
}
```

Untuk gagal:

``` json
{
  "action": "LOGIN_FAILED",
  "module": "AUTH",
  "entity_type": "AUTH",
  "new_values": {
    "identifier": "admin",
    "reason": "INVALID_CREDENTIALS"
  }
}
```

Jangan menyimpan password.

------------------------------------------------------------------------

# 46. Session Revocation

Session dapat dicabut karena:

``` text
logout
password change
user disabled
admin revoke
security incident
refresh token rotation
expiry
```

Expired/revoked sessions tidak boleh digunakan.

------------------------------------------------------------------------

# 47. Session Cleanup

Scheduled job atau periodic cleanup:

``` text
delete/revoke sessions where expires_at < NOW()
```

Jika retention audit diperlukan, session records dapat dipertahankan
sebagai histori.

------------------------------------------------------------------------

# 48. Database Migration Tambahan

Tambahkan migration:

``` text
create-auth-sessions
```

Logical position setelah:

``` text
users
```

karena membutuhkan:

``` text
user_id → users.id
```

Schema:

``` text
auth_sessions
--------------
id BIGINT UNSIGNED PK
user_id BIGINT UNSIGNED NOT NULL
token_hash VARCHAR(255) NOT NULL UNIQUE
expires_at DATETIME NOT NULL
revoked_at DATETIME NULL
last_used_at DATETIME NULL
ip_address VARCHAR(45) NULL
user_agent VARCHAR(500) NULL
created_at DATETIME NOT NULL
updated_at DATETIME NOT NULL
```

FK:

``` text
user_id → users.id
ON DELETE RESTRICT
```

------------------------------------------------------------------------

# 49. Backend Folder Structure

``` text
backend/src/
├── config/
│   ├── env.ts
│   └── database.ts
│
├── controllers/
│
├── services/
│   ├── auth/
│   │   ├── auth.service.ts
│   │   ├── password.service.ts
│   │   ├── token.service.ts
│   │   └── session.service.ts
│
├── repositories/
│   ├── user.repository.ts
│   ├── role.repository.ts
│   └── session.repository.ts
│
├── middleware/
│   ├── authenticate.ts
│   ├── authorize.ts
│   └── rateLimiter.ts
│
├── validators/
│   └── auth.validators.ts
│
├── routes/
│   └── auth.routes.ts
│
├── models/
│   ├── User.ts
│   ├── Role.ts
│   ├── Permission.ts
│   ├── RolePermission.ts
│   └── AuthSession.ts
│
└── types/
    └── express.d.ts
```

------------------------------------------------------------------------

# 50. Auth Service Pseudocode

``` ts
async login(input: LoginInput) {
  validateInput(input);

  const user = await userRepository.findByUsernameOrEmail(
    input.username
  );

  if (!user) {
    await auditLoginFailure(input.username);
    throw invalidCredentials();
  }

  const validPassword = await passwordService.verify(
    input.password,
    user.passwordHash
  );

  if (!validPassword) {
    await auditLoginFailure(input.username);
    throw invalidCredentials();
  }

  if (!user.isActive) {
    throw accountDisabled();
  }

  const session = await sessionService.create(user);

  const accessToken = tokenService.createAccessToken({
    userId: user.id,
    roleCode: user.role.code,
    sessionId: session.id,
  });

  await auditLoginSuccess(user, session);

  return {
    accessToken,
    refreshToken: session.refreshToken,
    user: sanitizeUser(user),
  };
}
```

------------------------------------------------------------------------

# 51. Authorization Pseudocode

``` ts
export const authorize =
  (permission: string) =>
  async (req, res, next) => {

    const user = req.auth;

    if (!user) {
      return next(new UnauthorizedError());
    }

    const allowed =
      await permissionService.userHasPermission(
        user.userId,
        permission
      );

    if (!allowed) {
      return next(new ForbiddenError());
    }

    next();
  };
```

Authorization harus berada sebelum controller.

------------------------------------------------------------------------

# 52. Service Layer Rule

Controller:

``` text
HTTP concerns
```

Service:

``` text
business/security rules
```

Repository:

``` text
database access
```

Contoh:

``` text
AuthController
   ↓
AuthService
   ↓
UserRepository
   ↓
Sequelize
```

Jangan menaruh password verification langsung di route.

------------------------------------------------------------------------

# 53. Secure User Serialization

Create helper:

``` ts
sanitizeUser(user)
```

Return:

``` text
id
name
username
email
phone
role
is_active
```

Never return:

``` text
password_hash
refresh_token
token_hash
```

------------------------------------------------------------------------

# 54. Login UX

Login screen:

``` text
GARAGE PRO logo
Workshop Management System

Username / Email
Password
[ Show / Hide ]

[ MASUK ]

Forgot password? (future)
```

State:

``` text
idle
loading
success
error
disabled
```

Error generic:

``` text
Username atau password salah.
```

Account disabled:

``` text
Akun Anda tidak aktif. Hubungi administrator.
```

------------------------------------------------------------------------

# 55. Frontend Loading Rules

Saat login:

``` text
button disabled
spinner
prevent double submit
```

Saat refresh:

``` text
silent refresh
```

Jangan menampilkan error refresh token kepada user jika session memang
harus berakhir.

Redirect:

``` text
→ /login
```

------------------------------------------------------------------------

# 56. Authenticated Application Bootstrap

Sequence:

``` text
App start
 ↓
AuthProvider
 ↓
load persisted session
 ↓
GET /auth/me
 ↓
success
 ↓
load permissions
 ↓
render App
```

Jika gagal:

``` text
clear local auth state
→ login
```

------------------------------------------------------------------------

# 57. Permission Caching

Backend dapat cache:

``` text
userId → permissions
```

TTL pendek jika diperlukan.

Namun setelah:

``` text
role changed
user disabled
permission changed
```

cache harus di-invalidate.

V1 boleh menggunakan direct DB lookup jika traffic masih rendah.

Prioritas:

``` text
correctness > premature optimization
```

------------------------------------------------------------------------

# 58. Security Rules

WAJIB:

``` text
HTTPS production
password hashing
JWT signature validation
short access token expiry
refresh token rotation
session revocation
rate limiting
helmet
input validation
permission middleware
audit login
no plaintext password
no token logging
```

DILARANG:

``` text
password in logs
refresh token in audit
access token in audit
permission check only frontend
hard-coded JWT secret
hard-coded production password
```

------------------------------------------------------------------------

# 59. Environment Variables

`.env.example`:

``` env
NODE_ENV=development

JWT_SECRET=change-me
JWT_ACCESS_EXPIRES_IN=15m
JWT_REFRESH_EXPIRES_IN=7d

BCRYPT_ROUNDS=12

AUTH_COOKIE_SECURE=false
AUTH_COOKIE_SAME_SITE=lax

LOGIN_RATE_LIMIT_WINDOW_MS=900000
LOGIN_RATE_LIMIT_MAX=5
```

Production:

``` text
JWT_SECRET must be long random secret
AUTH_COOKIE_SECURE=true
HTTPS=true
```

------------------------------------------------------------------------

# 60. Testing Strategy

## Unit Test

Test:

``` text
password hashing
password verification
JWT generation
JWT verification
permission matching
session expiry
session revocation
```

## Integration Test

Test:

``` text
login success
login invalid password
login disabled account
auth/me
refresh
logout
change password
```

## Authorization Test

Test every critical permission:

``` text
ADMIN cannot inventory.adjust
MECHANIC cannot payments.create
WAREHOUSE cannot payments.create
OWNER can access all
```

------------------------------------------------------------------------

# 61. Auth Test Matrix

  Scenario                             Expected
  ------------------------------------ ---------------------------
  valid login                          200
  invalid password                     401
  unknown username                     401
  disabled user                        401
  missing token                        401
  invalid JWT                          401
  expired JWT                          401
  revoked session                      401
  valid permission                     200
  missing permission                   403
  valid refresh                        200
  revoked refresh                      401
  expired refresh                      401
  logout                               200
  reuse rotated refresh token          401
  change password                      200
  old sessions after password change   revoked
  disable user                         existing sessions revoked
  disable last OWNER                   rejected

------------------------------------------------------------------------

# 62. Security Acceptance Criteria

Phase Auth/RBAC dianggap selesai jika:

-   [ ] Login berhasil untuk user aktif.
-   [ ] Password tidak plaintext.
-   [ ] Password hash tidak pernah dikembalikan API.
-   [ ] JWT access token valid.
-   [ ] JWT expiry bekerja.
-   [ ] Refresh token bekerja.
-   [ ] Refresh token rotation bekerja.
-   [ ] Session dapat direvoke.
-   [ ] Logout merevoke session.
-   [ ] `/auth/me` bekerja.
-   [ ] Disabled user tidak dapat login.
-   [ ] Permission middleware bekerja.
-   [ ] 401 dan 403 benar.
-   [ ] OWNER protection bekerja.
-   [ ] Login rate limit aktif.
-   [ ] Login success/failure diaudit.
-   [ ] Password change merevoke session.
-   [ ] Frontend route guard bekerja.
-   [ ] Frontend permission gate bekerja.
-   [ ] Production secret tidak hard-coded.
-   [ ] TypeScript build PASS.
-   [ ] Auth integration tests PASS.

------------------------------------------------------------------------

# 63. Definition of Done

``` text
DATABASE
  ↓
auth_sessions
  ↓
USER MODEL
  ↓
PASSWORD SERVICE
  ↓
SESSION SERVICE
  ↓
TOKEN SERVICE
  ↓
AUTH SERVICE
  ↓
AUTH MIDDLEWARE
  ↓
AUTHORIZATION MIDDLEWARE
  ↓
AUTH API
  ↓
USER/ROLE API
  ↓
FRONTEND AUTH
  ↓
ROUTE GUARD
  ↓
PERMISSION GUARD
  ↓
AUDIT
  ↓
TEST
```

Jika salah satu critical security layer belum selesai, Phase 10 belum
DONE.

------------------------------------------------------------------------

# 64. Claude Code Master Prompt --- AUTH + RBAC

``` text
You are the senior security-focused backend engineer implementing Phase 10 of GARAGE PRO.

PROJECT:
GARAGE PRO — Workshop Management System V1

STACK:
- Node.js
- Express
- TypeScript
- Sequelize 6
- MySQL 8+
- React
- Vite
- TanStack Query
- Axios
- Zod

REFERENCE DOCUMENTS:
- GARAGE_PRO_MASTER_DEVELOPMENT_SPECIFICATION_V1.md
- GARAGE_PRO_DATABASE_DESIGN_V1.md
- GARAGE_PRO_API_SPECIFICATION_V1.md
- GARAGE_PRO_BUSINESS_RULES_V1.md
- GARAGE_PRO_PROJECT_ARCHITECTURE_V1.md
- GARAGE_PRO_PROJECT_SCAFFOLD_V1.md
- GARAGE_PRO_DATABASE_MIGRATION_V1.md
- GARAGE_PRO_AUTH_RBAC_V1.md

OBJECTIVE:
Implement production-quality Authentication and RBAC without redesigning the existing architecture.

REQUIREMENTS:

1. AUTHENTICATION
Implement:
- POST /api/v1/auth/login
- POST /api/v1/auth/refresh
- POST /api/v1/auth/logout
- GET /api/v1/auth/me
- POST /api/v1/auth/change-password

2. PASSWORD
Use bcrypt or Argon2id.
Never store plaintext passwords.
Never log passwords.
Never return password_hash.

3. JWT
Use:
- short-lived access token
- server-side refresh session
- refresh token rotation
- session revocation

JWT claims should be minimal:
- sub/userId
- roleCode
- sessionId
- iat
- exp

Do not put sensitive data or the full permission list into JWT.

4. AUTH SESSIONS
Create auth_sessions table if not already present.

Fields:
- id
- user_id
- token_hash
- expires_at
- revoked_at
- last_used_at
- ip_address
- user_agent
- created_at
- updated_at

FK:
user_id → users.id
ON DELETE RESTRICT

5. ROLES
Use existing system roles:
- OWNER
- ADMIN
- MECHANIC
- WAREHOUSE

Do not create unrelated roles.

6. PERMISSIONS
Use the permission catalog in GARAGE_PRO_AUTH_RBAC_V1.md.

7. AUTHORIZATION
Create:
- authenticate middleware
- authorize(permission)
- optional authorizeAny/authorizeAll

Backend authorization is mandatory.
Frontend permission checks are UX only.

8. USER MANAGEMENT
Implement:
- list users
- create user
- get user
- update user
- disable user
- enable user
- revoke sessions

Protect last active OWNER.

9. AUDIT
Audit:
- LOGIN_SUCCESS
- LOGIN_FAILED
- LOGOUT
- TOKEN_REFRESH
- PASSWORD_CHANGED
- USER_CREATED
- USER_UPDATED
- USER_DISABLED
- USER_ENABLED
- ROLE_CHANGED
- SESSIONS_REVOKED

Never store:
- password
- password hash
- access token
- refresh token
- token hash

10. SECURITY
Use:
- helmet
- rate limiting
- Zod validation
- secure error handling
- HTTPS-ready cookies
- no secret hardcoding
- no token logging

11. RATE LIMIT
Login endpoint must have brute-force protection.
Default:
5 failed attempts per 15 minutes per IP or equivalent safe strategy.

Do not permanently lock accounts based only on failed attempts.

12. FRONTEND
Implement:
- AuthProvider/Auth state
- LoginPage
- LoginForm
- route guard
- PermissionGate
- useAuth
- usePermission
- API interceptor
- refresh-once behavior
- automatic logout on unrecoverable 401

Prefer:
access token in memory
refresh token in HttpOnly Secure SameSite cookie

13. ERROR HANDLING
401:
- missing token
- invalid token
- expired token
- revoked session
- invalid refresh

403:
- authenticated but permission denied

14. TESTS
Write unit and integration tests for:
- password
- login
- invalid credentials
- disabled account
- JWT
- refresh
- rotation
- logout
- password change
- permission middleware
- role permissions
- last OWNER protection
- rate limiting

15. DEVELOPMENT ADMIN
Use existing seed environment variables.
Never hard-code production credentials.

16. CODE QUALITY
- TypeScript strict
- no unnecessary any
- follow current project structure
- reuse existing error architecture
- reuse existing Sequelize configuration
- do not rewrite scaffold
- do not modify unrelated modules

17. IMPLEMENTATION PROCESS

First:
- inspect current repository
- inspect existing models
- inspect migrations
- inspect API architecture
- inspect frontend routing
- identify what already exists

Then:
- implement missing migration
- implement models
- implement services
- implement middleware
- implement controllers
- implement routes
- implement frontend auth
- implement tests

Do not stop after planning.
Actually modify the repository.

Finally run:
- migration
- seed
- TypeScript typecheck
- backend tests
- frontend build
- relevant integration tests

Return a final report:
1. files created
2. files modified
3. endpoints implemented
4. permissions implemented
5. security controls
6. tests run
7. test results
8. known limitations
9. next recommended phase
```

------------------------------------------------------------------------

# 65. Roadmap Setelah Phase 10

``` text
01 ✅ MASTER DEVELOPMENT SPECIFICATION
02 ✅ DATABASE DESIGN + ERD
03 ✅ API SPECIFICATION
04 ✅ UI/UX SCREEN BIBLE
05 ✅ DESIGN SYSTEM
06 ✅ PROJECT ARCHITECTURE
07 ✅ BUSINESS RULES & WORKFLOW ENGINE
08 ✅ PROJECT SCAFFOLD
09 ✅ DATABASE MIGRATION + SEED DATA
10 ✅ AUTHENTICATION + RBAC
11 🔜 MASTER DATA
12    WORK ORDER ENGINE
13    INVENTORY ENGINE
14    INVOICE + PAYMENT
15    FRONTEND INTEGRATION
16    TESTING
17    DEPLOYMENT
```

------------------------------------------------------------------------

# 66. Next Phase

Setelah Auth/RBAC selesai, phase berikutnya adalah:

**GARAGE_PRO_MASTER_DATA_V1.md**

Cakupan:

``` text
Customer
Vehicle
Mechanic
Service Category
Service
Part Category
Spare Part
Supplier
Warehouse
Warehouse Location
Inspection Item
```

Dengan CRUD, validation, search, pagination, soft delete, import-ready
structure, permission mapping, API specification, frontend screen
behavior, dan Claude Code implementation prompt.
