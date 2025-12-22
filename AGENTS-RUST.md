# AGENTS.md — Rust 1.88+ Development Standards

  

This document defines the authoritative rules for Rust 1.88+ development.

All MUST/NEVER statements are binding.

This file contains only factual rules, no opinions.

  

If you feel unsure about any implementations, use **Context7 MCP** to check the documentation.

  

---

  

# 1. Core Development Philosophy

  

## 1.1 KISS and YAGNI

  

- Code MUST use the simplest correct approach that satisfies the requirements.

- Abstractions MUST be introduced only when they remove real duplication or complexity.

- Unused extension points, speculative generic code, and premature abstractions MUST NOT be added.

  

## 1.2 Idiomatic Rust First

  

- Code MUST prefer idiomatic Rust constructs (iterators, enums, traits, pattern matching) over custom frameworks or ad-hoc patterns.

- Manual memory management, hand-rolled reference counting, or custom smart pointers MUST NOT be used unless standard library or established crates cannot satisfy the requirement.

  

## 1.3 Fail Fast

  

- Error paths MUST surface problems as early as possible.

- Silent fallbacks, hidden default values, and error swallowing MUST NOT be used, unless explicitly documented for a specific case.

  

```rust

// ❌ Forbidden: Silent default hides the real error

fn parse_count(input: &str) -> u32 {

input.parse().unwrap_or(0)

}

  

// ✅ Required: Fail fast with explicit error

fn parse_count(input: &str) -> Result<u32, ParseIntError> {

input.parse()

}

```

  

## 1.4 Unsafe Code Policy

  

* `unsafe` code MUST be exceptional.

* Every `unsafe` block MUST include a `// SAFETY:` comment explaining:

  

* Why `unsafe` is needed

* The invariants and preconditions required

* How those invariants are upheld

  

```rust

// SAFETY: `ptr` is obtained from Box::into_raw, non-null, and not aliased elsewhere.

unsafe { Box::from_raw(ptr) };

```

  

* New `unsafe` code MUST be accompanied by tests (unit/property/fuzz) that exercise the invariants.

  

---

  

# 2. Architecture & Project Structure

  

## 2.1 Workspace-First Layout

  

All projects MUST be workspace-based:

  

```text

project-root/

Cargo.toml # Workspace manifest

crates/

core/ # Business logic (library crate)

src/lib.rs

cli/ # Command-line interface (binary crate)

src/main.rs

api/ # HTTP or gRPC interface (library or binary)

src/lib.rs

domain/ # Domain models and logic (library)

src/lib.rs

infra/ # Infrastructure adapters (DB, queues, file I/O)

src/lib.rs

xtask/ # Dev-only utility crate (codegen, maintenance)

```

  

* Each crate MUST represent a cohesive vertical slice or layer.

* Cross-cutting utilities MUST live in a dedicated `utils`-style crate, not duplicated.

  

## 2.2 Module Structure and Limits

  

Per crate:

  

* Files MUST NOT exceed **500 lines**.

* Functions MUST NOT exceed **50 lines**.

* Public modules MUST NOT contain mixed concerns; split by responsibility.

  

Example structure:

  

```text

crates/core/src/

lib.rs

services/

mod.rs

user_service.rs

models/

mod.rs

user.rs

ports/

mod.rs

user_repository.rs

```

  

## 2.3 Import Rules (No Relative Up-Chains)

  

* Module paths MUST be crate-root based (`crate::` or crate name), not deep `super::super` ladders.

  

```rust

// ❌ Forbidden

use super::super::models::user::User;

  

// ✅ Required

use crate::models::user::User;

```

  

* Re-exporting from `mod.rs` for simpler paths is allowed and encouraged when it reduces path complexity.

  

## 2.4 Constants and “Magic” Values

  

* String literals, numeric thresholds, and configuration-like values MUST NOT be duplicated inline.

* Shared constants MUST be defined in a dedicated module per crate, e.g.:

  

```text

crates/core/src/constants.rs

crates/api/src/constants.rs

```

  

```rust

// crates/core/src/constants.rs

pub const MAX_RETRY_ATTEMPTS: u8 = 3;

pub const DEFAULT_PAGE_SIZE: usize = 50;

```

  

---

  

# 3. Cargo and Tooling Configuration

  

## 3.1 Workspace Cargo.toml (Required Baseline)

  

```toml

[workspace]

members = ["crates/*", "xtask"]

  

[workspace.package]

edition = "2024"

rust-version = "1.88"

  

[workspace.lints.rust]

unsafe_code = "forbid"

unused = "deny"

  

[workspace.lints.clippy]

pedantic = "warn"

nursery = "warn"

unwrap_used = "deny"

expect_used = "deny"

```

  

* `cargo` configuration MUST enforce `unsafe_code = "forbid"` at workspace level.

* Explicit `allow(unsafe_code)` MUST be applied only to modules that require it, with justification.

  

## 3.2 Mandatory Tooling

  

The following tools and commands MUST be used:

  

