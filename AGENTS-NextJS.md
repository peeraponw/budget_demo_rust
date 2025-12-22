# AGENTS.md — TypeScript + React 19 Development Standards

This document defines the authoritative rules for writing, structuring, testing, deploying, and maintaining React 19 + TypeScript code.

All rules are binding.

All MUST/NEVER statements are strictly enforced.

  

This file contains:

- Architecture rules

- DevOps rules

- TypeScript/React-specific rules

- AI-assistant guidance

  

No opinions. Only enforceable, factual standards.

  

If you feel unsure about any implementations, use **Context7 MCP** to check the documentation.

  

---

  
  

# 1. Core Development Philosophy

  
  

## KISS (Keep It Simple, Stupid) — MUST

Code MUST prefer the simplest correct solution, minimizing cognitive load.

  

## YAGNI (You Aren’t Gonna Need It) — MUST

No speculative features. Implement only when required.

  

## Fail Fast — MUST

- Errors MUST be raised immediately.

- No silent fallbacks or default calculations unless explicitly documented.

- Forbidden pattern:

  

```ts

// ❌ Forbidden: silent fallback

if (isValid(input)) {

return calculate(input);

} else {

return 0; // hides error

}

```

  

```ts

// ✅ Required: fail fast

if (!isValid(input)) throw new Error("Invalid input");

return calculate(input);

```

  

---

  
  

# 2. Architecture Standards

  
  
  

## 2.1 Vertical Slice Architecture — MANDATORY

  

Every feature is self-contained with:

  

* components

* hooks

* schemas

* types

* API clients

* tests

  

## 2.2 Project Structure (MUST)

  

```

src/

features/

[feature]/

__tests__/

components/

hooks/

api/

schemas/

types/

index.ts

shared/

components/

hooks/

utils/

constants/

test/

setup.ts

test-utils.tsx

```

  

## 2.3 File, Function, and Complexity Limits — MUST

  

* Max file size: **500 lines**

* Max component file size: **200 lines**

* Max function length: **50 lines**

* Max cyclomatic complexity: **10**

* Max cognitive complexity: **15**

* Lines ≤ **100 characters**

  

## 2.4 Never Use Relative Imports — MUST

  

```ts

// ❌ Forbidden

import { A } from "../../utils/A";

  

// ✅ Required

import { A } from "project/shared/utils/A";

```

  

## 2.5 Avoid Magic Strings — MUST

  

All constants MUST be defined in:

  

```

src/shared/constants/index.ts

```

  

## 2.6 Dependency Injection for Complex Components — MUST

  

Services, repositories, API clients, and env-config MUST be injected.

  

## 2.7 No Opinions in Documentation — MUST

  

Only rules. No subjective statements.

  

---

  
  
  

# 3. DevOps Standards

  
  
  

## 3.1 `.dockerignore` + `.containerignore` — REQUIRED

  

```

**/tests/

**/__tests__/

*.log

*.tmp

__pycache__/

node_modules/

coverage/

dist/

.vitest/

```

  

Test files MUST NOT be included in production images.

  

## 3.2 Explicit COPY Rules — MUST

  

NEVER `COPY . .`.

  

Use:

  

```dockerfile

COPY package.json package-lock.json tsconfig.* biome.json ./

COPY src ./src

```

  

## 3.3 Git Workflow — REQUIRED

  

Branches:

  

* `main`

* `dev`

* `feat/*`

* `fix/*`

* `docs/*`

* `refactor/*`

* `test/*`

  

Process:

  

1. `git checkout main && git pull`

2. `git checkout -b feat/...`

3. Code + tests

4. `git push`

5. PR → Review → Merge

  

## 3.4 Semantic Commits — REQUIRED

  

```

feat(scope): message

fix(scope): message

docs(scope): message

refactor(scope): message

test(scope): message

```

  

## 3.5 Use `rg` Instead of `grep` or `find` — REQUIRED

  

```bash

rg "pattern"

rg --files -g "*.ts"

```

  

---

  
  
  

# 4. React 19 Standards

  
  
  

## 4.1 Component Return Types — STRICT

  

* Components MUST return `ReactElement`.

* MUST import from `"react"`.

  

```ts

import { ReactElement } from "react";

  

export function Button(): ReactElement {

return <button>OK</button>;

}

```

  

* `JSX.Element` MUST NOT appear in code.

  

## 4.2 Component Declaration Style — CHOOSE ONE (strict rule)

  

We enforce the **named function style** for clarity and consistency.

  

```ts

// ✅ Required

export function Button(props: ButtonProps): ReactElement { ... }

  

// ❌ Forbidden

const Button: React.FC<ButtonProps> = () => { ... }

```

  

## 4.3 Component Requirements

  

* MUST handle: loading, error, empty, success

* MUST be ≤ 200 lines

* MUST include ARIA attributes

* MUST validate props using Zod when receiving external data

* MUST NOT return `null` without documented empty state

  

---

  
  
  

# 5. TypeScript Standards

  
  
  

## 5.1 Compiler Strictness — MUST

  

