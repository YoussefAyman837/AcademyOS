# AcademyOS — Implementation Plan

## Project Status

Current Phase: Phase 5 — Multi-Branch Management (complete); Phase 6 is next.

Database: PostgreSQL

Architecture: Modular Monolith

Frontend: React + TypeScript + Vite

Backend: Node.js + Express + TypeScript

Multi-tenancy: Required

## Phase 0 — Inspection and Planning
- [x] Inspect repository and development environment.
- [x] Confirm Node.js, npm, Docker, and PostgreSQL availability.
- [x] Confirm ORM/database access strategy.
- [x] Propose project structure.
- [x] Design tenant-aware database schema.
- [x] Identify technical blockers.

Inspection result (2026-11-16): repository contains only project instructions and this plan; no README, application source, package manifests, migrations, or tests. Node.js v24.20.0 and npm v11.19.0 are available (`npm.cmd` is needed because PowerShell blocks `npm.ps1`). `psql`, Docker, Docker Compose, Podman, and `pg_ctl` were not found on PATH. Service inspection was unavailable due to Windows Service Control Manager permissions. No Git commits exist yet.

Phase 1 began after the user requested to proceed despite the missing local database tooling. A Docker Compose PostgreSQL service is configured, but migrations and database-backed validation remain blocked until Docker/PostgreSQL is installed and running.

## Phase 1 — Foundation
- [x] Frontend and backend workspace setup.
- [x] PostgreSQL and Docker Compose configuration.
- [x] Environment configuration.
- [x] Database schema and versioned migration authored, Prisma-validated, and applied to hosted PostgreSQL.
- [x] Authentication implementation and database-backed validation (Argon2id, opaque DB sessions, HttpOnly cookie, rate limits).
- [x] Academy onboarding and transaction rollback validated against PostgreSQL.
- [x] Tenant-aware schema relationships, authenticated tenant context, and cross-tenant database constraint tests.
- [x] Role-based access-control middleware.
- [x] API validation and shared error handling.
- [x] Foundation unit tests (6 pass: validation, role checks, Argon2id hashing).
- [x] PostgreSQL integration tests (2 pass: onboarding/session/rollback and cross-tenant foreign-key rejection).
- [x] Type checks and production build pass.
- [x] Dependency audit reports zero vulnerabilities after pinning Prisma 6.12.0 and upgrading Zod to 4.6.7.

Phase 1 is complete. The hosted Supabase connection uses strict TLS verification with its CA certificate, and the initial migration is up to date. API isolation tests for student/payment/report/export endpoints remain part of their respective later phases because those endpoints are not implemented yet. English onboarding and sign-in UI are wired to the real auth endpoints.

## Phase 2 — Core Academy Operations
- [x] Tenant-scoped student API, search, status filters, pagination, details/history, create, update, archive; English list/create/edit/detail/archive UI.
- [x] Tenant-scoped course API, search, status filters, pagination, details, create, update/deactivate; English list/create/edit/detail/deactivate UI.
- [x] Tenant-scoped enrollment API, create with exact price/discount calculation, status/notes update, list/detail, and transactional installment generation; English list/create/edit/detail UI.
- [x] Tenant-scoped paginated installment listing with pending/partial/paid and derived overdue display; English schedule and detail view.
- [x] Exact piastre conversion and installment distribution unit tests.
- [x] PostgreSQL integration tests for cross-tenant list/get/update protection, successful record edits, and enrollment schedule integrity.
- [x] Type checks, unit tests, production build, and PostgreSQL integration tests pass.

## Phase 3 — Financial Operations
- [x] Tenant-scoped payment recording, paginated/searchable history, exact piastre amounts, duplicate-reference protection, and transactional FIFO or explicit installment allocation.
- [x] Digital payment verification/rejection; pending amounts reserve the enrollment balance but only verified payments affect installment status or cash collections.
- [x] Full payment reversal with reason, actor, timestamp, preserved allocation history, explicit status, balance recalculation, and audit trail.
- [x] Tenant-scoped paid/planned expense creation and audited updates; planned expenses are distinct from paid expenses.
- [x] Financial and administrative audit log listing (admin-only) with actor and tenant scoping.
- [x] Printable verified-payment receipt view with browser print-to-PDF, payment details, reference, receiver, and remaining enrollment balance.
- [x] PostgreSQL integration coverage for tenant boundaries, payment allocation/verification/reversal, installment status recalculation, concurrent overpayment prevention, expense status, and audit records.
- [x] Finish UI smoke review of payments, expenses, audit logs, and payment/expense forms in the running preview. Confirmed the available-balance picker displays active enrollments after its API request completes; no records were submitted during review.