* Formatting: `cargo fmt --all`

* Linting: `cargo clippy --all-targets --all-features -- -D warnings`

* Testing: `cargo test` (or `cargo nextest run` when configured)

* Coverage: `cargo tarpaulin --fail-under 80`

* Security: `cargo audit` and, if configured, `cargo deny`

* Documentation: `cargo doc --no-deps`

  

---

  

# 4. Code Style & Structure

  

## 4.1 Formatting

  

* Code MUST be formatted with `rustfmt`.

* `rustfmt.toml` MUST be checked into the repository and used consistently.

  

Example:

  

```toml

edition = "2024"

style_edition = "2024"

max_width = 100

```

  

## 4.2 Clippy Standards

  

* `cargo clippy -- -D warnings` MUST pass.

* The following patterns MUST NOT appear in non-test code:

  

* `unwrap()`

* `expect()`

* `todo!()`

* `panic!()` except in unreachable/unrecoverable conditions with clear justification

  

## 4.3 Documentation

  

* All public items (functions, types, traits, modules) MUST have rustdoc comments.

* Rustdoc examples MUST compile as doctests when possible.

  

```rust

/// Parses a user ID from string input.

///

/// # Errors

///

/// Returns an error if the input is not a valid UUID.

pub fn parse_user_id(input: &str) -> Result<Uuid, uuid::Error> {

input.parse()

}

```

  

---

  

# 5. Error Handling

  

## 5.1 Library Crates

  

* Library crates MUST define typed error enums using `thiserror` or equivalent.

  

```rust

use thiserror::Error;

  

#[derive(Debug, Error)]

pub enum UserError {

#[error("user not found: {id}")]

NotFound { id: Uuid },

  

#[error("database error: {0}")]

Database(#[from] sqlx::Error),

}

```

  

## 5.2 Binary Crates

  

* Binary crates MAY use `anyhow::Result<T>` for ergonomic top-level error propagation.

* All public APIs (even in binaries) MUST attach context to errors.

  

```rust

use anyhow::{Context, Result};

  

fn load_config(path: &Path) -> Result<String> {

std::fs::read_to_string(path)

.with_context(|| format!("failed to read config file at {:?}", path))

}

```

  

## 5.3 Result Handling

  

* `Result` MUST NOT be ignored.

  

```rust

// ❌ Forbidden

do_something(); // returns Result, ignored

  

// ✅ Required

do_something()?; // propagate

```

  

---

  

# 6. Concurrency and Async

  

## 6.1 Async Runtime Usage

  

* Async code MUST use a single chosen runtime (e.g. `tokio`) per application.

* Blocking operations MUST NOT be executed directly in async contexts (no `std::thread::sleep` or blocking I/O inside async functions); use runtime-specific blocking utilities instead.

  

```rust

// ❌ Forbidden in async fn

std::thread::sleep(Duration::from_secs(1));

  

// ✅ Required

tokio::time::sleep(Duration::from_secs(1)).await;

```

  

## 6.2 Concurrency Hierarchy

  

Order of preference:

  

1. Single-threaded synchronous code

2. Async I/O with runtime (`tokio`)

3. Data-parallelism with `rayon` for CPU-bound workloads

4. Explicit synchronization primitives (`Mutex`, `RwLock`, `parking_lot`) only when necessary

  

* Shared mutable state MUST use appropriate synchronization primitives.

* `Send` and `Sync` auto-implementations MUST NOT be overridden via `unsafe impl` without a precise, documented justification.

  

---

  

# 7. Data, Serialization, and Validation

  

## 7.1 Serialization

  

* Data structures exposed over boundaries (HTTP, IPC, file formats) MUST derive `serde::Serialize` and/or `serde::Deserialize`.

* Versioned formats MUST use explicit version fields or separate types per version.

  

## 7.2 Input Validation

  

* All external input (CLI arguments, environment variables, HTTP bodies, DB rows) MUST be validated.

* For CLI, structured parsers like `clap` or `argh` MUST be used instead of manual argument parsing.

* For JSON and other data protocols, validation MUST occur immediately after deserialization.

  

---

  

# 8. Testing Strategy

  

## 8.1 Coverage Requirements

  

* Overall test coverage MUST be at least **80%**.

* New crates or modules MUST NOT reduce coverage below 80%.

  

## 8.2 Test Types

  

* Unit tests:

  

* Must live next to the code in `#[cfg(test)] mod tests { … }`.

* Integration tests:

  

* Must live in `tests/` at crate root.

* Property-based tests:

  

* MUST be used for critical algorithms and core business logic when applicable.

* Doctests:

  

* MUST be maintained for rustdoc examples.

  

## 8.3 Test Structure Example

  

```rust

#[cfg(test)]

mod tests {

use super::*;

  

#[test]

fn parses_valid_user_id() {

let id = parse_user_id("550e8400-e29b-41d4-a716-446655440000").unwrap();

assert_eq!(id.to_string(), "550e8400-e29b-41d4-a716-446655440000");

}

}

```

  

