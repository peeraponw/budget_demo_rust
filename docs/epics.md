# budget_demo_rust - Epic Breakdown

**Date:** 2025-12-07
**Project Level:** Quick-Flow Greenfield

---

## Epic 1: Budgeting MVP

**Slug:** budgeting-mvp

### Goal

Deliver a contract-first budgeting product with Rust API + React web that supports categories, monthly budgets, transactions, and a variance summary, using shared OpenAPI-derived types.

### Scope

- Categories, budgets (per month), transactions CRUD
- Variance summary per month
- OpenAPI spec + generated TS client
- React UI for lists/forms + variance view
- SQLite default, Postgres-ready configuration

### Success Criteria

1. API exposes documented endpoints for categories, budgets, transactions, variance with OpenAPI served at /api-docs/openapi.json.
2. Duplicate budget month/category returns conflict; variance math matches stored data using integer cents.
3. React app consumes generated TS client, lets users create/list entities, and shows variance for selected month with loading/error states.
4. Contract regeneration produces no drift (CI fails on diff); lint/test pipelines pass.

### Dependencies

- None external beyond SQLite/Postgres and Node/Rust toolchains; relies on openapi-typescript generator and sqlx migrations.

---

## Story Map - Epic 1

Epic: Budgeting MVP (9 points)
├── Story 1.1: API foundation & schema (3 points)
│   Dependencies: none
├── Story 1.2: Contracts & web data layer (3 points)
│   Dependencies: Story 1.1 (needs running API + spec)
└── Story 1.3: UI flows for budgets & transactions (3 points)
    Dependencies: Stories 1.1, 1.2 (needs client + API)

---

## Stories - Epic 1

### Story 1.1: API foundation & schema

As a developer,
I want a working Axum API with validated models and migrations for budgets/categories/transactions,
So that the product has a reliable backend with documented contract.

**Acceptance Criteria:**

**Given** migrations are applied, **When** the API starts, **Then** healthz returns 200 and tables exist for categories, budgets, transactions.
**Given** a duplicate budget (same category + month) is posted, **When** the request is processed, **Then** the API returns 409 with code `duplicate_budget_month`.
**Given** transactions exist, **When** GET /api/variance?month=YYYY-MM is called, **Then** it returns per-category and total budget/spent/remaining in integer cents with overspend flag.
**Prerequisites:** none
**Technical Notes:** implement sqlx migrations, services, routes with utoipa annotations; return structured errors; include Swagger UI.
**Estimated Effort:** 3 points (time estimate not provided by policy)

---

### Story 1.2: Contracts & web data layer

As a front-end developer,
I want generated TypeScript types/clients from the OpenAPI spec and a shared fetch wrapper,
So that the web app can call the API safely without drift.

**Acceptance Criteria:**

**Given** the API is running, **When** `pnpm turbo run generate-contracts` runs, **Then** packages/contracts/src/index.ts regenerates with no manual edits and matches the live spec.
**Given** the web client calls categories/budgets/transactions endpoints, **When** responses return, **Then** they are typed through the generated client and handled via React Query with loading/error states.
**Given** CI runs, **When** contracts are outdated, **Then** the pipeline fails on diff.
**Prerequisites:** Story 1.1 API and OpenAPI spec available
**Technical Notes:** configure openapi-typescript 7.9.1; add fetch wrapper; wire React Query provider; add pnpm/turbo tasks for generate-contracts.
**Estimated Effort:** 3 points (time estimate not provided by policy)

---

### Story 1.3: UI flows for budgets & transactions

As a user,
I want to create and review categories, budgets, and transactions and see a monthly variance summary,
So that I can understand spending against budget.

**Acceptance Criteria:**

**Given** the app is running, **When** I open Categories/Budgets/Transactions pages, **Then** I can view tables, create new records with required validation, and see success/error toasts.
**Given** I select a month on the variance page, **When** data loads, **Then** I see per-category budget, spent, remaining, and overspend flag along with totals; loading and empty states are handled.
**Given** pagination is needed for transactions, **When** I load the page, **Then** it shows a paginated list (default 50) sorted by date desc and supports next/prev.
**Prerequisites:** Stories 1.1 and 1.2
**Technical Notes:** build pages with generated client + React Query; forms with zod/yup-free manual validation; components FormField/Table/Layout; routing via react-router-dom; simple CSS modules.
**Estimated Effort:** 3 points (time estimate not provided by policy)

---

## Implementation Timeline - Epic 1

**Total Story Points:** 9

**Estimated Timeline:** not provided (time estimates prohibited by policy)

