---
name: conventional-commits
description: >-
  Use this skill to generate conventional git commit messages and automatically commit changes (autocommit without an approval gate), or amend the previous commit when the user wants changes. Trigger whenever the user asks to "commit this", "commit", creates or asks for a commit message, inspects staged or unstaged changes to commit, or asks to modify/amend the commit message. Applies to all change types (features, bug fixes, refactoring, tests, docs, build/ci, chores, breaking changes), even if Conventional Commits is not explicitly mentioned. Do not use for general git operations (rebase, squash, cherry-pick, push troubleshooting), PR descriptions, code reviews, or tooling setup.
---

# Conventional Commits & Autocommit

Generates standardized, concise git commit messages following the [Conventional Commits v1.0.0](https://www.conventionalcommits.org/en/v1.0.0/) specification and commits changes directly without approval gates, amending previous commits when revisions are requested.

## Core Commit Structure

```text
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### Key Formatting Rules

1. **Type**: Required noun identifying change category (`feat`, `fix`, `refactor`, `perf`, `docs`, `style`, `test`, `build`, `ci`, `chore`, `revert`).
2. **Scope**: Optional noun in parentheses indicating the affected module, component, or package (e.g., `auth`, `ui`, `api`, `parser`). Omit if change is cross-cutting or repo-wide.
3. **Breaking Change Marker (`!`)**: If the change introduces breaking changes, place an exclamation mark immediately before the colon: `<type>(<scope>)!: <description>`.
4. **Description**:
   - Starts immediately after the colon and space.
   - Use the **imperative, present tense**: "add", "fix", "update", "remove" (NOT "added", "fixes", "updating").
   - Lowercase the initial letter (unless starting with a proper noun or code identifier).
   - No trailing period (`.`).
   - Keep total subject line length under 72 characters (ideally 50-60 chars).
5. **Body**:
   - Optional. Separate from the header by one blank line.
   - Default to a single-line commit for simple, self-explanatory changes.
   - Include a body when the *why* or additional context is necessary for future maintainers.
   - Keep bullet points concise and informative.
6. **Footer**:
   - Optional. Separate from body (or header) by one blank line.
   - Token format: `Token: <value>` or `Token #<issue>` (token uses hyphens instead of spaces, e.g., `Reviewed-by: ...`, `Refs: #123`, `Closes: #456`).
   - Exception: `BREAKING CHANGE: <explanation and migration steps>` is uppercase and may contain a space.

---

## Workflow

### 1. Gather Changes
- **If in a git repository**:
  - Run `git status -s` to inspect modified, unstaged, and untracked files.
  - Run `git diff --cached` to inspect staged changes.
  - If nothing is staged, run `git diff` to inspect unstaged changes.
- **If user wants to revise or amend the previous commit**:
  - Run `git log -1 --stat` to inspect the latest commit message and affected files.
  - Check `git status -s` for any new unstaged/staged additions that should be included.
- **If user provided text or diff**:
  - Analyze the provided diff, patch, commit notes, or change description.

### 2. Determine Type & Scope

| Type | When to Use | SemVer Impact |
| :--- | :--- | :--- |
| `feat` | Adds a new feature or user-facing capability | MINOR |
| `fix` | Fixes a bug or patch defect | PATCH |
| `refactor` | Code restructuring that neither fixes a bug nor adds a feature | None |
| `perf` | Code change that improves performance | PATCH or None |
| `docs` | Documentation changes only (README, comments, docs) | None |
| `style` | Formatting, whitespace, lint fixes with zero logic change | None |
| `test` | Adding, refactoring, or fixing tests | None |
| `build` | Build system, toolchain, or external dependencies (npm, Cargo, etc.) | None |
| `ci` | CI/CD pipeline scripts and configuration files | None |
| `chore` | Maintenance tasks, configs, tooling not affecting src/test | None |
| `revert` | Reverts a previous commit | Depends |

*For deep dive on types, scopes, and edge cases, see [references/types.md](references/types.md).*

### 3. Check for Breaking Changes
- Does this remove an existing API, parameter, or behavior that external consumers rely on?
- If yes:
  - Add `!` before `:` in the header (e.g. `feat(api)!: remove deprecated v1 endpoints`).
  - Add a footer `BREAKING CHANGE: <what broke and how to migrate>`.

### 4. Autocommit (No Approval Gate)
When working in a git repository and asked to commit, **commit immediately without a confirmation gate**. Do not pause to ask the user "Should I commit?" or stop at a proposal gate.

1. **Stage files**:
   - If changes are not yet staged, stage the relevant modified and untracked files using `git add <files>` (or `git add -A` for repository-wide work).
2. **Execute commit**:
   - Run `git commit -m "<type>(<scope>): <description>"` directly.
   - If body or footers are included, use multiple `-m` flags:
     ```bash
     git commit -m "<type>(<scope>): <description>" \
       -m "<optional body explaining rationale>" \
       -m "<optional footer or BREAKING CHANGE>"
     ```
3. **Report result**:
   - Output the commit SHA, the commit message, and summary of committed files.

### 5. Amending Commits (When User Requests Changes)
If the user wants to change the commit message, adjust files in the commit, or revise anything after committing:
- **Do NOT create an extra redundant commit or run destructive resets.**
- **Use `git commit --amend`**:
  - **Message revision**:
    ```bash
    git commit --amend -m "<new type>(<new scope>): <new description>"
    ```
    (Add additional `-m` flags if body or footers are needed).
  - **Adding / modifying files**: Stage the updated files and amend:
    ```bash
    git add <files>
    git commit --amend --no-edit
    ```
    (Or supply `-m` if also updating the commit message).
- Report the updated commit SHA and message.

### 6. Text / Non-Git Mode
If the user only asks to generate, craft, or suggest a commit message (or provides a raw text diff without a git repository):
- Present the formatted Conventional Commit message in a `gitcommit` block along with the corresponding command.

---

## Examples

### Example 1: Autocommit Feature (No Gate)
**Context**: Added user authentication with JWT in `src/auth/`
**Action**:
1. Run `git add src/auth/`
2. Run `git commit -m "feat(auth): implement jwt-based user authentication"`
3. Report:
```text
Committed [a1b2c3d]: feat(auth): implement jwt-based user authentication
 2 files changed, 45 insertions(+)
```

### Example 2: Amending Commit Message
**Context**: User says: "Actually change the commit to mention ticket #45"
**Action**:
1. Run `git commit --amend -m "feat(auth): implement jwt-based user authentication" -m "Closes: #45"`
2. Report:
```text
Amended commit [e4f5g6h]: feat(auth): implement jwt-based user authentication
Closes: #45
```

### Example 3: Amending to Include Modified Files
**Context**: User says: "I forgot to add the tests in test/auth.test.ts to that commit"
**Action**:
1. Run `git add test/auth.test.ts`
2. Run `git commit --amend --no-edit`
3. Report:
```text
Amended commit [8j9k0l1]: feat(auth): implement jwt-based user authentication
 3 files changed, 68 insertions(+)
```

### Example 4: Autocommit Bug Fix with Body
**Context**: Fixed crash on empty checkout form
**Action**:
1. Run `git add src/checkout/`
2. Run `git commit -m "fix(checkout): prevent crash on empty form submission" -m "Add guard clause for empty cart and null address payloads."`
3. Report commit hash and summary.

### Example 5: Breaking Change
**Context**: Changed auth header from `X-Token` to `Authorization: Bearer`
**Action**:
1. Stage changes.
2. Run:
```bash
git commit -m "feat(auth)!: switch to bearer token authentication header" \
  -m "BREAKING CHANGE: clients must now send credentials via standard Authorization: Bearer header instead of X-Token."
```
3. Report commit hash and breaking change notice.
