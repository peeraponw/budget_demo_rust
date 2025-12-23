# Story 1.3: UI flows for budgets & transactions

**Status:** Draft

---

## User Story

As a user,
I want to create and review categories, budgets, and transactions and see a monthly variance summary,
So that I can understand spending against budget.

---

## Acceptance Criteria

**Given** the app is running, **When** I open Categories, Budgets, and Transactions pages, **Then** I can view tables, create new records with required validation, and see success/error toasts.
**Given** I select a month on the variance page, **When** data loads, **Then** I see per-category budget, spent, remaining, and overspend flag along with totals; loading and empty states are handled.
**Given** pagination is needed for transactions, **When** I load the page, **Then** it shows a paginated list (default 50) sorted by date descending and supports next/prev controls.

---

## Implementation Details

### Tasks / Subtasks

- [ ] Add routing shell and layout components; include nav to Categories, Budgets, Transactions, Variance. (AC: all)
- [ ] Categories page: list/table, create form with name validation, success/error toasts. (AC: 1)
- [ ] Budgets page: list/table, create form with category select + month + amount (cents), handle duplicate conflict message. (AC: 1)
- [ ] Transactions page: list with pagination (limit 50, default page 1), sort by date desc, create form with optional category assignment. (AC: 1,3)
- [ ] Variance page: month selector, fetch variance endpoint, render per-category rows and totals with overspend indicator; loading/empty/error states. (AC: 2)
- [ ] Wire pages to generated client + React Query; reuse FormField/Table components; basic responsive CSS.

### Technical Summary

React 18 + Vite 6 app using React Query 5; forms use controlled inputs with manual validation; tables use basic CSS modules; data via generated TS client; pagination query params passed to API; toasts via lightweight custom hook.

### Project Structure Notes

- **Files to modify:** apps/web/src/{App.tsx,routes.tsx,api/client.ts,pages/{Categories.tsx,Budgets.tsx,Transactions.tsx,Variance.tsx},components/{FormField.tsx,Table.tsx,Layout.tsx},styles/*.css}.
- **Expected test locations:** apps/web/src/**/*.test.tsx for pages/components; optional MSW handlers for API stubs.
- **Estimated effort:** 3 story points (time estimate not provided by policy)
- **Prerequisites:** Stories 1.1 and 1.2

### Key Code References

- Generated client from packages/contracts
- Variance endpoint at /api/variance (from Story 1.1)
- Fetch wrapper in apps/web/src/api/client.ts (from Story 1.2)

---

## Context References

**Tech-Spec:** [tech-spec.md](../tech-spec.md) – UX scope, acceptance criteria, and file paths.

**Architecture:** See "UX/UI Considerations" and "Developer Resources" in tech-spec.md.

---

## Dev Agent Record

### Agent Model Used

### Debug Log References

### Completion Notes

### Files Modified

### Test Results

---

## Review Notes

