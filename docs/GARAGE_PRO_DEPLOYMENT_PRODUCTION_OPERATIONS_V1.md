# GARAGE PRO --- DEPLOYMENT + PRODUCTION OPERATIONS V1

## Production Infrastructure, CI/CD, Security, Backup, Monitoring & Rollback Specification

**Project:** GARAGE PRO --- Workshop Management System\
**Phase:** 17 --- Deployment + Production Operations\
**Version:** V1.0\
**Status:** Production Implementation Ready\
**Primary stack:** React + Vite + TypeScript / Node.js + Express +
Sequelize + MySQL 8+ / Nginx / PM2

------------------------------------------------------------------------

# 1. PURPOSE

Phase 17 membawa GARAGE PRO dari aplikasi yang sudah selesai
dikembangkan dan diuji menjadi sistem production yang dapat digunakan
secara aman dan stabil oleh bengkel.

Target:

``` text
SOURCE CODE
   ↓
GIT / GITHUB
   ↓
CI
   ↓
BUILD + TEST
   ↓
STAGING
   ↓
QA / UAT
   ↓
PRODUCTION
   ↓
NGINX
   ↓
HTTPS
   ↓
FRONTEND
   ↓
BACKEND
   ↓
MYSQL
   ↓
BACKUP + MONITORING
```

Production environment harus memiliki:

-   predictable deployment
-   secure configuration
-   database backup
-   rollback procedure
-   monitoring
-   logging
-   health checks
-   disaster recovery
-   operational runbook

------------------------------------------------------------------------

# 2. PRODUCTION OBJECTIVES

System production harus:

1.  Dapat diakses melalui HTTPS.
2.  Frontend dapat melayani user.
3.  Backend API stabil.
4.  Database aman.
5.  Migration dapat dijalankan secara controlled.
6.  Process restart otomatis jika crash.
7.  Backup database otomatis.
8.  Log tersedia dan dapat ditelusuri.
9.  Health check tersedia.
10. Deployment dapat diulang.
11. Rollback tersedia.
12. Secrets tidak berada di source code.
13. Production dan staging terpisah.
14. Monitoring dasar tersedia.
15. Recovery procedure terdokumentasi.

------------------------------------------------------------------------

# 3. RECOMMENDED PRODUCTION ARCHITECTURE

V1 dapat menggunakan single application server dengan database server
yang sama untuk skala awal, tetapi struktur harus disiapkan agar dapat
dipisahkan kemudian.

Recommended:

``` text
                         INTERNET
                            │
                            ▼
                    ┌───────────────┐
                    │   CLOUDFLARE  │
                    │ DNS / WAF     │
                    └───────┬───────┘
                            │
                            ▼
                    ┌───────────────┐
                    │     NGINX     │
                    │ HTTPS / Proxy │
                    └───────┬───────┘
                            │
                 ┌──────────┴──────────┐
                 ▼                     ▼
        ┌────────────────┐    ┌────────────────┐
        │ STATIC FRONTEND │    │ NODE API       │
        │ React/Vite      │    │ Express + PM2  │
        └────────────────┘    └───────┬────────┘
                                      │
                                      ▼
                              ┌──────────────┐
                              │    MYSQL     │
                              │  InnoDB 8+   │
                              └──────┬───────┘
                                     │
                                     ▼
                              ┌──────────────┐
                              │ BACKUP STORE │
                              └──────────────┘
```

------------------------------------------------------------------------

# 4. DOMAIN ARCHITECTURE

Recommended domains:

``` text
app.garagepro.example
api.garagepro.example
```

Alternative:

``` text
garagepro.example
api.garagepro.example
```

Frontend:

``` text
https://app.garagepro.example
```

Backend:

``` text
https://api.garagepro.example
```

Do not expose Node.js directly to the public internet if Nginx reverse
proxy is available.

------------------------------------------------------------------------

# 5. ENVIRONMENTS

Minimum:

``` text
LOCAL
STAGING
PRODUCTION
```

## LOCAL

Developer machine.

## STAGING

Production-like environment for:

-   integration test
-   UAT
-   smoke test
-   release validation

## PRODUCTION

Real workshop transaction environment.

Never point automated tests at production.

------------------------------------------------------------------------

# 6. SERVER RECOMMENDATION

For initial single-workshop/small network deployment:

``` text
CPU: 4 vCPU
RAM: 8 GB
SSD: 100+ GB
OS: Ubuntu LTS
```

For larger multi-branch deployment:

``` text
CPU: 8+ vCPU
RAM: 16+ GB
SSD: 200+ GB
```

These are starting recommendations, not absolute requirements. Actual
capacity must be validated with load testing and monitoring.

------------------------------------------------------------------------

