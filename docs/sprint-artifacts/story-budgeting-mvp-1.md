# Story 1.1: API foundation & schema

**Status:** Draft

---

## User Story

As a developer,
I want a working Axum API with validated models and migrations for budgets, categories, and transactions,
So that the product has a reliable backend with a documented contract.

---

## Acceptance Criteria

**Given** migrations are applied, **When** the API starts, **Then** `/healthz` returns 200 and tables exist for categories, budgets, and transactions.
**Given** a duplicate budget (same category + month) is posted, **When** the request is processed, **Then** the API returns 409 with code `duplicate_budget_month`.
**Given** transactions exist, **When** GET `/api/variance?month=YYYY-MM` is called, **Then** it returns per-category and total budget/spent/remaining in integer cents with an overspend flag.

---

## Implementation Details

### Tasks / Subtasks

- [ ] Initialize Cargo workspace and api crate with dependencies (axum, sqlx, utoipa, tracing). (AC: all)
- [ ] Add config loader (DATABASE_URL, PORT, RUST_LOG) with defaults for SQLite. (AC: 1)
- [ ] Write migrations 0001 for categories, budgets, transactions with constraints and indexes. (AC: 1,2,3)
- [ ] Implement sqlx models/services for CRUD and variance aggregation. (AC: 2,3)
- [ ] Add routes: healthz, categories, budgets, transactions, variance; wire router. (AC: all)
- [ ] Derive OpenAPI schemas/paths and serve JSON at /api-docs/openapi.json plus Swagger UI. (AC: all)
- [ ] Add error mapping to JSON envelope with proper status codes. (AC: 2)
- [ ] Add tracing middleware and request logging. (AC: 1)

### Technical Summary

Axum 0.8 router with tower middlewares; sqlx 0.8.6 pools for SQLite/Postgres; migrations via `sqlx migrate`; utoipa 5.4 for OpenAPI; structured errors via thiserror; tracing subscriber for logs.

### Project Structure Notes

- **Files to modify:** Cargo.toml; apps/api/Cargo.toml; apps/api/src/{main.rs,config.rs,db/mod.rs,models/*,services/*,routes/*,errors.rs,openapi.rs}; apps/api/migrations/0001_init.sql
- **Expected test locations:** apps/api/tests or module tests in services/routes
- **Estimated effort:** 3 story points (time estimate not provided by policy)
- **Prerequisites:** none

### Key Code References

Set by this story (no prior code). Key pieces to highlight after implementation: router in main.rs; variance query in services/variance.rs; migrations in 0001_init.sql.

---

## Context References

**Tech-Spec:** [tech-spec.md](../tech-spec.md) – context, stack, schema, acceptance criteria.

**Architecture:** See sections "Technical Approach" and "Technical Details" in tech-spec.md.

---

## Dev Agent Record

### Agent Model Used

### Debug Log References

### Completion Notes

### Files Modified

### Test Results

---

## Review Notes

