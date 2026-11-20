# Example workflow session

This walkthrough shows a complete feature-to-release cycle in `agy` with `agy-standards` enabled. It uses a fictional Go CLI (`dotctl`).

---

## Stage 0 — Spec authoring

```bash
> /plan
> /effort high
```

🧑 You:

```bash
> /standards:00-spec we want to add a "dotctl clean" subcommand that removes broken symlinks from "$HOME"
```

🤖 Antigravity:

> ▸ Thought for 7s
>
>  Loaded `agy-standards` (`go.mod` detected → `standards/go.md`). Read `README.md`, `go.mod`, and `docs/spec/global.md`.
>
>  Before updating `docs/spec/`, three edge cases need a decision:
>  1. **Scope:** Should `dotctl clean` scan `$HOME` non-recursively, recurse into all directories, or only scan paths managed in `dotctl.yaml`?
>  2. **Confirmation and safety:** Should it delete immediately, or require `--yes` or default to `--dry-run`?
>  3. **Exit code:** If broken symlinks are found in `--dry-run` mode, should it exit `0` or non-zero?

🧑 You:

```bash
> 1. Only scan target paths declared in `dotctl.yaml` (never walk arbitrary `$HOME` dirs).
  2. Delete only with `--force` (or interactive prompt); `--dry-run` prints candidates and exits `0`.
  3. Exit `2` on invalid config, `1` on filesystem I/O error.
```

🤖 Antigravity:

> ▸ Thought for 2s
>
>  Updated `docs/spec/global.md` (subcommand registry and exit code `2` invariant) and created `docs/spec/clean.md` (flags, `Runner` / `Inspector` boundary interface, error matrix).
>
>  Skipped optional sections in `clean.md`: Legacy discrepancies (greenfield subcommand).

---

## Stage 1 — Epic roadmap *(greenfield / multi-PR epics)*