# 7. SOFTWARE BASELINE

Recommended:

``` text
Ubuntu LTS
Nginx
Node.js LTS
npm
PM2
MySQL 8+
Git
UFW
Fail2ban optional
Certbot or managed TLS
```

Application:

``` text
React
Vite
Express
Sequelize
TypeScript
```

------------------------------------------------------------------------

# 8. SERVER DIRECTORY STRUCTURE

Recommended:

``` text
/opt/garage-pro/
├── releases/
│   ├── 20260914-001/
│   ├── 20260915-001/
│   └── current -> ...
│
├── shared/
│   ├── logs/
│   ├── uploads/
│   └── .env
│
└── scripts/
```

Alternative simple V1:

``` text
/var/www/garage-pro/
├── frontend/
└── backend/
```

Release-based deployment is preferred because rollback becomes easier.

------------------------------------------------------------------------

# 9. PRODUCTION USER

Do not run application as root.

Recommended:

``` text
deploy
```

or:

``` text
garagepro
```

Application process should run under a non-root user.

------------------------------------------------------------------------

# 10. FIREWALL

Only expose required ports.

Public:

``` text
80
443
```

SSH:

``` text
22
```

preferably restricted by trusted IP/network where practical.

MySQL:

``` text
3306
```

should NOT be publicly exposed.

------------------------------------------------------------------------

# 11. UFW BASELINE

Conceptually:

``` text
allow SSH
allow HTTP
allow HTTPS
deny public MySQL
```

Before enabling firewall remotely, ensure SSH access has been tested.

------------------------------------------------------------------------

# 12. SSH SECURITY

Recommended:

``` text
SSH key authentication
Disable password login where operationally safe
Disable root login
```

Before disabling password authentication, verify working key-based
access.

------------------------------------------------------------------------

# 13. NODE.JS PROCESS

Backend should run through:

``` text
PM2
```

Benefits:

-   restart on crash
-   startup persistence
-   logs
-   process monitoring
-   graceful restart

------------------------------------------------------------------------

# 14. PM2 ECOSYSTEM

Example:

``` js
module.exports = {
  apps: [
    {
      name: "garage-pro-api",
      cwd: "/opt/garage-pro/current/backend",
      script: "dist/server.js",
      instances: 1,
      exec_mode: "fork",
      autorestart: true,
      watch: false,
      max_memory_restart: "500M",
      env: {
        NODE_ENV: "production"
      }
    }
  ]
};
```

For larger deployments, cluster mode can be considered after validating
session/auth, DB pool, and graceful shutdown behavior.

------------------------------------------------------------------------

# 15. PM2 COMMANDS

Typical:

``` bash
pm2 start ecosystem.config.js
pm2 status
pm2 logs garage-pro-api
pm2 restart garage-pro-api
pm2 reload garage-pro-api
pm2 save
pm2 startup
```

Do not blindly use `pm2 reload` if the application does not support
graceful shutdown correctly.

------------------------------------------------------------------------

# 16. BACKEND BUILD

Production build:

``` bash
npm ci
npm run build
```

Expected:

``` text
dist/
```

Do not run TypeScript source directly in production unless that is
explicitly the project architecture.

------------------------------------------------------------------------

# 17. FRONTEND BUILD

Production:

``` bash
npm ci
npm run build
```

Expected:

``` text
dist/
```

Nginx serves static assets.

------------------------------------------------------------------------

# 18. NGINX ROLE

Nginx handles:

``` text
TLS
Static frontend
Reverse proxy
Compression
Security headers
Request size
Access logs
```

Flow:

``` text
Browser
 ↓
Nginx :443
 ├── / → frontend
 └── /api → backend
```

------------------------------------------------------------------------

# 19. NGINX FRONTEND CONFIG CONCEPT

Example:

``` nginx
server {
    listen 443 ssl http2;
    server_name app.garagepro.example;

    root /opt/garage-pro/current/frontend/dist;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location ~* \.(js|css|png|jpg|jpeg|svg|webp|woff2)$ {
        expires 7d;
        add_header Cache-Control "public, immutable";
    }
}
```

Cache strategy must account for Vite hashed assets.

------------------------------------------------------------------------

# 20. NGINX API CONFIG CONCEPT

``` nginx
server {
    listen 443 ssl http2;
    server_name api.garagepro.example;

    location / {
        proxy_pass http://127.0.0.1:3000;

        proxy_http_version 1.1;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_connect_timeout 10s;
        proxy_read_timeout 60s;
    }
}
```

Do not expose port 3000 publicly.

------------------------------------------------------------------------

# 21. HTTPS

Production must use:

``` text
HTTPS only
```

Redirect:

``` text
HTTP → HTTPS
```

