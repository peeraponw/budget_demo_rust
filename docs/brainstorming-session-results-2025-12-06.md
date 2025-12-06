# Brainstorming Session Results

**Session Date:** 2025-12-06
**Facilitator:** Business Analyst Mary
**Participant:** Warm

## Session Start

Approach: AI-recommended techniques (First Principles Thinking, Question Storming, Resource Constraints, Mind Mapping) tailored for a lean MVP. Focus stays on simplest viable budgeting flows in a monorepo (React frontend, Rust backend).

## Executive Summary

**Topic:** MVP monorepo budgeting app with React frontend and Rust backend

**Session Goals:** Build only MVP complexity; allow setting monthly budgets per category; enter transactions and assign to categories; keep architecture simple; draw inspiration without over-engineering

**Techniques Used:** First Principles Thinking; Question Storming; Resource Constraints; Mind Mapping

**Total Ideas Generated:** 26

### Key Themes Identified:

1) Keep core domain tiny: Budgets, Categories, Transactions, and simple Variance.  
2) UX must make assignment frictionless: fast add, quick category pick, defaults.  
3) Monorepo simplifies DX: shared types, schema, and test fixtures across React + Rust.

## Technique Sessions

**First Principles Thinking (creative)**  
- Fundamental objects: category, monthly_budget, transaction, assignment, user (optional later).  
- Core flows: set budgets per category → add transaction → assign category → view remaining/variance.  
- Minimal data model: Category{id, name, limit, period}; BudgetPeriod{month, year}; Transaction{id, date, amount, payee, note, category_id?}.  
- Constraints: local-first optional, single currency initially, single user, no reports beyond balance/variance.  

**Question Storming (deep)**  
- How to prevent unassigned transactions from getting lost?  
- Should category limits roll over or reset monthly?  
- Do we need recurring transactions or templates in MVP?  
- How to keep performance simple—client-only filters or backend queries?  
- What’s the minimal auth—none vs. single-user token?  
- Is offline entry important for MVP?  
- How to present “remaining this month” without graphs?  
- Should imports (CSV/bank) wait until post-MVP?  

**Resource Constraints (structured)**  
- Must have: category CRUD, monthly budgets, transaction add/edit/delete, assign category, remaining/variance per category, simple list filters.  
- Nice-to-have later: CSV import, recurring transactions, multi-user, multi-currency, chart widgets.  
- Delayed: bank sync, forecasting, shared households, mobile app.  

**Mind Mapping (structured)**  
- User flows: Budget setup → Category limits → Add transaction → Assign → View summary.  
- Surfaces cross-cutting needs: validation, empty states, keyboard shortcuts, audit-friendly log.  
- Integration points: shared type definitions; Rust exposes REST/JSON; React uses query hooks with optimistic update.

## Idea Categorization

### Immediate Opportunities

_Ideas ready to implement now_

- Finalize minimal data model and shared types in monorepo.  
- Build category + budget CRUD UI with per-month limits and remaining amount display.  
- Create transaction entry form (amount, date, payee, note, optional category) with quick-assign chips.  
- Summary view: table of categories with budget, spent, remaining; highlight overspending.  
- Unassigned tray: list unassigned transactions to nudge categorization.

### Future Innovations

_Ideas requiring development/research_

- CSV import template for bank exports.  
- Recurring transactions/templates for rent, salary, utilities.  
- Multi-currency and FX normalization.  
- Shared household profiles and role-based access.  
- Basic spend visualizations (trend line, category pie) once data volume grows.

### Moonshots

_Ambitious, transformative concepts_

- Lightweight spend forecasting using simple smoothing.  
- Natural-language entry (“Groceries 54.20”).  
- Privacy-preserving bank sync via tokenized aggregator.  
- Budget coach tips generated from patterns.

### Insights and Learnings

_Key realizations from the session_

- Keep everything oriented around “remaining this month” to drive decisions.  
- Unassigned transactions are risk—surface them prominently.  
- Shared schema in monorepo reduces drift between frontend and backend.  
- MVP is viable without imports; manual entry plus good UX is enough to learn.

## Action Planning

### Top 3 Priority Ideas

#### #1 Priority: Ship the core data model and API

- Rationale: Everything depends on stable types and endpoints.  
- Next steps: Define Rust structs (Category, BudgetPeriod, Transaction); expose CRUD + summary endpoints; add OpenAPI; share generated TS types to React.  
- Resources needed: Rust web framework (Axum/Actix), SQLite or Postgres, codegen tool (e.g., oapi-codegen or openapi-typescript), Docker for local db.  
- Timeline: Align with first development sprint sequence (schema → handlers → tests).

#### #2 Priority: Budget & category UX

- Rationale: Users must see limits and remaining at a glance.  
- Next steps: Build budget setup page; inline add/edit category; per-category remaining; overspend badge; empty states.  
- Resources needed: React + UI kit, form/state libs, shared types, fixtures for Storybook/local mocks.  
- Timeline: Follows API readiness; can stub with mock data in parallel.

#### #3 Priority: Transaction capture & assignment

- Rationale: Without fast entry/assign, budgets are meaningless.  
- Next steps: Transaction form with keyboard shortcuts; unassigned tray; quick-assign chips; basic filters; optimistic updates.  
- Resources needed: React query layer, validation helpers, design tokens, backend endpoints for create/update/delete.  
- Timeline: Parallelizable once data model fixed; wire to live API after mock flow works.

## Reflection and Follow-up

### What Worked Well

Fast convergence by enforcing MVP boundaries; technique mix balanced creativity with structure; shared types idea reduces rework.

### Areas for Further Exploration

Decide on auth posture (none vs. minimal token); confirm db choice (SQLite for simplicity vs. Postgres for concurrency); clarify rollover rules.

### Recommended Follow-up Techniques

Follow-up: SCAMPER on transaction UX; Assumption Reversal for auth/privacy; Analogical Thinking using envelope budgeting metaphors.

### Questions That Emerged

Do budgets reset or roll over each month?  
Single-user only or invite flow later?  
Is offline/local-first valuable for target users?  
Do we need audit trail/versioning on edits?

### Next Session Planning

- **Suggested topics:** Deep-dive auth decision; CSV import viability; UX for rollover rules.  
- **Recommended timeframe:** After completing research workflow.  
- **Preparation needed:** Gather sample bank CSVs; choose db; align on auth stance.

---

_Session facilitated using the BMAD CIS brainstorming framework_
