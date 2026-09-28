# Household Finance Dashboard

A private, two-user financial dashboard built on the two users' shared [Up Bank](https://up.com.au) 2Up (joint) account(s).

> **Status:** Phases 0–2 complete (planning, stack, environment). Phase 3 (architecture) decisions made; skeleton not yet built. Empty Next.js shell is deployed.
>
> **Open items to confirm:** Tailwind CSS (yes/no), Vercel function region (match Supabase region), whether Vercel auto-deploy stays on.

---

## Overview

This app pulls data from the users' shared 2Up account(s) via the [Up API](https://developer.up.com.au), stores it in its own database, and presents it as a single household dashboard.

- **Users:** 2 (private use only — no public signup)
- **Data source:** Up Bank API (one personal access token, stored encrypted)
- **Hosting budget:** Free tier where possible

---

## Core Features (v1)

- Secure login for the two users
- One Up personal access token registered and stored encrypted (never exposed to the client)
- Scheduled polling sync of accounts and transactions from Up into the app's own database
- Single transaction feed from the shared account(s) — no per-person tagging (Up's data does not identify which user made a purchase)
- **Joint (2Up) accounts only**: spending and saver accounts. Individual accounts and Home Loan accounts are excluded
- Spending categorization using Up's built-in transaction categories

## Out of Scope for v1

- Historical backfill — syncing starts from go-live date onward
- Per-person transaction tagging
- Custom/manual categorization on top of Up's categories
- Budgets, goals, and spending alerts
- Public signup or support for more than 2 users

---

## Tech Stack (Phase 1 — Decided)

| Decision | Choice |
|---|---|
| Frontend framework | Next.js |
| Backend | Next.js API routes (same project as frontend) |
| Language | TypeScript |
| Database | Supabase (Postgres) |
| Auth | Supabase Auth |
| Hosting | Vercel |
| Scheduled sync | Vercel Cron, once-daily (Hobby/free tier limit — see note below) |

**Note on sync frequency:** Vercel's free (Hobby) tier limits cron jobs to once per day with timing accurate only within the scheduled hour. This is an accepted trade-off for a household dashboard where real-time updates aren't required. If more frequent syncing is ever needed, the sync logic itself doesn't need to change — only what triggers it (e.g. an external free scheduler calling the same API route).

---

## Environment & Workflow (Phase 2 — Decided)

| Decision | Choice |
|---|---|
| Repository | GitHub, **private** |
| Git authentication | SSH key |
| Package manager | npm (Node LTS, installed via nvm) |
| Next.js routing | App Router |
| Supabase integration | Framework route: `@supabase/supabase-js` + `@supabase/ssr` (cookie-based sessions) |
| Hosting | Vercel (Hobby/free plan), deployed from GitHub |

### Environment variables

- Real values live in `.env.local` (git-ignored, never committed).
- `.env.example` documents the required variable names with no values and **is** committed (`.gitignore` has an `!.env.example` exception).
- Current variables:
  - `NEXT_PUBLIC_SUPABASE_URL`
  - `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY`
- Only variables prefixed `NEXT_PUBLIC_` are exposed to the browser. Anything sensitive must **not** use that prefix.
- The Supabase **secret key** is not configured yet. It will be added as `SUPABASE_SECRET_KEY` (server-only, no `NEXT_PUBLIC_` prefix) when the daily sync job is built.
- The same variables must also be set in Vercel's project settings for the deployed app.

### Development workflow

- `main` is the live branch. Unfinished work stays on feature branches (e.g. `feature/login`).
- Each feature: branch from `main`, push to the branch, check the Vercel Preview URL, open a pull request, merge to `main` when finished.
- Preview deployments share the single Supabase project (see Phase 3), so avoid destructive database changes while testing on a branch.

---

## Architecture (Phase 3 — Decided)

### Code organization

- Folders organized **by type**: `components/`, `hooks/`, `lib/` (plus `types/`). Supabase and Up API code lives in `lib/`.

### Data model principles

- **Accounts are stored once**, keyed by Up's own account ID. Because both users hold the same 2Up account, both tokens return the same accounts; the sync deduplicates by that ID.
- Transactions link to accounts by Up's account ID. Accounts have no per-user owner.
- The sync requests only joint accounts (filtered by ownership type at the API) and excludes Home Loan accounts.
- **Transactions are upserted by Up's transaction ID.** Up transactions move from held to settled, so a later sync overwrites the earlier state; no change history is kept.
- **Money is stored as integer cents** (never floating point). *(Assumed default — adjustable.)*
- A **sync log table** records each sync run and its outcome. *(Assumed default — adjustable.)*
- Only **one** Up token is stored.

### Access model & security

- Public signups are **disabled** in Supabase Auth; the two users are created manually or by invite.
- **Row-level security is enabled on every table from day one, deny by default.**
- Shared tables (accounts, transactions): any logged-in user may read. This is only safe because signups are disabled.
- No writes to shared tables from the browser. Only the server-side sync writes, using the secret key (which bypasses RLS and must remain server-only).
- The Up token has no browser access. It is encrypted with **Supabase Vault** and only read by server code.
- Database is a managed provider with automated backups enabled.

### Database workflow

- Schema changes are **migration files in the repo** (Supabase CLI), with TypeScript types generated from the database.
- A **single Supabase project** is used (no separate dev database). Because Preview deployments and local testing hit real data, be careful with destructive changes.

---

## Platform

- Responsive web app, usable on both desktop and mobile browsers.
- PWA installability is a possible future enhancement, not a v1 requirement.

---

## Constraints Summary

| Constraint | Decision |
|---|---|
| Sync method | Scheduled polling (once daily) |
| Backfill | None — starts from go-live date |
| Data view | Single feed from shared account(s), no per-person tag |
| Account scope | Joint (2Up) spending and saver accounts only |
| Up tokens stored | One, encrypted in Supabase Vault |
| Categorization | Up's built-in categories |
| Hosting budget | Free tier |
| Platform | Responsive web (mobile-friendly) |