Use:

``` text
Let's Encrypt
```

or another trusted certificate provider.

Automate renewal.

------------------------------------------------------------------------

# 22. SECURITY HEADERS

Recommended through Nginx/application:

``` text
Strict-Transport-Security
X-Content-Type-Options
X-Frame-Options
Referrer-Policy
Content-Security-Policy
Permissions-Policy
```

CSP should be introduced carefully and validated against actual frontend
resources.

------------------------------------------------------------------------

# 23. CLOUDFLARE / DNS

If using Cloudflare:

``` text
Domain
 ↓
Cloudflare DNS
 ↓
Server IP
 ↓
Nginx
```

Recommended:

``` text
DNS proxy where appropriate
TLS Full/Strict
WAF rules
rate limiting where useful
```

Never rely on Cloudflare alone for application authorization.

------------------------------------------------------------------------

# 24. DNS RECORDS

Example:

``` text
A / AAAA
app → server
api → server
```

If using IPv6, configure AAAA correctly and verify firewall/Nginx
support.

------------------------------------------------------------------------

# 25. ENVIRONMENT VARIABLES

Backend:

``` env
NODE_ENV=production
PORT=3000

APP_URL=https://app.garagepro.example
API_URL=https://api.garagepro.example

DB_HOST=127.0.0.1
DB_PORT=3306
DB_NAME=garage_pro
DB_USER=garage_pro_app
DB_PASSWORD=<SECRET>

JWT_ACCESS_SECRET=<SECRET>
JWT_REFRESH_SECRET=<SECRET>

LOG_LEVEL=info
```

Additional secrets depend on installed integrations.

------------------------------------------------------------------------

# 26. ENV SECURITY

Never commit:

``` text
.env
production secrets
private keys
database passwords
JWT secrets
API keys
```

Git:

``` gitignore
.env
.env.*
!.env.example
```

Production `.env` should exist only on server/secret manager.

------------------------------------------------------------------------

# 27. DATABASE USER

Do not use MySQL root for application runtime.

Create dedicated:

``` text
garage_pro_app
```

Grant only required privileges.

Migration user can be separate:

``` text
garage_pro_migrator
```

Application user should not have unnecessary schema-altering privileges.

------------------------------------------------------------------------

# 28. MYSQL BASELINE

Recommended:

``` text
MySQL 8+
InnoDB
utf8mb4
```

Verify:

``` text
timezone
sql_mode
max_connections
innodb_buffer_pool_size
slow query log
```

Tune based on server resources rather than blindly copying values.

------------------------------------------------------------------------

# 29. DATABASE CONNECTION POOL

Production pool should be bounded.

Example concept:

``` text
min = 2
max = 10
acquire timeout
idle timeout
```

Actual values depend on:

``` text
server RAM
CPU
MySQL max_connections
application instances
query profile
```

Rule:

``` text
total application DB connections
<
MySQL max_connections
```

with headroom.

------------------------------------------------------------------------

# 30. DATABASE MIGRATION

Production migration flow:

``` text
Backup
 ↓
Check current migration
 ↓
Deploy application compatible with migration
 ↓
Run migration
 ↓
Verify
 ↓
Smoke test
```

Never blindly execute destructive migration against production.

------------------------------------------------------------------------

# 31. MIGRATION RULES

Every migration must:

-   be versioned
-   be reviewed
-   be tested in staging
-   have predictable runtime
-   avoid unnecessary table locks
-   avoid destructive changes without migration plan

For high-risk schema changes:

``` text
expand
→ migrate
→ verify
→ contract
```

------------------------------------------------------------------------

# 32. PRODUCTION DEPLOYMENT ORDER

Recommended:

``` text
1. Backup DB
2. Verify backup
3. Put release in server
4. Install production dependencies
5. Build
6. Run migration if required
7. Switch release
8. Restart/reload backend
9. Reload Nginx if needed
10. Health check
11. Smoke test
12. Monitor
```

------------------------------------------------------------------------

# 33. ZERO/MINIMAL DOWNTIME STRATEGY

For V1 single server:

``` text
prepare new release
 ↓
migration
 ↓
switch symlink
 ↓
restart/reload PM2
```

For schema changes that are not backward compatible, do not deploy
frontend/backend independently.

Use backward-compatible migration strategy.

------------------------------------------------------------------------

# 34. RELEASE DIRECTORY

Example:

``` text
/opt/garage-pro/releases/20260914-001
```

After validation:

``` text
current → releases/20260914-001
```

Rollback:

``` text
current → releases/previous
```

This is safer than overwriting the live directory.

------------------------------------------------------------------------

# 35. DEPLOYMENT SCRIPT

Recommended:

