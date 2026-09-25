# Household Finance Dashboard

A private, two-user financial dashboard that combines transaction and balance data from both users' [Up Bank](https://up.com.au) accounts into a single household view.

> **Status:** Planning (Phase 0 & 1 complete). No implementation yet.

---

## Overview

This app pulls financial data from each user's own Up Bank account via the [Up API](https://developer.up.com.au) and presents it as one combined dashboard, so two people can see their shared household finances in one place — without merging bank accounts or sharing bank credentials with each other.

- **Users:** 2 (private use only — no public signup)
- **Data source:** Up Bank API (personal access token per user)
- **Hosting budget:** Free tier where possible

---

## Core Features (v1)

- Secure login for the two users
- Each user registers their own Up personal access token (stored encrypted, never exposed to the client)
- Scheduled polling sync of transactions and balances from Up into the app's own database
- Combined transaction feed — both users' transactions merged into one view, each tagged by owner
- All accounts included per user (everyday spending + all savers)
- Spending categorization using Up's built-in transaction categories

## Out of Scope for v1

- Historical backfill — syncing starts from go-live date onward, not full account history
- Custom/manual categorization on top of Up's categories
- Budgets, goals, and spending alerts
- Public signup or support for more than 2 users

---

## Data Model Notes

- Transactions are stored in a single shared table/collection, tagged with the owning user, rather than kept in separate per-user datasets.
- No aggregation logic is applied beyond combining feeds — categorization and transaction detail come directly from Up.

---

## Security Requirements

Since this app handles real financial account access, the following are treated as hard requirements, not optional hardening:

- Up personal access tokens are **encrypted at rest** in the database.
- Tokens and raw financial data are **never sent to or stored in the client/browser** — all Up API calls happen server-side.
- Database is hosted with a **managed provider** (not self-hosted) for patching, access control, and backups.
- **Automated backups** are enabled at the database level.

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

## Platform

- Responsive web app, usable on both desktop and mobile browsers.
- PWA installability is a possible future enhancement, not a v1 requirement.

---

## Constraints Summary

| Constraint | Decision |
|---|---|
| Sync method | Scheduled polling |
| Backfill | None — starts from go-live date |
| Data view | Merged feed, tagged by user |
| Account scope | All accounts per user |
| Categorization | Up's built-in categories |
| Hosting budget | Free tier |
| Platform | Responsive web (mobile-friendly) |