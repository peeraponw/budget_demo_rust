# budget_demo_rust - Technical Specification

**Author:** Warm
**Date:** 2025-12-07
**Project Level:** Quick-Flow Greenfield
**Change Type:** Budgeting MVP (Rust API + React web, contract-first)
**Development Context:** Greenfield monorepo; Axum 0.8 + sqlx 0.8.6 + utoipa 5.4.0 backend; pnpm 10 + Turborepo 2.2 + Vite 6 + React 18 frontend; story_count = 3.

---

## Context

### Available Documents

- Research: docs/bmm-research-technical-2025-12-06.md — recommends Axum 0.8 + sqlx 0.8.6 + utoipa 5.4.0, SQLite→Postgres, OpenAPI-driven TS types, pnpm 10 + Turborepo 2.2 + Vite 6.
- Product brief: not found (no docs/*brief*.md or shard)
- Brownfield map/index: not found (no docs/index.md); field_type = greenfield
- Other inputs: none detected

### Project Stack

- Current manifests: none detected (no Cargo.toml, package.json, pnpm-lock.yaml, pyproject, etc.) — fresh repo
- Recommended stack (from research 2025-12-06): Axum 0.8 + sqlx 0.8.6 + utoipa 5.4.0 with SQLite dev / Postgres prod; OpenAPI → TS via openapi-typescript 7.9.1; JS toolchain pnpm 10 + Turborepo 2.2 + Vite 6 + React 18; monorepo with shared contracts package
- Decision status: not yet initialized; finalize with starter selection and manifest creation

### Existing Codebase Structure

Greenfield project – no existing codebase or conventions yet. Need to establish directory layout, code style, test setup, and tooling during implementation.

---

## The Change

### Problem Statement

We need a first-release budgeting MVP that supports categories, monthly budgets, transactions, and a variance view. It must expose a typed REST API with an OpenAPI contract that drives generated TypeScript clients for the React front end. Ops footprint must stay minimal (single container, SQLite locally with a Postgres path). No authentication is required for the MVP; focus on correctness, data integrity, and fast developer feedback.

### Proposed Solution

Stand up a monorepo with a Rust API (Axum + sqlx) that serves REST endpoints documented via utoipa/OpenAPI, persists to SQLite by default, and can switch to Postgres through configuration. Generate a TS client with openapi-typescript and consume it in a React app (Vite, Turborepo, pnpm). Deliver baseline UI screens for categories, budgets, transactions, and a monthly variance summary using the generated client. Enforce shared types and schema-first development to prevent drift.

### Scope

**In Scope:**

- Monorepo scaffolding (Cargo workspace + pnpm/turbo workspace)
- Database schema for categories, budgets (per month), transactions
- REST endpoints: health, categories CRUD, budgets CRUD, transactions CRUD, monthly variance summary
- OpenAPI generation and TS client package
- React web app with pages to view/create categories, budgets, transactions, and monthly variance
- Basic validation and error handling (required fields, amount/date formats)
- Env-configurable database (SQLite default, Postgres optional)

**Out of Scope:**

- Authentication/authorization and multi-tenant roles
- File import/export (CSV/QIF), bank sync, or webhook ingestion
- Recurring budgets, rollovers, or scheduled tasks
- Advanced analytics, charts beyond simple totals, or notifications
- Mobile app, offline mode, and accessibility hardening beyond semantic HTML and basic ARIA
- Localization/i18n and multi-currency support

---

## Implementation Details

### Source Tree Changes

- CREATE Cargo.toml (workspace) and rust-toolchain.toml at repo root; include members `apps/api`.
- CREATE apps/api/Cargo.toml with dependencies: axum = "0.8", tokio = { version = "1.38", features = ["full"] }, sqlx = { version = "0.8.6", features = ["runtime-tokio", "sqlite", "postgres", "macros", "uuid", "chrono"] }, utoipa = "5.4", utoipa-swagger-ui = "7.1", serde/serde_json = "1.0", thiserror = "1.0", tracing = "0.1".
- CREATE apps/api/src/main.rs to wire router, tracing, DB pool, health route, and Swagger UI.
- CREATE apps/api/src/config.rs to read env (DATABASE_URL, RUST_LOG, PORT) with defaults for SQLite file `data/budget.db`.
- CREATE apps/api/src/db/mod.rs plus migrations folder with SQLx migrations.
- CREATE apps/api/migrations/0001_init.sql defining tables: categories (id uuid, name unique), budgets (id uuid, category_id FK, month date, amount_cents integer, UNIQUE(category_id, month)), transactions (id uuid, category_id FK nullable, occurred_at timestamptz, description text, amount_cents integer, source text).
- CREATE apps/api/src/models/{category.rs,budget.rs,transaction.rs,variance.rs} with serde + utoipa schemas.
- CREATE apps/api/src/routes/{health.rs,categories.rs,budgets.rs,transactions.rs,variance.rs} exposing REST handlers; mount under /api.
- CREATE apps/api/src/services/{categories.rs,budgets.rs,transactions.rs,variance.rs} encapsulating DB logic using sqlx queries with compile-time checking.
- CREATE apps/api/src/errors.rs defining ApiError + IntoResponse mapping.
- CREATE apps/api/src/openapi.rs registering components and paths for utoipa.
- CREATE apps/api/Makefile (optional) with tasks: fmt, lint (cargo clippy -- -D warnings), test, dev (cargo watch -x run).
- CREATE packages/contracts/package.json (type: module) with scripts `generate` (fetch OpenAPI JSON from api, run openapi-typescript 7.9.1), `build` (tsc).
- CREATE packages/contracts/src/index.ts generated output (git-ignored, regenerated) and a stub README.md describing regeneration steps.
- CREATE apps/web/package.json with pnpm workspace, React 18, Vite 6, TypeScript 5.3, eslint 8.56, prettier 3; add scripts dev/build/test/lint.
- CREATE apps/web/vite.config.ts, tsconfig.json, src/main.tsx, src/App.tsx.
- CREATE apps/web/src/api/client.ts using fetch + generated contracts types.
- CREATE apps/web/src/pages/{Categories.tsx,Budgets.tsx,Transactions.tsx,Variance.tsx} with forms + tables powered by generated client; add simple router in src/routes.tsx.
- CREATE apps/web/src/components/{FormField.tsx,Table.tsx,Layout.tsx} for consistent UI.
- CREATE turbo.json defining pipelines: lint, test, build, generate-contracts; pnpm-workspace.yaml at root to include apps/* and packages/*.

### Technical Approach

- Runtime: Rust 1.83 (stable) for API; Node.js 22 for tooling/front-end.
- API framework: Axum 0.8 with tower middleware; OpenAPI via utoipa 5.4 + Swagger UI via utoipa-swagger-ui 7.1.
- Data layer: sqlx 0.8.6 with SQLite (dev) and Postgres (prod) feature flags; migrations managed by `sqlx migrate`.
- Serialization/validation: serde for DTOs; basic input validation in handlers; amount stored as integer cents; dates as chrono `NaiveDate`/`DateTime<Utc>`.
- Error handling: thiserror + IntoResponse to emit consistent JSON error envelope `{code,message,details?}`; map SQLx errors to 400/404/409/500.
- Logging/observability: tracing subscriber with env-filter; request/response logging middleware; health at `/healthz`.
- OpenAPI contract: derive schemas on models; annotate routes; serve spec at `/api-docs/openapi.json`; Swagger UI at `/api-docs`.
- Generated TS client: openapi-typescript 7.9.1 outputs `packages/contracts/src/index.ts`; consumed by React via lightweight fetch wrapper in `apps/web/src/api/client.ts`.
- Frontend: React 18 + Vite 6 + TypeScript 5.3; UI built with minimal custom components and CSS modules; state via React Query (v5) for API calls and caching.
- Tooling: pnpm 10 + Turborepo 2.2 for task orchestration and caching; ESLint + Prettier + TypeScript strict mode; rustfmt + clippy with `-D warnings`.

### Existing Patterns to Follow

Greenfield conventions to adopt:
- Rust: snake_case modules, Result-returning handlers, tracing spans per request, error type per domain module.
- Database: UUID primary keys (sqlx `uuid` feature), integer cents for money, `CHECK(amount_cents != 0)`, UTC timestamps, foreign keys with ON DELETE SET NULL for transactions.category_id.
- HTTP: JSON only, verbs aligned to resources (GET/POST/PATCH/DELETE), idempotent PATCH for partial updates.
- TypeScript: ESLint airbnb-ish rules with Prettier formatting; path aliases via tsconfig `@contracts/*` to `packages/contracts/src`.
- Testing: cargo test for Rust; vitest + testing-library for React; keep test files alongside code (`*.test.ts[x]`).

### Integration Points

- Database connections via `DATABASE_URL`; default `sqlite://data/budget.db`; Postgres supported when `postgres` feature enabled and DSN provided.
- OpenAPI spec served at `/api-docs/openapi.json`; contracts generation consumes this endpoint.
- Frontend consumes TS client from `packages/contracts`; fetch base URL from `VITE_API_BASE` env.
- Logging to stdout; optional JSON logs controlled by `RUST_LOG`.
- Health endpoint `/healthz` for liveness/readiness probes.

---

## Development Context

### Relevant Existing Code

None yet; greenfield scaffolding will establish patterns listed above.

### Dependencies

**Framework/Libraries:**
- Axum 0.8, sqlx 0.8.6, utoipa 5.4.0, utoipa-swagger-ui 7.1, tokio 1.38, tracing 0.1, serde 1.0, chrono 0.4, uuid 1.7.
- React 18.3, Vite 6, TypeScript 5.3, React Query 5, eslint 8.56, prettier 3.1, vitest 1.6, @testing-library/react 14.
- Tooling: pnpm 10, Turborepo 2.2, openapi-typescript 7.9.1.

**Internal Modules:**
- `apps/api/src/routes/*` for HTTP endpoints.
- `apps/api/src/services/*` for DB operations.
- `apps/api/src/models/*` for DTOs and OpenAPI schemas.
- `packages/contracts` for generated API client types.
- `apps/web/src/api/client.ts` fetch wrapper consuming contracts.

### Configuration Changes

- `.env` at repo root with `DATABASE_URL`, `PORT` (default 3000), `RUST_LOG=info`, `VITE_API_BASE=http://localhost:3000`.
- `sqlx-data.json` generated for offline compile checks (commit to repo).
- `pnpm-workspace.yaml` and `turbo.json` define pipelines `lint`, `test`, `build`, `generate-contracts`.

### Existing Conventions (Brownfield)

Not applicable; adopting conventions defined above for new codebase.

### Test Framework & Standards

- Rust: `cargo test` with sqlx test helpers using in-memory SQLite; aim for table-level tests plus handler integration tests hitting axum router with test client.
- JS/TS: `pnpm test` running vitest with jsdom; component tests with testing-library; API hooks tested with mocked fetch using MSW.
- Linting: `cargo clippy -D warnings`; `pnpm lint` (eslint) must pass; format via rustfmt and prettier.

---

## Implementation Stack

- Runtime: Rust 1.83, Node.js 22
- Backend: Axum 0.8, sqlx 0.8.6, utoipa 5.4.0, tokio 1.38
- Database: SQLite (dev) with Postgres-ready migrations
- Frontend: React 18.3, Vite 6, React Query 5, TypeScript 5.3
- Tooling: pnpm 10, Turborepo 2.2, openapi-typescript 7.9.1, eslint 8.56, prettier 3.1, vitest 1.6

---

## Technical Details

- Schema design: categories (id, name unique), budgets (id, category_id, month, amount_cents, UNIQUE(category_id, month)), transactions (id, category_id nullable, occurred_at, description, amount_cents signed, source). Add view/query for monthly variance: sum of transactions per category per month vs budget; compute remaining and overspend flag.
- Money handling: store integer cents; parse/format on API boundary; prohibit zero amounts; accept negative for refunds/credits.
- API surface:
  - GET /healthz
  - GET/POST /api/categories, PATCH/DELETE /api/categories/:id
  - GET/POST /api/budgets, PATCH/DELETE /api/budgets/:id
  - GET/POST /api/transactions, PATCH/DELETE /api/transactions/:id
  - GET /api/variance?month=YYYY-MM: returns per-category and overall totals {budget_cents, spent_cents, remaining_cents, overspend: bool}
- Error model: {code:string, message:string, details?:object}; 400 validation, 404 not found, 409 conflict (unique month/category), 500 unexpected.
- Security: no auth for MVP; still validate input; enable CORS for Vite dev origin.
- Performance considerations: use prepared statements via sqlx; indices on (category_id, month) and occurred_at; paginate transaction list (limit+offset, default limit 50).
- Observability: tracing layers for request IDs; log route, status, latency.

---

## Development Setup

1) Install Rust 1.83 (rustup default) and Node.js 22 (fnm/nvm).
2) `pnpm install` (after package manifests are created).
3) `cargo sqlx prepare -- --lib` (generates sqlx-data.json after DATABASE_URL is set).
4) `pnpm turbo run generate-contracts --filter=apps/web` (after API running to serve OpenAPI).
5) `cargo run -p api` to start backend; `pnpm dev --filter=apps/web` to start frontend.

---

## Implementation Guide

### Setup Steps

- Create feature branch for tech-spec implementation.
- Add root Cargo workspace and pnpm/turbo workspace files.
- Configure .env with DATABASE_URL (SQLite default) and VITE_API_BASE.
- Initialize SQLx migrations and generate sqlx-data.json.

### Implementation Steps

1) Bootstrap Rust workspace and API crate `apps/api`; add dependencies and basic main.rs with health route and tracing.
2) Add config module and DB pool initialization; wire migrations runner on startup.
3) Implement migrations and sqlx query layer; define models and services for categories, budgets, transactions, variance.
4) Implement routes with Axum handlers; include OpenAPI annotations; expose Swagger UI and JSON spec.
5) Create `packages/contracts` with openapi-typescript; script to fetch spec from running API; publish types locally for web app.
6) Scaffold React app under `apps/web` with Vite; add API client wrapper using generated contracts; implement pages for categories, budgets, transactions, variance summary; add routing and layout.
7) Add validation and error toasts; handle loading/empty states; paginate transactions.
8) Tests: cargo tests for services/routes; vitest for hooks/components; MSW for API mocks; ensure linting/formatting pass.

### Testing Strategy

- Backend unit/integration: services with in-memory SQLite; route tests via axum Router and hyper client; fixtures for sample categories/budgets/transactions; assert variance math.
- Frontend: vitest + testing-library for pages/components; mock API via MSW using generated types; verify form validation, table rendering, pagination, and variance display.
- Contract sync: CI step to regenerate OpenAPI and TS client; fail if diff detected.

### Acceptance Criteria

1. Given the API is running, when a category is created, it is persisted and retrievable via GET /api/categories and includes UUID, name. 
2. Given a budget is created for a category/month, when a duplicate month for the same category is sent, the API returns 409 conflict.
3. Given transactions exist across categories and months, when GET /api/variance?month=YYYY-MM is called, response returns per-category budget, spent, remaining, overspend flag, and overall totals; numbers are integer cents and match stored data.
4. Frontend displays lists for categories, budgets, transactions, and a variance summary for a selected month; users can create each entity via forms with required-field validation and see success/error states.
5. Generated TS client matches current OpenAPI spec; `pnpm test` and `cargo test` both pass; linting passes with no warnings.

---

## Developer Resources

### File Paths Reference

- Cargo.toml (workspace root)
- apps/api/src/main.rs
- apps/api/src/{config.rs,db/mod.rs,models/*,services/*,routes/*,errors.rs,openapi.rs}
- apps/api/migrations/0001_init.sql
- packages/contracts/package.json, src/index.ts (generated)
- apps/web/src/{api/client.ts,routes.tsx,App.tsx,pages/*,components/*}
- turbo.json, pnpm-workspace.yaml, .env.example

### Key Code Locations

- API router setup: apps/api/src/main.rs
- OpenAPI registration: apps/api/src/openapi.rs
- Variance calculation: apps/api/src/services/variance.rs
- Frontend API wrapper: apps/web/src/api/client.ts
- Variance UI: apps/web/src/pages/Variance.tsx

### Testing Locations

- Backend tests: apps/api/tests or apps/api/src/**/mod.rs with #[cfg(test)]
- Frontend tests: apps/web/src/**/*.test.ts[x]
- Contract regeneration check: CI step running `pnpm turbo run generate-contracts --filter=packages/contracts`

### Documentation to Update

- README.md (project setup + commands)
- docs/tech-spec.md (this file) as changes occur
- API docs auto-served at /api-docs

---

## UX/UI Considerations

- Simple, form-first UI with inline validation messages; disable submit while pending; show toast for success/error.
- Tables with sticky header and pagination for transactions (page size 50); sort by date desc.
- Responsive layout with single column on mobile; keep form labels above inputs.
- Accessibility: labeled inputs, aria-live for toasts, focus states, keyboard navigable buttons/links.

---

## Testing Approach

- Enforce contract sync by regenerating TS client in CI and failing on diff.
- Backend: run cargo test with sqlite in-memory; seed fixtures for variance math; check error envelopes.
- Frontend: vitest + testing-library; MSW to stub endpoints; snapshot variance totals; validate form error states.
- Lint/format gates: cargo fmt/clippy; pnpm lint/format.

---

## Deployment Strategy

### Deployment Steps

1) Build API release binary via `cargo build --release -p api`.
2) Build frontend via `pnpm turbo run build --filter=apps/web`; serve static assets (e.g., from Nginx) or bundle with API via reverse proxy.
3) Package container with multi-stage Dockerfile: builder for Rust + Node; runtime with API binary + static assets under /public; set DATABASE_URL env.
4) Run migrations on startup (`sqlx migrate run`).
5) Expose PORT 3000; configure healthz for liveness/readiness.

### Rollback Plan

- Keep previous container image tagged; redeploy previous image; run migrations down only if strictly necessary (prefer backward-compatible migrations).

### Monitoring

- Track request rate/latency and 5xx via logs; watch DB errors; add simple log-based alerting; optional access logs sampling for variance endpoint.