``` text
scripts/
├── deploy.sh
├── rollback.sh
├── backup-db.sh
├── restore-db.sh
├── health-check.sh
└── smoke-test.sh
```

Each script should fail fast.

------------------------------------------------------------------------

# 36. DEPLOY SCRIPT CONCEPT

``` bash
set -euo pipefail

git fetch --all
git checkout "$RELEASE"

npm ci
npm run build

npm run db:migrate

ln -sfn "$RELEASE_DIR" "$CURRENT_LINK"

pm2 reload garage-pro-api

curl --fail https://api.garagepro.example/health
```

Do not run an unreviewed script in production. Adapt paths and commands
to the actual repository.

------------------------------------------------------------------------

# 37. ROLLBACK

Rollback must be documented before first production deployment.

Concept:

``` text
Detect failure
 ↓
Stop further rollout
 ↓
Identify previous known-good release
 ↓
Switch current symlink
 ↓
Reload backend
 ↓
Health check
 ↓
Smoke test
```

------------------------------------------------------------------------

# 38. DATABASE ROLLBACK WARNING

Application rollback and database rollback are not always symmetrical.

Example:

``` text
Release A
 ↓
Migration adds column
 ↓
Release B
```

Rolling application back to A may still work if migration was backward
compatible.

Never assume:

``` text
app rollback = DB rollback
```

For destructive migrations, restore strategy must be explicitly planned.

------------------------------------------------------------------------

# 39. BACKUP STRATEGY

Minimum:

``` text
Daily full database backup
```

Recommended:

``` text
Daily automated backup
Weekly retained backup
Monthly long-term backup
```

Retention depends on business requirements.

------------------------------------------------------------------------

# 40. BACKUP LOCATION

Do not keep the only backup on the same server.

Recommended:

``` text
Production MySQL
      ↓
Local temporary backup
      ↓
Offsite object storage
```

Examples may include S3-compatible storage.

------------------------------------------------------------------------

# 41. MYSQL BACKUP

For V1 logical backup:

``` bash
mysqldump \
  --single-transaction \
  --routines \
  --triggers \
  garage_pro > backup.sql
```

Compress:

``` bash
gzip backup.sql
```

For larger databases, consider MySQL-native physical backup tooling.

------------------------------------------------------------------------

# 42. BACKUP ENCRYPTION

Backup containing customer/financial data must be protected.

Recommended:

``` text
encryption at rest
restricted access
secure credentials
```

Never upload raw production database dumps to public repositories.

------------------------------------------------------------------------

# 43. BACKUP VERIFICATION

A backup is not considered valid merely because the command succeeded.

Regularly:

``` text
restore backup to isolated test DB
 ↓
run integrity checks
 ↓
verify key tables
```

Verify:

``` text
customers
vehicles
work_orders
stock_movements
invoices
payments
audit_logs
```

------------------------------------------------------------------------

# 44. RECOVERY OBJECTIVES

Define:

``` text
RPO = acceptable data loss window
RTO = acceptable recovery time
```

Example starting target:

``` text
RPO ≤ 24 hours
RTO ≤ 4 hours
```

For a business requiring tighter recovery, improve backup frequency and
infrastructure accordingly.

------------------------------------------------------------------------

# 45. DISASTER RECOVERY

If server is lost:

``` text
Provision replacement server
 ↓
Install OS
 ↓
Install Nginx/Node/MySQL/PM2
 ↓
Restore application release
 ↓
Restore database
 ↓
Configure secrets
 ↓
Configure DNS
 ↓
HTTPS
 ↓
Migration verification
 ↓
Smoke test
```

Document every step.

------------------------------------------------------------------------

# 46. MONITORING

Monitor:

``` text
Server uptime
CPU
RAM
Disk
MySQL
PM2
Nginx
API
HTTP errors
Payment errors
Inventory errors
Backup status
```

------------------------------------------------------------------------

# 47. BASIC HEALTH ENDPOINTS

Backend:

``` http
GET /health
GET /health/live
GET /health/ready
```

Suggested:

### live

``` text
process is running
```

### ready

``` text
process running
database available
```

Do not make health endpoint expose secrets or internal configuration.

------------------------------------------------------------------------

# 48. APPLICATION LOGGING

Use structured logging:

``` json
{
  "level": "info",
  "requestId": "req_123",
  "method": "POST",
  "route": "/api/v1/invoices/123/payments",
  "status": 201,
  "durationMs": 142
}
```

------------------------------------------------------------------------

# 49. ERROR LOGGING

Include:

``` text
timestamp
requestId
userId where appropriate
route
method
status
error code
duration
```

Never include:

``` text
password
JWT
refresh token
card data
payment secrets
```

------------------------------------------------------------------------

# 50. LOG ROTATION

