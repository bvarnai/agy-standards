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
   - Read `docs/roadmap.md` when present to identify the active slice and parent `epic/<E>` integration branch.
   - When `docs/spec/global.md` exists alongside `docs/spec/<unit>.md`, read both `global.md` (for shared architecture, error codes, execution boundaries, and package dependency rules) and the target `<unit>.md` before writing code.
2. **Topic branch check**:
   - Inspect `git branch --show-current`:
     - **On the default branch:** With `git-brx` on `PATH`, propose `git brx create issue/<id> -y` for a tracked issue (the user confirms or changes the branch type; follow the `git-brx` skill for everything after that). Without `git-brx`, ask how the project creates branches; do not invent a convention.
     - **On an `epic/<E>` integration branch:** Never commit implementation code directly onto `epic/<E>`. With `git-brx` on `PATH`, ensure `epic/<E>` is current (`git brx update`) and propose `git brx create issue/<id> -y` to fork a child branch off `epic/<E>` (user confirms). Without `git-brx`, ask how child branches fork off `epic/<E>`.
     - **On an existing child `issue/<id>` branch with `docs/roadmap.md`:** Resolve parent `epic/<E>` from `docs/roadmap.md` so sync and PR review target `epic/<E>`.

## Implementation workflows

### 1. Feature / refactor workflow
1. **Day-0 scaffolding (new repositories only)**: When starting a new project from `docs/spec/`, initialize standard governance and build files (`LICENSE`, `README.md`, `CHANGELOG.md`, `.gitignore`, `.gitattributes` with `* text=auto eol=lf`, and `.github/workflows/ci.yml` triggering `push` and `pull_request` on both the default branch (`main`) and `"epic/**"`).
2. **Boundary & domain implementation**: Implement narrow execution/runner interfaces (`Runner`, `Inspector`, `HAL`) and core domain logic in small, bisectable units matching the package dependency direction in `global.md`. Update `docs/spec/` and affected user-facing documentation (`README.md`, `docs/user-guide/`) in the same change when CLI flags, configuration keys, or user-visible behaviors change. When implementing a slice tracked in `docs/roadmap.md`, update the slice's `Status` from `Pending` to `Done (#<id>)` (using the slice issue number) in the same child PR (parallel-slice teams may instead update `docs/roadmap.md` directly on `epic/<E>` post-merge).
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
3. **Handoff**:
   - **Standard PRs (targeting default branch):** With `git-brx` on `PATH`, continue with the `git-brx` skill (`git brx sync`, `git brx publish`, `git brx review`), then post-merge run `git brx delete issue/<id>` and `git brx update`. Without `git-brx`, stop after the commit; the user publishes and opens the PR their own way.
   - **Child slice PRs (under `epic/<E>`):** With `git-brx` on `PATH`, pass `epic/<E>` explicitly (`git brx sync epic/<E>`, `git brx publish`, `git brx review epic/<E>`). Post-merge, switch back to and update `epic/<E>` before cleaning up or starting the next slice (`git brx select epic/<E>`, `git brx update`, `git brx delete issue/<id>`, or if `git brx delete issue/<id>` ran first and switched to the default branch, run `git brx select epic/<E>` and `git brx update`). After the final slice lands, follow Phase C of `/01-roadmap`.