```json

{

"strict": true,

"noUnusedLocals": true,

"noUnusedParameters": true,

"noImplicitReturns": true,

"noUncheckedIndexedAccess": true,

"exactOptionalPropertyTypes": true,

"strictNullChecks": true

}

```

  

## 5.2 No `any` — MUST

  

Use `unknown` if needed with validation.

  

## 5.3 No Suppression — MUST

  

* `@ts-ignore` and `@ts-expect-error` forbidden

* Exception: ONLY when working around third-party types AND must include a comment + issue reference.

  

## 5.4 Domain Types MUST Use Zod Branded Types

  

```ts

const UserIdSchema = z.string().uuid().brand<"UserId">();

type UserId = z.infer<typeof UserIdSchema>;

```

  

## 5.5 All External Data MUST Be Validated

  

Includes:

  

* API responses

* Form inputs

* URL params

* Environment variables

  

## 5.6 Environment Config — MUST

  

* MUST NOT read `process.env` outside a single config module.

* MUST validate environment variables with Zod.

  

---

  
  
  

# 6. State Management Rules

  
  
  

Strict hierarchy:

  

1. Local state (`useState`)

2. Feature-level context

3. Server state (MUST use TanStack Query)

4. Global state (Zustand only when required)

5. URL state via search params

  

Server state MUST validate API data with Zod.

  

---

  
  
  

# 7. API Layer Rules

  
  
  

* All API functions MUST:

  

* validate request bodies with Zod

* validate server responses with Zod

* throw errors on non-OK responses

* never return invalid data

  

---

  
  
  

# 8. Data Validation (MANDATORY)

  
  
  

* MUST validate all external data

* MUST use Zod schemas

* MUST infer types from Zod (`z.infer`)

* MUST use branded types for ID fields

* MUST fail fast

  

---

  
  
  

# 9. Testing Standards (Shared With Python)

  
  
  

## 9.1 Coverage — MUST

  

Minimum **80%**.

  

## 9.2 Co-located Tests — MUST

  

Use `__tests__` inside each feature.

  

## 9.3 Vitest + React Testing Library — REQUIRED

  

User-interaction tests only (no implementation tests).

  

## 9.4 NEVER Skip Tests

  

Never commit `.skip` or `.todo`.

  

## 9.5 Test File Requirements

  

* MUST document all test files

* MUST use `unknown` instead of `any` in type merges

* MUST use `globalThis`, not `global`

  

---

  
  
  

# 10. Code Style (Biome-Only)

  
  
  

## 10.1 Biome is the Only Linter — MUST

  

ESLint MUST NOT be used.

  

## 10.2 Required Biome Config

  

```json

{

"files": { "include": ["src/**/*"] },

"linter": {

"rules": {

"complexity": { "noExcessiveCognitiveComplexity": "error" },

"correctness": { "noUnusedVariables": "error" }

}

},

"formatter": { "enabled": true }

}

```

  

## 10.3 No `console.log`

  

Only:

  

```ts

console.warn(), console.error()

```

  

Or structured logging.

  

---

  
  
  

# 11. Logging Standards

  
  
  

* MUST use structured logging (e.g., Pino)

* MUST include contextual metadata

* MUST log errors with stack traces

* MUST NOT log sensitive data

  

---

  
  
  

# 12. Security Requirements

  
  
  

* MUST validate and sanitize all user input

* MUST validate API responses

* MUST store secrets in environment variables

* MUST NOT commit secrets

* MUST use HTTPS in production

* MUST sanitize all HTML before using `dangerouslySetInnerHTML`

* CSP headers MUST be enabled in production builds

  

---

  
  
  

# 13. Performance Requirements

  
  
  

## 13.1 React 19 Optimization Rules

  

* Prefer clean code; compiler handles memoization

* MUST profile before adding manual `useMemo`, `useCallback`, or `React.memo`

* MUST lazy-load heavy components

* MUST use Suspense boundaries for async data

  

## 13.2 Bundle Optimization

  

Code splitting MUST be documented in `vite.config.ts`.

  

---

  
  
  

# 14. AI Assistant Rules

  
  
  

* MUST follow all rules strictly

* MUST check existing project patterns before generating new code

* MUST decompose tasks into small units

* MUST validate assumptions before generating code

* MUST generate tests before implementation (TDD)

  

NEVER:

  

* invent new dependencies

* modify core architecture

* introduce ESLint patterns

* introduce JSX.Element

* introduce relative imports

* create test files without documentation

  

---

  
  
  

# 15. Pre-Commit Checklist

  
  
  

* [ ] TypeScript passes with zero errors

* [ ] Zod validates all external data

* [ ] All features have tests

* [ ] ≥80% test coverage

* [ ] Biome passes with zero warnings

* [ ] File sizes within limits

* [ ] Complexity within limits

* [ ] No console.log

* [ ] All functions documented

* [ ] All components documented

* [ ] All TODOs include issue numbers

* [ ] No relative imports

* [ ] No magic strings

* [ ] No unused code

* [ ] No forbidden patterns

  

---