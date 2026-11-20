---
name: 01-roadmap
description: Partition baseline or multi-module docs/spec/ contracts into dependency-ordered vertical slices tracked in a transient docs/roadmap.md on an epic/* integration branch. Activate for greenfield bootstraps, multi-PR epics, or running /01-roadmap.
metadata:
  lifecycle_stage: "01-roadmap"
  mode: "interactive"
  recommended_flags: "--mode plan --effort high"
---

# 01 — Epic roadmap & slicing

## Constraints
- **Tone:** Follow [strict-technical-tone.md](../../fragments/strict-technical-tone.md).
- **Permission:** Create or overwrite `.issue-draft.md` in CWD; create, update, or remove `docs/roadmap.md` on an `epic/*` integration branch (never on the default branch).
- **Activation scope:** Activate only for greenfield bootstraps after `/00-spec` or multi-PR epics requiring staged vertical delivery. Skip for routine single-PR changes (proceed directly to `/plan` and `/02-issue`).

## Slicing heuristics & invariants
1. **No spec duplication (DRY):** Never restate CLI flags, schemas, error matrices, or invariants in `docs/roadmap.md`. Cite exact `docs/spec/` filenames, section headings, and `TC-*` scenario IDs instead.
2. **Just-in-time issue drafting:** Keep each slice entry in `docs/roadmap.md` to a proposed commit-style title, dependencies, spec references, verification milestone, and 2–4 scope bullets. Draft full issue bodies right before implementing each slice via `/02-issue`.
3. **Walking skeleton + vertical increments:** Make `S1` a runnable foundation with green CI (including Day-0 scaffolding for greenfield repositories), and make subsequent slices thin vertical end-to-end capabilities rather than horizontal layers (avoid *"all models"* $\rightarrow$ *"all storage"* $\rightarrow$ *"all CLI"*).
4. **100% spec-coverage gate:** Include a **Spec-coverage matrix** mapping every file, major section, and `TC-*` scenario in `docs/spec/` to an owner slice (or explicitly marking deferred non-goals).
5. **Transient lifecycle:** `docs/roadmap.md` lives strictly on `epic/<E>`, updates slice status (`Pending` $\rightarrow$ `Done (#<N>)` using the slice issue number) inside each child slice PR (parallel-slice teams may instead update `docs/roadmap.md` directly on `epic/<E>` post-merge to avoid rebase conflicts), and is removed on `epic/<E>` before merging into the default branch.

## Execution protocol

### Phase A — Preview & parent epic ticket (on clean default branch)
1. Inspect `docs/spec/` and existing build/CI files in `CWD`; verify the working tree is clean on the default branch.
2. Partition the specification into slices `S1`–`Sn` and preview `docs/roadmap.md` in chat without writing it to the default branch.
3. Write the parent epic ticket summary (milestone scope, ordered slice list `S1`–`Sn`, and end-to-end verification criteria) to `.issue-draft.md` (`Overwrite: true`).
4. When using `git-brx`, remind the operator platform-neutrally of tracker label prerequisites: ensure an `epic` label or issue type exists in the repository tracker (new GitHub repositories do not include an `epic` label by default—create one such as `epic` / `Multi-issue milestone or integration effort` / `#D97706`) and apply `epic` to parent ticket `#E`, while applying `enhancement` (or `bug` / `feature`) to subsequent child slice tickets.

### Phase B — Provision `epic/<E>` & commit roadmap (after operator provides `#E`)
1. Wait for explicit confirmation and create the integration branch: `git brx create epic/<E> -y` when `git-brx` is on `PATH` (or follow the project's branching convention otherwise).
2. Write `docs/roadmap.md` using the template below with `#E` and `epic/<E>` populated cleanly (no `<E>` placeholders).
3. Propose staging and committing on `epic/<E>` (`docs(roadmap): add <milestone> implementation roadmap`), wait for confirmation, publish `epic/<E>` (`git brx publish`) so child `issue/*` PRs can target it, and hand off `S1` to `/02-issue`.

### Phase C — Epic closure (after final slice merges into `epic/<E>`)
1. Switch to and fast-forward `epic/<E>` (`git brx select epic/<E>`, `git brx update`).
2. Verify every slice in `docs/roadmap.md` is marked `Done (#N)` and run the full stack test and linter suite on `epic/<E>`.
3. Propose removing the transient roadmap (`git rm docs/roadmap.md` and `git commit -m "chore(roadmap): remove transient roadmap before merge"`), and wait for confirmation.
4. Sync `epic/<E>` with the default branch (`git brx sync`), publish (`git brx publish`), and open the final PR targeting the default branch (`git brx review`, selecting a true non-squash merge).
5. After the epic merges, clean up (`git brx delete epic/<E>`, `git brx update`) and proceed to `/04-docs` and `/05-audit`.

## Mandatory `docs/roadmap.md` template

```markdown
# Implementation roadmap: <project> <milestone>

## Target milestone & scope

- **Milestone:** `<vX.Y.Z or epic theme>`
- **Epic:** `#<E>` (branch `epic/<E>`, true-merged to `<default>` after all slices complete and this file is removed)
- **Non-goals:** `<Explicitly deferred spec extensions or out-of-scope items>`

## Ordered slices

| Slice | Proposed issue title | Depends on | Spec refs | Verification milestone | Status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `S1` | `<type>(<scope>): <concise title>` | `—` | `<spec.md> §...` | `<Runnable milestone + checks>` | Pending |

## Slice scope notes

- **`S1` — `<short label>`:** Owns `<paths/modules>`. `<Key boundary or decoupling note>`.

## Spec-coverage matrix

| Spec document | Sections & verification scenarios | Owner slices |
| :--- | :--- | :--- |
| `docs/spec/global.md` | `<sections / TC-*>` | `S1`–`Sn` |
```
