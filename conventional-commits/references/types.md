# Conventional Commits: Type & Scope Reference Guide

This guide provides deep-dive rules and edge-case resolution for selecting types and scopes under Conventional Commits v1.0.0.

---

## 1. Type Selection Guide

Choosing the right type ensures automated versioning (Semantic Versioning), automated changelog generation, and immediate team clarity.

### `feat`
- **When**: Adding functionality that affects user or consumer behavior.
- **Rule of Thumb**: If this change warrants being mentioned in a product release note or API change list as something new, use `feat`.
- **Examples**:
  - `feat(cart): add one-click checkout button`
  - `feat(api): support pagination in list_users endpoint`

### `fix`
- **When**: Remedying a bug, defect, regression, or unexpected behavior in code.
- **Rule of Thumb**: Does this restore intended behavior without introducing new features? If so, use `fix`.
- **Examples**:
  - `fix(auth): handle expired refresh token gracefully`
  - `fix(billing): correct vat calculation for eu orders`

### `refactor`
- **When**: Restructuring, cleaning, or reorganizing existing code without adding new features or fixing a bug.
- **Rule of Thumb**: Code changes under the hood with zero external change in observable behavior.
- **Examples**:
  - `refactor(db): extract connection pool logic into singleton`
  - `refactor(parser): simplify ast traversal loop`

### `perf`
- **When**: Code changes made specifically to improve performance, latency, throughput, or resource consumption.
- **Examples**:
  - `perf(query): add compound index on user_id and created_at`
  - `perf(renderer): memoize expensive table row components`

### `docs`
- **When**: Changes only to documentation, Markdown files, inline comments, or typing documentation.
- **Examples**:
  - `docs(readme): add docker setup instructions`
  - `docs(api): document rate limiting headers`

### `test`
- **When**: Adding missing tests, updating test suites, adding mocks, or fixing failing tests without altering application logic.
- **Examples**:
  - `test(e2e): add test suite for checkout flow`
  - `test(auth): add unit tests for token expiration`

### `style`
- **When**: Formatting, white-space, missing semi-colons, variable renames for style, with zero runtime logic changes.
- **Note**: If your project uses automated formatting/linters, changes from these tools usually belong under `style` or `chore`.
- **Examples**:
  - `style: format with prettier and organize imports`
  - `style(components): adhere to airbnb style guide`

### `build`
- **When**: Changes affecting build tools, package managers, compiler configurations, or external dependencies.
- **Examples**:
  - `build(deps): bump pydantic from 2.5.0 to 2.6.0`
  - `build(webpack): split vendor bundle into separate chunk`

### `ci`
- **When**: Changes to CI/CD configuration files, workflows, runners, or scripts.
- **Examples**:
  - `ci(github): add matrix testing for python 3.10 and 3.11`
  - `ci(deploy): add automated staging deploy step`

### `chore`
- **When**: Miscellaneous maintenance tasks, updating non-source configuration files (.gitignore, .editorconfig, release scripts) that do not modify src or test.
- **Examples**:
  - `chore: update .gitignore to ignore .env.local`
  - `chore(release): cut version 1.4.0`

### `revert`
- **When**: Reverting a previous commit.
- **Header**: `revert: <header of reverted commit>`
- **Body**: Include SHA of reverted commit (`This reverts commit <hash>.`).

---

## 2. Determining Scope

A scope provides immediate geographic context inside the repository.

- **Monorepos / Multi-package**: Use the package or workspace name (e.g., `web`, `api`, `core`, `mobile`).
- **Single application**: Use the domain or layer (e.g., `auth`, `ui`, `billing`, `router`, `nav`, `db`).
- **Dependencies**: Use `deps` (e.g., `build(deps): ...` or `chore(deps): ...`).
- **When to omit**: If the change touches multiple unrelated subsystems or is repository-wide (e.g., global formatting or root config), omit the scope: `style: apply ruff format` or `chore: update license year`.

---

## 3. Resolving Ambiguities & Overlaps

| Scenario | Recommended Type | Rationale |
| :--- | :--- | :--- |
| Refactored code AND fixed a small bug along the way | `fix` (or split into 2 commits) | A fix takes precedence for versioning/changelogs. If possible, split into separate commits. |
| Upgraded a dependency to fix a security vulnerability | `fix(deps)` or `build(deps)` | If the primary driver is fixing a known bug/vuln, `fix(deps)` communicates impact best. |
| Upgraded dependencies routine bump | `build(deps)` or `chore(deps)` | Routine dependency updates. |
| Added tests alongside a new feature | `feat` | New feature commit should include its corresponding tests. Don't split tests into a separate `test` commit unless adding tests retroactively. |
| Refactored code to make it faster | `perf` | When optimization is the explicit goal, prefer `perf` over `refactor`. |

---

## 4. Breaking Changes Checklist

A commit is a **breaking change** if:
1. It alters an existing public function, class, or API signature.
2. It removes or renames an endpoint, parameter, or response field.
3. It changes configuration schema in a non-backward-compatible manner.
4. It raises the minimum supported language/runtime version.

**Format**:
- In Header: `type(scope)!: short description`
- In Footer:
  ```text
  BREAKING CHANGE: <Clear explanation of what broke and exact migration steps>
  ```
