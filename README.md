# Multi-Store Purchase Tracker — Railway Test Deployment

This is a **self-hosted, multi-store test build** of the Version 3 Purchase Tracker. It uses **Railway hosting + Railway PostgreSQL**, **no Supabase**, and no separate paid platform. It is *not already deployed* and it has *not yet been tested against a live Railway database*. Validate it with sample invoices before using live accounting data.

## Capabilities

- Owner/master dashboard: all stores, invoices, employee management, combined reports.
- Employee dashboard: only stores assigned to that employee; those limits are checked by the backend on *every* database request.
- Employees may enter, edit, delete, and mark their own store invoices paid/unpaid.
- 30-day payment terms (customizable), payment date, searchable invoices, monthly views by purchase/due/payment date, yearly totals.
- Suppliers seeded for new stores from the original template; owner and employees may manage suppliers of accessible stores.
- CSV/JSON download, creation/edit/deletion audit history, browser desktop-shortcut support.
- Sign-in sessions use secure HTTP-only cookies; passwords are salted and hashed.

## Set up with your existing Railway Hobby account

1. Unzip the project. Upload its contents to a **private GitHub repository**. Do not upload `.env` or your passwords. `node_modules` is not part of this bundle.
2. In Railway, create a **new project** from the GitHub repository.
3. Add a **PostgreSQL** service to that same project (Railway's database template). Allow it to provision.
4. Open the **web application** service and set these Railway **Variables**:
   - `DATABASE_URL` = `${{Postgres.DATABASE_URL}}` (use Railway's **Add Reference** interface; service name may differ from `Postgres`).
   - `OWNER_EMAIL` = your private master account email address.
   - `OWNER_PASSWORD` = a unique, randomly generated password **at least 16 characters**.
   - `NODE_ENV` = `production`.
5. Generate a Railway public domain for the **web app** under Networking. The Postgres database should **not** have a public domain.
6. Deploy the app. A healthy deployment returns `{ "ok": true }` at `/health` and shows the sign-in page at `/`.
7. Sign in with your configured owner email/password. The *first* app startup creates that owner account. Modifying Railway `OWNER_PASSWORD` **after** the owner is created will not change the existing password.
8. In **Settings**, add the first store (e.g. `Store 1 – Parkville`); the initial supplier list is created automatically.
9. Add employees from **Settings → Employee access**. Each employee gets an email, temporary password, and at least one assigned store. Share their password privately and ask them to change it after signing in (see password note below).
10. On each store's PC, open the **same** Railway URL in Chrome or Edge and use the browser menu **Create shortcut / Install this site as an app** (menu wording varies). Employees sign in to their own accounts.

> **Important password note:** The app includes an authenticated `/api/change-password` endpoint, but the browser version currently does not expose an employee self-service password change form. The owner can reset an employee password in Settings. Add a proper change-password screen before rollout to employees with sensitive invoice data.

## Cost control

Railway Hobby's **$5 monthly subscription includes $5 of resource usage**, shared by all services in the account. Running both a web service and PostgreSQL might consume the allowance or exceed it. Check Railway **Usage/Billing** daily during testing. Consider stopping the deployment when not testing. This bundle makes **no** promise of zero overage and **does not** set a spending cap for you.

## Data safety and migration

- Existing Version 3 invoices stored in browser `localStorage` **will not automatically migrate** into the hosted app. Export JSON from Version 3 first. This package supports exporting hosted data but **does not yet implement automatic importing of that V3 JSON**; enter test invoices manually. Plan a carefully verified migration before going live.
- PostgreSQL is persistent on Railway when correctly configured, but **do not rely on persistence alone as a backup**. CSV and JSON exports are manual *copies*, not an automatic backup or tested restore process.
- This release has **no scheduled offsite backup/restore workflow**. Before production, configure regular database backups and test restoring them. The app retains an audit trail of deleted invoices, but that is not a substitute for backups.
- A user can delete an invoice at their own store as requested. Its deletion appears in the audit log.
- The backend returns up to 20,000 most recent invoices per accessible owner account; larger stores need server-side pagination.
- Add password-change UX, stronger session controls, rate limiting in shared persistent storage, concurrent-edit handling, backup restore testing, and access-control integration tests before serious production use.

## Local developer check

Requirements: Node.js 20+, PostgreSQL available.

```
npm install
cp .env.example .env     # fill in your own credentials, keep .env private
npm run check
npm start
```

Environment variables are not automatically loaded from `.env` by Node itself; use a shell export or a dotenv runner for local testing. Railway injects the Variables automatically.

## Security model

`users`, `stores`, `memberships`, `companies`, and `invoices` are separate tables. The backend verifies that each employee belongs to the requested store for **reads and mutations**. Owner-only endpoints manage stores and users. There is no open public user registration. Sessions are random tokens stored hashed in PostgreSQL, with HTTP-only cookies. HTTPS is supplied by Railway.

This is a first deployable **test build**. Review security and data recovery before relying on it for live business bookkeeping.