Do not allow logs to fill disk.

Use:

``` text
logrotate
```

or PM2 log rotation.

Retention should be defined.

------------------------------------------------------------------------

# 51. ALERTS

Important alerts:

``` text
API 5xx spike
DB unavailable
Disk > 80%
Backup failed
PM2 process down
High memory
High CPU
Financial reconciliation mismatch
Inventory reconciliation mismatch
```

------------------------------------------------------------------------

# 52. DATABASE MONITORING

Track:

``` text
connections
slow queries
locks
deadlocks
buffer pool
disk usage
replication if used
```

------------------------------------------------------------------------

# 53. DEADLOCK HANDLING

Inventory/payment transactions may encounter DB deadlocks under
concurrency.

Application should:

-   keep transactions short
-   lock rows consistently
-   avoid unnecessary queries inside transaction
-   detect deadlock errors
-   retry safe operations where architecture permits

Payment retries must remain idempotent.

------------------------------------------------------------------------

# 54. SECURITY HARDENING

Production checklist:

``` text
[ ] HTTPS
[ ] Firewall
[ ] No public MySQL
[ ] Non-root app
[ ] SSH key
[ ] Secrets outside Git
[ ] Secure headers
[ ] Rate limiting
[ ] RBAC
[ ] Audit
[ ] Backup
[ ] Log rotation
[ ] Dependency updates
```

------------------------------------------------------------------------

# 55. DEPENDENCY MANAGEMENT

Regularly:

``` bash
npm audit
```

Review:

``` text
critical
high
```

Do not blindly update every dependency in production.

Use staging regression testing.

------------------------------------------------------------------------

# 56. NODE PROCESS SECURITY

Node backend should:

-   run as non-root
-   use production environment
-   have bounded memory
-   have graceful shutdown
-   not expose debug endpoints
-   not use development stack traces in responses

------------------------------------------------------------------------

# 57. MYSQL SECURITY

MySQL:

-   bind locally/private network
-   dedicated application user
-   strong password
-   no public port
-   backup credentials protected
-   least privilege
-   audit access where appropriate

------------------------------------------------------------------------

# 58. CORS

Allow only trusted frontend origin:

``` text
https://app.garagepro.example
```

Do not use:

``` text
Access-Control-Allow-Origin: *
```

for authenticated production API unless there is a deliberate
architecture requiring it.

------------------------------------------------------------------------

# 59. COOKIE/TOKEN SECURITY

Follow the authentication architecture already implemented.

If cookies are used:

``` text
Secure
HttpOnly
SameSite
```

If token-based headers are used:

``` text
HTTPS
short-lived access token
secure refresh strategy
```

Do not introduce a second authentication mechanism during deployment.

------------------------------------------------------------------------

# 60. RATE LIMITING

Apply stronger limits to:

``` text
login
password change
payment endpoints
sensitive mutations
```

Do not rate-limit normal read APIs so aggressively that normal workshop
operation becomes painful.

------------------------------------------------------------------------

# 61. REQUEST SIZE

Set reasonable body limits.

Example:

``` text
JSON body 1–2 MB
```

Actual limit should match application needs.

Avoid unlimited request body sizes.

------------------------------------------------------------------------

# 62. TIMEOUTS

Define:

``` text
Nginx proxy timeout
API request timeout
DB acquire timeout
DB query timeout where appropriate
```

Long-running report endpoints should be optimized rather than simply
given extremely long timeouts.

------------------------------------------------------------------------

# 63. FILE STORAGE

If attachments/photos are added later:

``` text
Object Storage
```

preferred over local server disk for scalable deployments.

Local disk can be used for temporary files only.

------------------------------------------------------------------------

# 64. PWA PRODUCTION

Verify:

``` text
manifest
icons
service worker
HTTPS
installability
```

Offline should remain read-only for V1 unless a transaction queue has
been designed and tested.

------------------------------------------------------------------------

# 65. CI/CD

Recommended pipeline:

``` text
Push
 ↓
Install
 ↓
Typecheck
 ↓
Lint
 ↓
Unit
 ↓
Build
 ↓
API tests
 ↓
E2E
 ↓
Security scan
 ↓
Deploy staging
 ↓
Smoke
 ↓
Manual approval
 ↓
Production
```

------------------------------------------------------------------------

# 66. GITHUB ACTIONS CONCEPT

Example pipeline stages:

``` yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - checkout
      - setup-node
      - npm ci
      - npm run typecheck
      - npm run lint
      - npm run test
      - npm run build

  e2e:
    needs: test
    steps:
      - start test environment
      - run Playwright

  deploy-staging:
    needs: e2e
    steps:
      - deploy
      - smoke test
```

