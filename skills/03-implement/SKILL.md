---
name: 03-implement
description: Implement features, refactors, or bug fixes strictly traceable to an approved specification or issue, enforcing stack-specific engineering standards, living spec synchronization, and automated test verification. Activate when implementing code, fixing bugs, or running /03-implement.
metadata:
  lifecycle_stage: "03-implement"
  mode: "interactive"
  recommended_flags: "--effort medium"
---

# 03 — Spec-driven implementation

## Constraints
- **Tone:** Follow [strict-technical-tone.md](../../fragments/strict-technical-tone.md).
- **Permission:** Create and edit source, test, `docs/spec/`, `docs/roadmap.md` (when implementing an epic slice), and user documentation files in CWD that the issue or spec requires.

## Input contract
1. **Source of truth**:
   - Require an explicit issue (or `.issue-draft.md` in `CWD`) and/or approved specification in `docs/spec/`. If neither exists, halt and invoke `/02-issue` or `/00-spec` first.
   - Read `docs/roadmap.md` when present to identify the active slice and parent integration branch.
   - When `docs/spec/global.md` exists alongside `docs/spec/<unit>.md`, read both `global.md` (for shared architecture, error codes, execution boundaries, and package dependency rules) and the target `<unit>.md` before writing code.
2. **Topic branch check**:
   - Inspect `git branch --show-current`. If still on the default branch or on an epic integration branch (carrying `docs/roadmap.md`):
     - Never commit slice implementation code directly onto the default branch or the integration branch; fork a dedicated topic branch for the issue/slice (off the default branch for standalone issues, or off the up-to-date integration branch for roadmap slices).
     - With `git-brx` on `PATH`: propose creating the topic branch via `git-brx` (the user confirms or changes the branch type) and follow the `git-brx` skill for everything after that.
     - Without `git-brx`: ask how the project creates topic branches. Do not invent a convention.

## Implementation workflows

### 1. Feature / refactor workflow
1. **Day-0 scaffolding (new repositories only)**: When starting a new project from `docs/spec/`, initialize standard governance and build files (`LICENSE`, `README.md`, `CHANGELOG.md`, `.gitignore`, `.gitattributes` with `* text=auto eol=lf`, and `.github/workflows/ci.yml`).
2. **Boundary & domain implementation**: Implement narrow execution/runner interfaces (`Runner`, `Inspector`, `HAL`) and core domain logic in small, bisectable units matching the package dependency direction in `global.md`. Update `docs/spec/` and affected user-facing documentation (`README.md`, `docs/user-guide/`) in the same change when CLI flags, configuration keys, or user-visible behaviors change. When implementing a slice tracked in `docs/roadmap.md`, update the slice's `Status` from `Pending` to `Done (#<id>)` (using the slice issue number) in the same child PR (parallel-slice teams may instead update `docs/roadmap.md` directly on the integration branch post-merge).
3. **Automated tests**: Write unit and black-box verification tests covering every Acceptance Criterion, Edge Case, and Error Matrix row in the spec/issue.

### 2. Bug fix / regression workflow (`fix:`)
1. **Reproduce first (TDD)**: Write a deterministic failing unit or integration test that reproduces the reported bug or edge case before modifying production logic.
2. **Root-cause fix**: Apply the minimal, idiomatic fix at the root cause (avoiding symptom-level workarounds).
3. **Spec check**: If the bug exposed an undocumented invariant or exit code trigger, update `docs/spec/` to lock in the expected behavior.

### 3. Verification gate
- Run the stack's automated test suite and static analysis commands from [`fragments/stack-routing.md`](../../fragments/stack-routing.md) (`go test -race ./...`, `go vet ./...`, `ruff check .`, `pytest`, `./mvnw test`, `shellcheck`, `bats`, `ctest`).
- Do not claim completion until all automated tests and linters pass with zero warnings or errors.
- After three failed fix/test attempts on the same blocker, stop and report `Blocked` with the failing check and observed cause.

## Output contract & commit / PR handoff
Upon passing verification, output:
1. **Acceptance criteria verification matrix**:
   | Acceptance criterion / edge case | Status | Verifying test / file |
   | :--- | :--- | :--- |
   | ... | Pass | `path/to/test_file:TestName` |
2. **Proposed commit** (format from [`fragments/git.md`](../../fragments/git.md); keep `.issue-draft.md` untracked as the local work-unit spec—update it and notify the user if scope changes mid-flight so the remote issue can be synced, and remove it only after the PR merges; wait for user confirmation before executing mutating Git commands):
   ```bash
   git add <modified-files>
   git commit -m "<type>(<component>): <concise summary>"
   ```
3. **Handoff**: Target the default branch for standalone issues, or the parent integration branch recorded in `docs/roadmap.md` for epic slices (and follow Phase C of `/01-roadmap` once the final slice merges). With `git-brx` on `PATH`, continue with the `git-brx` skill (sync, publish, review, and post-merge branch cleanup against the target branch). Without it, stop after the commit; the user publishes and opens the PR their own way.
