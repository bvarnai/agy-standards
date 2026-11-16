# Example workflow session

This walkthrough shows a complete feature-to-release cycle in `agy` with `agy-standards` enabled. It uses a fictional Go CLI (`dotctl`) and assumes [`git-brx`](https://github.com/bvarnai/git-brx) is on `PATH`.

> **Note:**
> - **Without `git-brx`:** The agent asks how your repository creates branches and stops after `git commit` instead of proposing `git brx create` and `git brx publish`.
> - **Trivial changes:** Small fixes, doc edits, and dependency bumps skip `/00-spec` and `/01-issue` and go straight to implementation.

---

## Stage 0 — Spec authoring (`/00-spec`, `--mode plan --effort high`)

```text
> agy --mode plan --effort high
[agy] Loaded plugin: agy-standards (rules: GEMINI, spec-driven, git, stack-routing)
[agy] Stack detected: go.mod → standards/go-cli.md
```

**User:**
> `/00-spec` We want to add a `dotctl clean` subcommand that removes broken symlinks from `$HOME`.

**Agy (`plan` • `high`):**
> *Reading `README.md`, `go.mod`, and `docs/spec/global.md`...*
>
> Before updating `docs/spec/`, three edge cases need a decision:
> 1. **Scope:** Should `dotctl clean` scan `$HOME` non-recursively, recurse into all directories, or only scan paths managed in `dotctl.yaml`?
> 2. **Confirmation and safety:** Should it delete immediately, or require `--yes` or default to `--dry-run`?
> 3. **Exit code:** If broken symlinks are found in `--dry-run` mode, should it exit `0` or non-zero?

**User:**
> 1. Only scan target paths declared in `dotctl.yaml` (never walk arbitrary `$HOME` dirs).
> 2. Delete only with `--force` (or interactive prompt); `--dry-run` prints candidates and exits `0`.
> 3. Exit `2` on invalid config, `1` on filesystem I/O error.

**Agy (`plan` • `high`):**
> *Skill permission: `docs/spec/` write allowed.*
> Updated `docs/spec/global.md` (subcommand registry and exit code `2` invariant) and created `docs/spec/clean.md` (flags, `Runner` / `Inspector` boundary interface, error matrix).
>
> *Skipped optional sections in `clean.md`: Legacy discrepancies (greenfield subcommand).*

---

## Stage 1 — Issue drafting (`/01-issue`, `medium`)

```text
> /compact   # or fresh session: agy --effort medium
```

**User:**
> `/01-issue` add the clean subcommand from the spec

*(If you run `/01-issue` with no description in a fresh session, the agent asks 1–3 clarifying questions first; otherwise it derives the commit-format title from context.)*

**Agy (`build` • `medium`):**
> *Reading `docs/spec/global.md` and `docs/spec/clean.md`...*
> *Skill permission: `.issue-draft.md` write allowed.*
>
> Drafted issue to `.issue-draft.md`:
>
> - **Title:** `feat(clean): add dotctl clean subcommand`
> - **Design:** Implement `cmd/clean.go` and `internal/clean/clean.go` using injected `Inspector` / `Remover` interfaces.
> - **Acceptance criteria:**
>   - Scans only target paths from `dotctl.yaml`.
>   - `--dry-run` lists broken symlinks without deleting and exits `0`.
>   - `--force` removes broken symlinks without prompt; without `--force`, prompts `[y/N]`.
> - **Edge cases and failure modes:** Valid symlinks untouched; missing target directory skipped with warning; permission error returns exit `1`; invalid `dotctl.yaml` returns exit `2`.
> - **Verification:** `go test -race ./...` and `go vet ./...`.
>
> *(Create the ticket in your tracker from `.issue-draft.md`, then run `/02-implement`.)*

---

## Stage 2 — Implementation (`/02-implement`, `medium`)

```text
> agy --effort medium
```

**User:**
> `/02-implement` Implement issue #42 from `.issue-draft.md`.

**Agy (`build` • `medium`):**
> *Checking `docs/spec/global.md`, `docs/spec/clean.md`, `.issue-draft.md`, and `git branch --show-current`...*
>
> Currently on `main` and `git-brx` is on `PATH`.
> **Approval required (mutating Git):**
>
> ```bash
> git brx create issue/42 -y
> ```
>
> Confirm to create the topic branch?

**User:**
> Yes.

**Agy (`build` • `medium`):**
> *Ran `git brx create issue/42 -y` → switched to `issue/42`.*
> *Skill permission: source, test, `docs/spec/`, and user docs allowed.*
>
> 1. Implemented `internal/clean/clean.go` (`Inspector` / `Remover` interfaces) and `cmd/clean.go`.
> 2. Added unit and table-driven tests in `internal/clean/clean_test.go`.
> 3. Updated `README.md` subcommand table (`docs/spec/clean.md` already matches).
> 4. Ran `go test -race ./...` and `go vet ./...` — all passed (0 warnings).
>
> **Acceptance criteria verification matrix:**
>
> | Acceptance criterion / edge case | Status | Verifying test / file |
> | :--- | :--- | :--- |
> | Scans only `dotctl.yaml` paths | Pass | `internal/clean/clean_test.go:TestClean_ConfiguredPathsOnly` |
> | `--dry-run` exits `0` without deleting | Pass | `internal/clean/clean_test.go:TestClean_DryRun` |
> | `--force` skips prompt; default prompts `[y/N]` | Pass | `internal/clean/clean_test.go:TestClean_Confirmation` |
> | Permission error exits `1`; bad config exits `2` | Pass | `internal/clean/clean_test.go:TestClean_ErrorCodes` |
>
> **Proposed commit (keeping `.issue-draft.md` untracked):**
>
> ```bash
> git add cmd/clean.go internal/clean/ README.md
> git commit -m "feat(clean): add dotctl clean subcommand"
> ```
>
> Confirm commit and `git brx publish`?

**User:**
> Commit and publish.

**Agy (`build` • `medium`):**
> *Committed on `issue/42` and published PR via `git brx publish`.*
> *(Once merged, remove `.issue-draft.md` and sync `main`.)*

---

## Stage 3–5 — Release pipeline (`/03-docs` → `/04-audit` → `/05-release`)

### 3. Documentation review (`/03-docs`, `medium`)

**User:**
> `/03-docs README.md`

**Agy (`build` • `medium`):**
> *Skill permission: `none` (propose edits only).*
> - `README.md:44`: Replace *"seamlessly cleans up"* with *"removes broken symlinks declared in `dotctl.yaml`"* (cut fluff, match `docs/spec/clean.md`).
> - `README.md:48`: Change heading `## Supported Flags` → `## Supported flags` (sentence case).
> Apply these edits?

**User:**
> Apply.

*(Applied, committed, and merged.)*

### 4. Pre-release audit (`/04-audit`, `low`)

```text
> agy --effort low
```

**User:**
> `/04-audit`

**Agy (`mechanical` • `low`):**
> *Skill permission: `none` (read-only).*
>
> | Checkpoint | Status | Details |
> | :--- | :--- | :--- |
> | Working tree hygiene | Pass | Clean on `main` |
> | Automated test suite and linter | Pass | `go test -race ./...`, `go vet ./...` passed |
> | Spec ↔ Code ↔ Docs alignment | Pass | `docs/spec/clean.md`, `cmd/clean.go`, `README.md` match |
> | Release and CI artifacts | Pass | `LICENSE`, `CHANGELOG.md`, `.github/workflows/ci.yml` present |
> | **Release readiness verdict** | **GO** | No `P0` blockers |

### 5. Release (`/05-release`, `low`)

**User:**
> `/05-release`

**Agy (`mechanical` • `low`):**
> *Latest tag: `v0.3.0`. Commits since `v0.3.0` include `feat(clean): add dotctl clean subcommand` → next SemVer: `v0.4.0`.*
> *Skill permission: `CHANGELOG.md` and manifest version strings.*
>
> Updated `CHANGELOG.md` (moved `[Unreleased]` items under `## [0.4.0] - 2026-11-16`).
> **Stopped before tagging.** Create a release prep branch (`chore(release): prepare v0.4.0`), merge it into `main`, then re-run `/05-release`.

*(User branches, commits, merges PR, updates `main`, and runs `/05-release` again.)*

**Agy (`mechanical` • `low`):**
> `CHANGELOG.md` matches `0.4.0` and `main` is up to date with `origin/main`.
> - **Calculated version:** `v0.4.0` (`feat(clean)` adds a backwards-compatible subcommand).
> - **Target commit:** `a1b2c3d chore(release): prepare v0.4.0`
>
> **Proposed commands (awaiting confirmation):**
>
> ```bash
> git tag -a v0.4.0 -m "Release v0.4.0"
> git push origin v0.4.0
> ```

---

## Key points of view reflected in this walkthrough

1. **You own intent and irreversible actions; the agent owns labor:** The agent interviews on ambiguity (`/00-spec`, `/01-issue`) rather than guessing, and stops at every mutating Git, delete, or network gate.
2. **Gear shifting by stage:** `plan` + `high` for `/00-spec`, `medium` for `/01-issue`, `/02-implement`, and `/03-docs`, and `low` for `/04-audit` and `/05-release`.
3. **Spec, code, and docs stay synchronized:** `docs/spec/` is the ground truth, `.issue-draft.md` stays untracked as the local work-unit spec until merge, and trivial fixes bypass `/01-issue`.
4. **Two-step release:** `/04-audit` is strictly read-only (`GO` / `NO-GO`), and `/05-release` prepares `CHANGELOG.md` via PR first, then tags `main` on the second run.