## 8.4 Test Execution

  

* `cargo test` (or `cargo nextest run` when configured) MUST pass without failures.

* Flaky tests MUST be fixed or removed; disabling tests without a tracking issue is not allowed.

  

---

  

# 9. Security Requirements

  

## 9.1 Dependencies

  

* `cargo audit` MUST pass with no unaddressed advisories.

* `cargo deny` (if used) MUST pass license and vulnerability checks.

  

## 9.2 Secrets and Configuration

  

* Secrets MUST NOT be committed to version control.

* Secrets MUST be supplied via environment variables, secrets managers, or OS keychains.

* Config crates MUST centralize environment variable access.

  

## 9.3 Unsafe and FFI

  

* FFI boundaries MUST NEVER panic.

* FFI functions MUST validate input before use.

* `unsafe` and FFI usage MUST always be covered by tests that exercise error paths.

  

---

  

# 10. Logging and Observability

  

## 10.1 Logging

  

* Logging MUST use a structured logging crate (e.g. `tracing`).

* Unstructured `println!` logging in production code MUST NOT be used.

  

```rust

tracing::info!(user_id = %user.id, "user logged in");

```

  

## 10.2 Error Logging

  

* All fatal errors at binary boundaries MUST be logged with error level.

* Logs MUST include enough context to debug issues (ids, relevant parameters) but MUST NOT include secrets.

  

---

  

# 11. DevOps: Containers and CI

  

## 11.1 Container Build Rules

  

* Docker or OCI builds MUST use both `.dockerignore` and `.containerignore`.

  

Example ignore patterns:

  

```text

**/target/

**/tests/

**/__pycache__/

*.log

*.tmp

.git

```

  

* Production images MUST NOT include:

  

* tests

* local tooling scripts

* unused dev artifacts

  

## 11.2 CI Pipeline Requirements

  

A CI job for Rust MUST at least:

  

1. Check out code

2. Install Rust stable

3. Run `cargo fmt --check`

4. Run `cargo clippy -- -D warnings`

5. Run tests (`cargo test` or `cargo nextest run`)

6. Run coverage (`cargo tarpaulin --fail-under 80`) where supported

7. Run `cargo audit`

  

Any failure in these steps MUST block merges to `main`.

  

---

  

# 12. Git Workflow and Search Commands

  

## 12.1 Git Branching

  

* Branches MUST follow:

  

* `main` — production-ready code

* `dev` — integration branch

* `feat/*` — new features

* `fix/*` — bug fixes

* `docs/*` — documentation

* `refactor/*` — refactoring

* `test/*` — test-specific changes

  

## 12.2 Commit Messages

  

* Commits MUST follow a conventional format:

  

```text

feat(core): add user deactivation

fix(api): handle invalid json payloads

docs(cli): update usage examples

```

  

## 12.3 Search Commands

  

* `rg` (ripgrep) MUST be used for code search.

* `grep` and `find` MUST NOT be used in scripts or documented workflows.

  

```bash

# Required

rg "pattern"

rg --files -g "*.rs"

```

  

---

  

# 13. AI Assistant Rules

  

The AI agent MUST:

  

* Obey all rules in this document.

* Inspect existing crates and modules before creating new ones.

* Prefer extending or reusing existing types/traits instead of introducing parallels.

* Create tests together with new functionality.

* Avoid adding dependencies without checking existing crates and patterns.

  

The AI agent MUST NOT:

  

* Introduce `unsafe` without explicit justification.

* Introduce new crates that duplicate existing functionality.

* Change workspace structure arbitrarily.

* Add container or CI configuration that conflicts with these rules.

  

---

  

# 14. Pre-Commit Checklist

  

Before committing, the following MUST all be true:

  

* [ ] `cargo fmt --check` passes

* [ ] `cargo clippy --all-targets --all-features -- -D warnings` passes

* [ ] All tests pass

* [ ] Coverage ≥ 80%

* [ ] `cargo audit` passes

* [ ] No `unwrap`, `expect`, `todo!`, or `panic!` in production paths (except justified cases)

* [ ] No new `unsafe` without `// SAFETY:` comment and tests

* [ ] All new public items have rustdoc

* [ ] No secrets added to the repository

* [ ] No relative import up-chains via `super::super` in module paths

* [ ] No `grep`/`find` references in scripts; only `rg`

  

---

  

# 15. Critical Non-Negotiables

  

1. Workspace MUST compile on Rust stable 1.88+.

2. `unsafe_code` MUST be forbidden at workspace root; explicit allowances MUST be justified.

3. All external inputs MUST be validated.

4. Overall coverage MUST be at least 80%.

5. All CI checks (format, lint, test, security) MUST pass before merging to `main`.

6. Architecture rules (workspace layout, module structure, constants, imports) MUST be followed strictly.

7. This document MUST be kept up to date with actual project practices; deviations MUST be resolved by updating code or this document, not by ignoring the rules.