*(Skip this stage for single-PR changes and jump to [Stage 2](#stage-2--issue-drafting). Run it after `/standards:00-spec` when bootstrapping a project or breaking a multi-module spec into staged PRs.)*

```bash
> /plan
> /effort high
```

🧑 You:

```bash
> /standards:01-roadmap slice the baseline specs for v0.1.0
```

🤖 Antigravity:

> ▸ Thought for 9s
>
>  Inspected `docs/spec/` on clean `main`. Previewed a 2-slice roadmap (`S1`: walking skeleton + CI with `epic/**` trigger; `S2`: `clean` command + integration tests) with a 100% spec-coverage matrix.
>
>  Drafted parent epic `#3` summary to `.issue-draft.md`. Create `#3` in your tracker with label `epic` (create the `epic` label first if your repository does not have one), then reply with the ticket number.

🧑 You:

```bash
> created #3 with the epic label
```

🤖 Antigravity:

> ▸ Thought for 6s
>
>  Confirm creating `epic/3` (`git brx create epic/3 -y`), committing `docs/roadmap.md`, and publishing `epic/3`?

*(User confirms. Antigravity creates `epic/3`, commits `docs(roadmap): add v0.1.0 implementation roadmap`, runs `git brx publish`, and proceeds to Stage 2 for Slice `S1`.)*

When Slice `S1` (`issue/4`, forked off `epic/3`) passes tests, Antigravity marks `S1` as `Done (#4)` in `docs/roadmap.md`, runs `git brx sync epic/3`, `git brx publish`, and `git brx review epic/3` to target `epic/3` directly. After merging PR `#5`, the user switches back to `epic/3` (`git brx select epic/3 && git brx update && git brx delete issue/4`) for `S2`. Once all slices land, Antigravity deletes `docs/roadmap.md` on `epic/3`, runs `git brx sync && git brx publish && git brx review` for a true merge to `main`, and cleans up (`git brx delete epic/3 && git brx update`).

---

## Stage 2 — Issue drafting

```bash
> /effort medium
```

🧑 You:

```bash
> /standards:02-issue add the clean subcommand from the spec
```

*(If you run `/standards:02-issue` with no description in a fresh session, the agent asks 1–3 clarifying questions first; otherwise it derives the commit-format title from context.)*

🤖 Antigravity:

> ▸ Thought for 11s
>
>  Read `docs/spec/global.md` and `docs/spec/clean.md`. Drafted issue to `.issue-draft.md`:
>
>   - **Title:** `feat(clean): add dotctl clean subcommand`
>   - **Design:** Implement `cmd/clean.go` and `internal/clean/clean.go` using injected `Inspector` / `Remover` interfaces.
>   - **Acceptance criteria:**
>     - Scans only target paths from `dotctl.yaml`.
>     - `--dry-run` lists broken symlinks without deleting and exits `0`.
>     - `--force` removes broken symlinks without prompt; without `--force`, prompts `[y/N]`.
>   - **Edge cases and failure modes:** Valid symlinks untouched; missing target directory skipped with warning; permission error returns exit `1`; invalid `dotctl.yaml` returns exit `2`.
>   - **Verification:** `go test -race ./...` and `go vet ./...`.
>
>  Create the ticket in your tracker from `.issue-draft.md`, then run `/standards:03-implement`.

---

## Stage 3 — Implementation

```bash
> /effort medium
```

🧑 You:

```bash
> /standards:03-implement implement issue from @[.issue-draft.md]
```

🤖 Antigravity:

> ▸ Thought for 18s
>
>  Checked `docs/spec/global.md`, `docs/spec/clean.md`, `.issue-draft.md`.
>
>  Confirm to create the topic branch?

🧑 You:

```bash
> Yes
```

🤖 Antigravity:

> ▸ Thought for 52s
>
>  Switched to topic branch.
>
>  1. Implemented `internal/clean/clean.go` (`Inspector` / `Remover` interfaces) and `cmd/clean.go`.
>  2. Added unit and table-driven tests in `internal/clean/clean_test.go`.
>  3. Updated `README.md` subcommand table (`docs/spec/clean.md` already matches).
>  4. Ran `go test -race ./...` and `go vet ./...` — all passed (0 warnings).
>
>  **Acceptance criteria verification matrix:**
>
>  | Acceptance criterion / edge case | Status | Verifying test / file |
>  | :--- | :--- | :--- |
>  | Scans only `dotctl.yaml` paths | Pass | `internal/clean/clean_test.go:TestClean_ConfiguredPathsOnly` |
>  | `--dry-run` exits `0` without deleting | Pass | `internal/clean/clean_test.go:TestClean_DryRun` |
>  | `--force` skips prompt; default prompts `[y/N]` | Pass | `internal/clean/clean_test.go:TestClean_Confirmation` |
>  | Permission error exits `1`; bad config exits `2` | Pass | `internal/clean/clean_test.go:TestClean_ErrorCodes` |
>
>  **Proposed commit:**
>
>  ```bash
>  git add cmd/clean.go internal/clean/ README.md
>  git commit -m "feat(clean): add dotctl clean subcommand"
>  ```
>
> Confirm commit and push/publish?

🧑 You:
```bash
> commit and publish
```

🤖 Antigravity:

> ▸ Thought for 22s
>
>  Committed on topic branch and published. Reviewed via PR. Once merged, remove `.issue-draft.md` and sync `main`.

---

## Stage 4–6 — Release pipeline

### 4. Documentation review

🧑 You:

```bash
> /standards:04-docs review README.md
```

🤖 Antigravity:

> ▸ Thought for 22s
>  Found the following review points:
>
>  - `README.md:44`: Replace *"seamlessly cleans up"* with *"removes broken symlinks declared in `dotctl.yaml`"* (cut fluff, match `docs/spec/clean.md`).
>  - `README.md:48`: Change heading `## Supported Flags` → `## Supported flags` (sentence case).
>
>  Apply these edits?

🧑 You:
```bash
> apply
```

*(Applied, committed, and merged.)*

### 5. Pre-release audit

```bash
> /effort low
```

🧑 You:
```bash
> /standards:05-audit
```

🤖 Antigravity:

> ▸ Thought for 22s
>
> All release checkpoints are passed:
>
>  | Checkpoint | Status | Details |
>  | :--- | :--- | :--- |
>  | Working tree hygiene | Pass | Clean on `main` |
>  | Automated test suite and linter | Pass | `go test -race ./...`, `go vet ./...` passed |
>  | Spec ↔ Code ↔ Docs alignment | Pass | `docs/spec/clean.md`, `cmd/clean.go`, `README.md` match |
>  | Release and CI artifacts | Pass | `LICENSE`, `CHANGELOG.md`, `.github/workflows/ci.yml` present |
>  | **Release readiness verdict** | **GO** | No `P0` blockers |

### 6. Release

🧑 You:
```bash
> /standards:06-release
```

🤖 Antigravity:

> ▸ Thought for 19s
>
>  Latest tag is `v0.3.0`. Commits since `v0.3.0` include `feat(clean): add dotctl clean subcommand` → next SemVer: `v0.4.0`.
>
>  Updated `CHANGELOG.md` (moved `[Unreleased]` items under `## [0.4.0] - 2026-11-16`). **Stopped before tagging.** Create a release prep branch (`chore(release): prepare v0.4.0`), merge it into `main`, then re-run `/standards:06-release`.

*(User branches, commits, merges PR, updates `main`, and runs `/standards:06-release` again.)*

🤖 Antigravity:

> ▸ Thought for 11s
>
>  `CHANGELOG.md` matches `0.4.0` and `main` is up to date with `origin/main`.
>
>  - **Calculated version:** `v0.4.0` (`feat(clean)` adds a backwards-compatible subcommand).
>  - **Target commit:** `a1b2c3d chore(release): prepare v0.4.0`
>
>  **Proposed commands (awaiting confirmation):**
>
>  ```bash
>  git tag -a v0.4.0 -m "Release v0.4.0"
>  git push origin v0.4.0
>  ```
