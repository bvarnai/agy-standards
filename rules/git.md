# Commit conventions

## Commit messages
- **Format:** `<type>(<component>): <description>` in lowercase imperative mood, at most 72 characters, no trailing period (e.g., `feat(cli): add --dry-run flag`). The scope names the component that changed, never an issue number.
- **Allowed types:** `feat`, `fix`, `refactor`, `perf`, `docs`, `test`, `ci`, `build`, `chore`.
- **Breaking changes:** append `!` after the type or scope (`feat(cli)!: ...`) and add a `BREAKING CHANGE: <details>` footer.
- **Submodules:** commit submodule pointer updates in their own commit (`chore(submodule): bump <name>`), only after the referenced commit is pushed to the submodule remote.

## Branches, syncing, PRs, and merging
When `git-brx` is on `PATH`, **always** use `git brx` instead of plain `git` or `gh` for branch and PR lifecycle operations:
- **Create branch:** `git brx create <type>/<id>` (never `git checkout -b` / `git switch -c`)
- **Switch branch:** `git brx select [<branch>]` (never `git checkout` / `git switch`)
- **Sync / update:** `git brx update` / `git brx sync` (never `git pull` / `git rebase`)
- **Publish branch:** `git brx publish` (never `git push` for branches)
- **Open PR:** `git brx review` (never `gh pr create`)
- **Delete merged branch:** `git brx delete` (never `git branch -d`)

Only use plain `git` for staging/committing (`git add`, `git commit`), tags (`git tag`, `git push origin <tag>`), or when `git-brx` is not on `PATH`.