Adapt to the repository's actual package structure.

------------------------------------------------------------------------

# 67. PRODUCTION APPROVAL

Recommended deployment policy:

``` text
Developer
   ↓
PR
   ↓
CI
   ↓
Code Review
   ↓
Staging
   ↓
QA/UAT
   ↓
Release Approval
   ↓
Production
```

Do not let a failed CI pipeline deploy production.

------------------------------------------------------------------------

# 68. RELEASE VERSIONING

Use:

``` text
v1.0.0
v1.0.1
v1.1.0
```

or internal release IDs:

``` text
2026.09.14.001
```

Every production deployment must be traceable to:

``` text
Git commit
Build
Migration version
Deployment time
Deployer
```

------------------------------------------------------------------------

# 69. RELEASE MANIFEST

Each release should record:

``` text
version
git_commit
build_time
migration_version
environment
```

Expose a safe version endpoint:

``` http
GET /health/version
```

Example:

``` json
{
  "version": "1.0.0",
  "commit": "abc123",
  "environment": "production"
}
```

Do not expose secrets.

------------------------------------------------------------------------

# 70. PRODUCTION SMOKE TEST

Immediately after deployment:

``` text
1. Open app
2. Login
3. Dashboard
4. Customer search
5. Vehicle search
6. WO search
7. Inventory read
8. Invoice read
9. Payment history read
10. Reports read
11. Logout
```

For controlled production transaction if approved:

``` text
Create/complete a safe operational flow
```

------------------------------------------------------------------------

# 71. PRODUCTION GOLDEN PATH

The most important business test:

``` text
Customer
 ↓
Vehicle
 ↓
WO
 ↓
Inspection
 ↓
Service
 ↓
Part
 ↓
Stock Issue
 ↓
QC
 ↓
READY
 ↓
Invoice
 ↓
Payment
 ↓
PAID
 ↓
COMPLETED
 ↓
Receipt
 ↓
History
 ↓
Report
```

Do this after major production releases according to operational policy.

------------------------------------------------------------------------

# 72. ROLLBACK TRIGGER

Rollback immediately if:

``` text
payment incorrect
inventory corruption
authentication broken
core WO workflow broken
database corruption
5xx severe spike
production startup failure
migration causes critical issue
```

For cosmetic issue:

``` text
do not necessarily rollback
```

Use severity classification from Phase 16.

------------------------------------------------------------------------

# 73. INCIDENT RESPONSE

When incident occurs:

``` text
Detect
 ↓
Assess
 ↓
Contain
 ↓
Communicate
 ↓
Recover
 ↓
Verify
 ↓
Root Cause Analysis
 ↓
Prevent recurrence
```

------------------------------------------------------------------------

# 74. FINANCIAL INCIDENT

If payment anomaly occurs:

``` text
STOP further financial deployment
 ↓
Preserve logs
 ↓
Check payment records
 ↓
Check invoice balances
 ↓
Run reconciliation
 ↓
Identify affected transactions
 ↓
Correct using controlled reversal
```

Never edit production database rows manually without documented
emergency procedure.

------------------------------------------------------------------------

# 75. INVENTORY INCIDENT

If stock mismatch:

``` text
Stop affected mutation if necessary
 ↓
Preserve ledger
 ↓
Run reconciliation
 ↓
Identify movement
 ↓
Verify physical stock
 ↓
Use stock adjustment/opname workflow
 ↓
Audit
```

Do not simply overwrite stock balance.

------------------------------------------------------------------------

# 76. DATABASE EMERGENCY

If DB becomes unavailable:

``` text
Check service
Check disk
Check connection
Check MySQL logs
Check server resources
```

If corruption suspected:

``` text
STOP destructive actions
Preserve evidence
Restore to isolated environment
Validate
Then recover production
```

------------------------------------------------------------------------

# 77. ROLLBACK RUNBOOK

Example:

``` text
1. Declare incident
2. Freeze deployment
3. Identify current version
4. Identify previous known-good version
5. Check migration compatibility
6. Switch application release
7. Reload PM2
8. Health check
9. Smoke test
10. Monitor
11. Reconcile financial/inventory state
12. Document incident
```

------------------------------------------------------------------------

# 78. POST-DEPLOYMENT MONITORING

First 15 minutes:

``` text
5xx
login
API latency
PM2
MySQL
```

First 60 minutes:

``` text
payment
inventory
WO
CPU
RAM
disk
```

First business day:

``` text
revenue
collection
stock
audit
user feedback
```

------------------------------------------------------------------------

# 79. PRODUCTION CHECKLIST

## Infrastructure

``` text
[ ] Server ready
[ ] OS updated
[ ] Firewall
[ ] SSH secured
[ ] Nginx
[ ] Node
[ ] PM2
[ ] MySQL
```

