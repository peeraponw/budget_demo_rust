# Story 1.2: Contracts & web data layer

**Status:** Draft

---

## User Story

As a front-end developer,
I want generated TypeScript types/clients from the OpenAPI spec and a shared fetch wrapper,
So that the web app can call the API safely without drift.

---

## Acceptance Criteria

**Given** the API is running, **When** `pnpm turbo run generate-contracts` runs, **Then** `packages/contracts/src/index.ts` regenerates from /api-docs/openapi.json with no manual edits.
**Given** the web client calls categories, budgets, and transactions endpoints, **When** responses return, **Then** they are typed via the generated client and handled through React Query with loading and error states.
**Given** CI runs, **When** contracts are outdated versus the spec, **Then** the pipeline fails on diff.

---

## Implementation Details

### Tasks / Subtasks

- [ ] Add packages/contracts with package.json, tsconfig, and openapi-typescript 7.9.1 dependency. (AC: 1)
- [ ] Script to fetch /api-docs/openapi.json and generate TS client; document in README. (AC: 1)
- [ ] Wire Turborepo pipeline: `generate-contracts`, `lint`, `test`, `build` using pnpm. (AC: 3)
- [ ] Add fetch wrapper in apps/web/src/api/client.ts using generated types; inject base URL from env. (AC: 2)
- [ ] Integrate React Query provider and typed hooks for categories/budgets/transactions. (AC: 2)
- [ ] Add CI check to regenerate contracts and fail on diff. (AC: 3)

### Technical Summary

OpenAPI source from Axum/utoipa; openapi-typescript 7.9.1 generates ESM client; Turborepo caches tasks; React Query 5 handles caching, retries, and loading states; fetch wrapper centralizes headers/error handling.

### Project Structure Notes

- **Files to modify:** packages/contracts/{package.json,tsconfig.json,README.md,src/index.ts (generated)}; apps/web/src/api/client.ts; turbo.json; pnpm-workspace.yaml; CI workflow (if added).
- **Expected test locations:** apps/web/src/**/*.test.ts[x] for hooks/components; possible contract-generation smoke test under packages/contracts.
- **Estimated effort:** 3 story points (time estimate not provided by policy)
- **Prerequisites:** Story 1.1 (API + OpenAPI spec available)

### Key Code References

- OpenAPI served at /api-docs/openapi.json (from Story 1.1)
- Generated client in packages/contracts/src/index.ts
- React Query hooks in apps/web/src/api/client.ts

---

## Context References

**Tech-Spec:** [tech-spec.md](../tech-spec.md) – contract-first approach, tooling, and stack versions.

**Architecture:** See "Integration Points" and "Implementation Stack" in tech-spec.md.

---

## Dev Agent Record

### Agent Model Used

### Debug Log References

### Completion Notes

### Files Modified

### Test Results

---

## Review Notes