Phase 3 implementation and UI review are complete. Current unit tests (9), type checks, and production build pass. The latest PostgreSQL integration rerun could not resolve the configured Supabase pooler hostname (`ENOTFOUND`), so the current environment’s connectivity prevented a fresh integration run; the financial integration suite had passed in the previous connected run.

## Phase 4 — Dashboard and Reports
- [x] Tenant-scoped dashboard metrics for academy size, enrollment sales, verified cash, current receivables, paid expenses, and net cash flow.
- [x] Monthly collections/paid-expense and verified payment-method charts.
- [x] Payments, expenses, outstanding balances, enrollments, monthly financial, and payment-method reports.
- [x] CSV export with EGP decimal output, formula-injection protection, 10,000-row cap, and on-screen preview.
- [x] Inclusive date filters for dashboard and reports; current outstanding balances remain date-independent and are labeled as such.

Phase 4 started: tenant-scoped PostgreSQL/Prisma endpoints, exact integer-piastres summaries, date validation, and the English dashboard/reports UI are in progress. Recharts could not be installed because the npm registry hostname did not resolve; this environment limitation was reported and the chart implementation uses accessible SVG without adding a dependency. A tenant/date index migration was added for filtered enrollment reports.

Phase 4 validation (2026-11-17): Prisma schema validation, TypeScript checks, production build, and 12 unit tests pass. UI smoke review confirmed dashboard metrics/charts and tenant-scoped EGP outstanding balances. After a network-enabled retry, migration `20261117000001_reports_date_index` applied successfully and all 5 PostgreSQL integration tests passed, including tenant isolation and financial report calculations. Phase 4 is complete.

## Phase 5 — Multi-Branch Management (requested)
- [x] Add tenant subscription plan/entitlement data (including `maxBranches`) independently from staff roles and permissions.
- [x] Enforce branch limits in backend branch creation/update services; frontend limit messaging is informational only.
- [x] Keep branch owner access and staff permissions role-based; do not grant paid features by marking a user privileged.
- [x] Define safe downgrade behavior: never delete branch history; require the owner to select which branches remain active when above the new limit.
- [x] Add branch entity and backfill each existing academy into one default branch without losing data.
- [x] Attribute enrollments and expenses to a branch; derive payment and installment branch ownership through the enrollment.
- [x] Add branch membership and branch-scoped permissions while preserving tenant-level isolation.
- [x] Give the academy owner an all-branches overview with branch-by-branch verified collections, paid expenses, and net cash flow.
- [x] Add branch details and branch filters to relevant dashboards and financial reports.
- [x] Test cross-branch access restrictions and owner visibility, including financial integrity and exports.

Requested before continuing Phase 4. Default product interpretation: the owner gets a central **Owner Overview** that compares and opens each branch; this is an owner panel, not a special accounting branch. If the academy has a physical headquarters that records its own transactions, it can also be created as a normal branch. Student and course records remain academy-wide; branch visibility for staff is constrained through their branch membership and associated enrollments. Commercial access is represented by tenant subscription entitlements (for example, one included branch plus paid additional branch capacity); user roles remain separate. Self-service SaaS billing is a later decision; this project specification excludes payment-gateway integration for academy collections.

Phase 5 implementation (2026-11-18): added the branch/subscription/membership schema and applied migration `20261118000000_multi_branch`, backfilling a Main Branch for existing academies and assigning prior records without deleting history. New academies receive a one-branch entitlement. Branch creation/reactivation is transactionally limited by `maxBranches`; branch archival preserves records and membership history, and an academy cannot archive its last active branch. Enrollment/expense ownership and branch-derived payment/installment/report scoping are enforced on the API. Owners can compare branch collections, paid expenses, receivables, and cash flow, switch branch scope, and manage branch staff. Users’ roles remain independent of plan capacity. No SaaS billing or self-service entitlement upgrade screen is included. Final validation: schema validation, TypeScript checks, unit tests (12), production build, and PostgreSQL integration tests (6) pass.

## Phase 6 — Final Validation
- [ ] Arabic RTL review.
- [ ] Responsive design.
- [ ] Security testing.
- [ ] Tenant isolation testing.
- [ ] Financial integrity testing.
- [ ] End-to-end testing.
- [ ] Backup and restore verification.
- [ ] Production readiness review.

## Working Rules

- Complete phases sequentially.
- Do not start a new phase before validating the previous one.
- Update this file after completing each meaningful task.
- Mark tasks complete only after implementation and testing.
- Record blockers explicitly.
- Do not repeat completed work unless a defect requires it.