## Application

``` text
[ ] Environment variables
[ ] Frontend build
[ ] Backend build
[ ] Migration
[ ] Seed policy
[ ] Health
```

## Security

``` text
[ ] HTTPS
[ ] CORS
[ ] RBAC
[ ] Rate limit
[ ] Secure headers
[ ] No secrets in Git
```

## Data

``` text
[ ] Backup
[ ] Backup offsite
[ ] Restore tested
[ ] Reconciliation tested
```

## Monitoring

``` text
[ ] Logs
[ ] Rotation
[ ] PM2
[ ] Server monitoring
[ ] DB monitoring
[ ] Backup alerts
```

------------------------------------------------------------------------

# 80. PRODUCTION GO / NO-GO

## GO

Only if:

``` text
QA PASS
UAT PASS
CI PASS
Staging PASS
Backup PASS
Rollback PASS
Security PASS
Migration PASS
Smoke PASS
```

And:

``` text
P0 = 0
P1 = 0
```

## NO-GO

If:

``` text
payment broken
stock integrity broken
auth broken
migration unsafe
backup unavailable
rollback impossible
core E2E failed
```

------------------------------------------------------------------------

# 81. DEPLOYMENT DEFINITION OF DONE

Phase 17 complete when:

``` text
[ ] Production server provisioned
[ ] Domain configured
[ ] HTTPS active
[ ] Nginx configured
[ ] Frontend deployed
[ ] Backend deployed
[ ] PM2 configured
[ ] MySQL configured
[ ] Environment secrets configured
[ ] Migration executed
[ ] Backup automated
[ ] Restore tested
[ ] Monitoring configured
[ ] Logging configured
[ ] CI/CD configured
[ ] Rollback tested
[ ] Smoke test passed
[ ] Production Golden Path passed
```

------------------------------------------------------------------------

# 82. CLAUDE CODE MASTER DEPLOYMENT PROMPT

``` text
You are the DevOps / Production Engineer for GARAGE PRO — Workshop Management System V1.

IMPORTANT:
This is an existing application.
Do not rewrite application business logic.
First inspect the repository and all existing deployment assumptions.

OBJECTIVE:
Prepare GARAGE PRO for secure, repeatable production deployment.

STACK:
Frontend:
- React
- Vite
- TypeScript
- Tailwind

Backend:
- Node.js
- Express
- TypeScript
- Sequelize

Database:
- MySQL 8+

Infrastructure:
- Ubuntu LTS
- Nginx
- PM2
- HTTPS
- GitHub Actions where appropriate

FIRST:
Inspect:
- package.json
- scripts
- frontend build
- backend build
- environment variables
- Sequelize migrations
- health endpoint
- authentication
- logging
- CORS
- production assumptions

DO NOT guess commands.
Use existing package.json scripts when available.

IMPLEMENT/VERIFY:

1. Production environment configuration
2. .env.example
3. frontend production build
4. backend production build
5. health endpoint
6. graceful shutdown
7. PM2 ecosystem
8. Nginx configuration
9. HTTPS readiness
10. database migration process
11. backup script
12. restore script
13. deployment script
14. rollback script
15. smoke test script
16. GitHub Actions CI/CD
17. logging
18. security headers
19. CORS
20. production documentation

ENV:
Never hard-code secrets.

Required backend variables should include as appropriate:
NODE_ENV
PORT
APP_URL
API_URL
DB_HOST
DB_PORT
DB_NAME
DB_USER
DB_PASSWORD
JWT_ACCESS_SECRET
JWT_REFRESH_SECRET
LOG_LEVEL

DATABASE:
Use dedicated application user.
Do not use root for application runtime.
Do not expose MySQL publicly.

NGINX:
Frontend serves static Vite dist.
API reverse proxies to Node.
HTTP redirects to HTTPS.
Forward:
Host
X-Real-IP
X-Forwarded-For
X-Forwarded-Proto

PM2:
- autorestart
- memory restart
- startup persistence
- graceful shutdown
- production env

DATABASE MIGRATION:
Backup first.
Run migrations only after staging validation.
Never execute destructive production migration without explicit safe plan.

BACKUP:
Create automated MySQL backup.
Compress.
Store offsite.
Never commit backups to Git.

RESTORE:
Provide documented restore procedure.
Test restore on isolated database.

DEPLOYMENT:
Prefer release directories:
releases/<version>
current symlink

Flow:
backup
→ deploy
→ install
→ build
→ migrate
→ switch
→ reload
→ health
→ smoke

ROLLBACK:
Support switching current symlink to previous known-good release.
Check database migration compatibility before rollback.

SECURITY:
- HTTPS
- firewall
- no public MySQL
- non-root app
- secure headers
- CORS restricted
- secrets outside Git
- rate limiting
- production error handling
- no sensitive logging

MONITORING:
Health:
GET /health
GET /health/live
GET /health/ready
GET /health/version

Log:
requestId
route
method
status
duration
error code

Never log:
password
tokens
card data
secrets

CI:
Run:
typecheck
lint
unit
build
integration
E2E

Deploy staging after tests.

Production deployment should require approval.

FINAL:
Run all available checks.

Do not claim deployment readiness unless:
- build passes
- tests pass
- migration is valid
- health works
- configuration is documented
- backup exists
- rollback procedure is tested

Output:
1. infrastructure files
2. scripts
3. environment variables
4. Nginx config
5. PM2 config
6. CI/CD
7. backup procedure
8. restore procedure
9. rollback procedure
10. smoke test
11. exact deployment commands
12. security notes
13. unresolved risks

Never expose actual secrets in output.
```

