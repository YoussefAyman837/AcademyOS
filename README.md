# AcademyOS

AcademyOS is a multi-tenant academy management system. The current foundation uses React, TypeScript, Vite, Express, Prisma, and PostgreSQL. The interface is English for now; Arabic/RTL is planned for the UI polish phase.

## Requirements

- Node.js 20.6 or newer and npm
- Docker Desktop with Docker Compose, or a PostgreSQL 17 server

## Local setup

1. Copy `.env.example` to `.env`.
2. Connect to PostgreSQL using Docker Compose, a native PostgreSQL 17 installation, or a hosted PostgreSQL provider (instructions below).
3. Install packages: `npm install`
4. Generate Prisma Client: `npm run db:generate`
5. Apply the development migration: `npm run db:migrate`
6. Start API and web app: `npm run dev`

### Native PostgreSQL on Windows (no Docker)

1. Install PostgreSQL 17 using the installer linked from the [official PostgreSQL Windows download page](https://www.postgresql.org/download/windows/). Keep the command-line tools selected and set a password for the `postgres` database administrator during setup.
2. Open a new PowerShell window and connect to the default database:

   ```powershell
   psql -U postgres -h localhost -d postgres
   ```

3. In the `psql` prompt, create a dedicated local application role and database. `\password` prompts for the password without putting it in shell history:

   ```sql
   CREATE ROLE academyos LOGIN;
   \password academyos
   CREATE DATABASE academyos OWNER academyos;
   ```

4. Exit `psql` with `\q`. In `.env`, set `DATABASE_URL` to use the same password. Prefer an alphanumeric local password to avoid URL-encoding characters:

   ```dotenv
   DATABASE_URL=postgresql://academyos:YOUR_LOCAL_PASSWORD@localhost:5432/academyos?schema=public
   ```

   `POSTGRES_PASSWORD` is used only by Docker Compose and is not needed for the native server.

5. Continue with steps 3–6 above. If `psql` is not recognized after installation, reopen PowerShell or add PostgreSQL's `bin` directory to your user PATH.

### Hosted PostgreSQL with Supabase (no local database install)

1. Create a Supabase project and set its database password.
2. In the dashboard, open **Connect** and copy the **Session pooler** URI for an IPv4 network. Replace its password placeholder with the project password and percent-encode reserved password characters.
3. In **Database settings → SSL Configuration**, download the root certificate and save it as `supabase-root.crt` in the project root.
4. Set `DATABASE_URL` in `.env` to the copied URI with `schema=public`, `sslmode=verify-full`, and `sslrootcert` pointing to the URL-encoded absolute path of `supabase-root.crt`.
5. Use `npm run db:deploy` to apply the committed migration, then continue with steps 3, 4, and 6 above.

Keep the database password private. The root certificate is public, but `DATABASE_URL` contains credentials and must stay in the ignored `.env` file. See Supabase's [PostgreSQL connection guide](https://supabase.com/docs/guides/database/connecting-to-postgres) for connection types and certificate setup.

The Vite server runs at `http://localhost:5173`; the API runs at `http://localhost:3000`. Vite proxies `/api` requests to the API. Check `GET /api/health` for API status.

## Database commands

- `npm run db:generate` regenerates Prisma Client from `apps/api/prisma/schema.prisma`.
- `npm run db:migrate` creates/applies a development migration.
- `npm run db:deploy` applies committed migrations in production.
- `npm run db:validate` validates the Prisma schema using the root `.env`.
- `docker compose down` stops PostgreSQL and keeps its named volume (Docker setup only).
- `docker compose down -v` deletes the local database volume and all local data.

The development seed script is not available yet; demo accounts and seed data remain planned. Do not use seed credentials in production.

## Dashboard and reports

The authenticated dashboard API is `GET /api/analytics/dashboard?from=YYYY-MM-DD&to=YYYY-MM-DD`. Reports are available at `GET /api/reports/{payments|expenses|balances|enrollments|monthly|payment-methods}` with the same optional date filters. All data is resolved from the authenticated tenant and authorized branch scope. Date ranges include both selected dates. Selected ranges filter enrollment sales, payments, expenses, charts, and reports; outstanding balances show the current receivable for the selected branch scope and are not date-limited.

Dashboard cash collections and payment-method totals include verified, non-reversed payments only. Planned expenses do not affect expenses paid or net cash flow. Enrollment sales reflect final agreed prices for enrollments created in the selected range. CSV exports use EGP decimal values and include up to 10,000 records; the screen previews up to 100 rows. Narrow the date range when an export reports the row limit.

## Branches and subscription entitlements

Every academy starts with a **Main Branch** and a one-active-branch entitlement. The owner can open **Branches** to compare each branch’s verified collections, paid expenses, net cash flow, and current outstanding balance, and to create, archive, or reactivate branches within the academy’s `maxBranches` entitlement. Branch staff are assigned through branch memberships; their operational and financial API access is scoped to those branches. The owner can switch between a single branch and all branches using the header selector. Students and courses remain academy records; student details are available to branch staff through the student’s home branch or an enrollment in an assigned branch. Enrollments and expenses belong to a branch; payments and installments inherit the enrollment’s branch.

Plan entitlements are separate from staff roles. This MVP does not include SaaS billing or a self-service plan upgrade screen; an operator must provision plan capacity before an owner can add branches. When a plan is reduced below the current active branch count, history remains intact and the owner must archive branches until the count fits. Archived branches can be reactivated when plan capacity is available.

## Core operations API

After signing in, the API provides tenant-scoped students, courses, enrollments, installments, payments, and expenses. Lists support pagination, search, and status filters. Admins and accountants can create/update operational and financial records; viewers are read-only and cannot access financial transaction modules. Enrollment creation accepts EGP decimal strings and generates a monthly installment schedule in one PostgreSQL transaction. Amounts are stored as integer piastres and serialized as strings to preserve precision. Payments allocate FIFO to the earliest available installments by default; explicit allocation is also supported by the API. Digital transfers stay pending until manual verification. Only verified payments affect installment status or collected cash. Full reversals preserve the original payment and allocation history. Expenses distinguish paid amounts from planned costs. Verified-payment receipt details can be printed or saved as PDF through the browser.

## Checks

- `npm test` runs API unit tests.
- `npm run test:integration` runs auth, tenant foreign-key, operational and financial workflow/isolation, and reporting integration tests against the PostgreSQL database in `.env`.
- `npm run typecheck` type-checks API and frontend workspaces.
- `npm run build` builds the API and frontend.

These commands require `npm install`. The integration tests create and clean up uniquely named temporary records in the configured database. The foundation includes unit tests for validation, role checks, and Argon2id password hashing.

## Authentication and tenant model

Academy registration creates an academy and its initial administrator in a database transaction. Login creates a random opaque session token, stores only its SHA-256 hash, and sends it as an HttpOnly, SameSite=Strict cookie. Set `COOKIE_SECURE=true` over HTTPS; production configuration rejects insecure cookies. Auth endpoints are rate-limited and mutating requests must come from `WEB_ORIGIN`.

Tenant identity is derived from the session's user record. Tenant-owned database relationships include `tenant_id`, with composite foreign keys for related records. Repository and service code must continue to scope every tenant query to the authenticated `tenantId`; never accept a tenant ID from the client as authority.

## Backup and restore

For a local custom-format backup and restore (PowerShell):

```powershell
pg_dump --dbname="$env:DATABASE_URL" --format=custom --file=academyos.dump
pg_restore --dbname="$env:DATABASE_URL" --clean --if-exists academyos.dump
```

Restore into a fresh database when you need to preserve existing data. Production deployments must enable managed PostgreSQL automated backups and verify restores before onboarding customer data.