------------------------------------------------------------------------

# 83. DEPLOYMENT RUNBOOK

Production deployment:

``` text
PRE-CHECK
 ↓
CI PASS
 ↓
STAGING PASS
 ↓
UAT PASS
 ↓
DB BACKUP
 ↓
VERIFY BACKUP
 ↓
DEPLOY RELEASE
 ↓
BUILD
 ↓
MIGRATE
 ↓
SWITCH RELEASE
 ↓
PM2 RELOAD
 ↓
HEALTH CHECK
 ↓
SMOKE TEST
 ↓
MONITOR
 ↓
GO LIVE
```

------------------------------------------------------------------------

# 84. DAILY OPERATIONS

Owner/Admin operational check:

``` text
Dashboard
 ↓
Revenue
 ↓
Collection
 ↓
Outstanding
 ↓
WO
 ↓
Low Stock
```

System operator:

``` text
PM2
 ↓
API health
 ↓
DB health
 ↓
Disk
 ↓
Backup
```

------------------------------------------------------------------------

# 85. WEEKLY OPERATIONS

``` text
Review error logs
Review disk
Review backup success
Test selected restore
Review slow queries
Review failed payments
Review inventory reconciliation
Review security updates
```

------------------------------------------------------------------------

# 86. MONTHLY OPERATIONS

``` text
Full backup restore drill
Security review
Dependency review
Database optimization review
User/permission review
Audit review
Server capacity review
Uptime review
Incident review
```

------------------------------------------------------------------------

# 87. CAPACITY EXPANSION PATH

Initial:

``` text
Single Server
```

Next:

``` text
Nginx
   ↓
App Server(s)
   ↓
Dedicated MySQL
```

Then:

``` text
Load Balancer
   ↓
App 1
App 2
App 3
   ↓
Managed/Dedicated MySQL
   ↓
Object Storage
```

Do not introduce distributed architecture before actual traffic requires
it.

------------------------------------------------------------------------

# 88. FINAL PRODUCTION ARCHITECTURE

``` text
                         USERS
                           │
                           ▼
                    ┌─────────────┐
                    │ Cloudflare  │
                    │ DNS / WAF   │
                    └──────┬──────┘
                           │ HTTPS
                           ▼
                    ┌─────────────┐
                    │    NGINX    │
                    └──────┬──────┘
                           │
             ┌─────────────┴──────────────┐
             ▼                            ▼
      ┌──────────────┐             ┌──────────────┐
      │ React/Vite   │             │ Node/Express │
      │ Static Build │             │ PM2          │
      └──────────────┘             └──────┬───────┘
                                         │
                                         ▼
                                  ┌──────────────┐
                                  │ MySQL 8+     │
                                  │ InnoDB       │
                                  └──────┬───────┘
                                         │
                              ┌──────────▼─────────┐
                              │ Backup / Offsite    │
                              └─────────────────────┘

                    ┌────────────────────────────┐
                    │ Monitoring / Logs / Alerts │
                    └────────────────────────────┘
```

------------------------------------------------------------------------

# 89. FINAL GARAGE PRO V1 ROADMAP

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
11 ✅ MASTER DATA
12 ✅ WORK ORDER ENGINE
13 ✅ INVENTORY ENGINE
14 ✅ INVOICE + PAYMENT ENGINE
15 ✅ FRONTEND INTEGRATION
16 ✅ TESTING + QA
17 ✅ DEPLOYMENT + PRODUCTION OPERATIONS
```

GARAGE PRO V1 documentation and implementation roadmap are now complete
from architecture through production operations.

The next logical activity is no longer another blueprint phase, but
**actual implementation in the repository**, followed by executing the
Phase 16 QA suite and Phase 17 deployment runbook against the real
environment.
