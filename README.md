# <img src="assets/logo.svg" width="28" height="28" alt="" align="absmiddle"> agy-standards

[Google Antigravity](https://antigravity.google) (`agy`) plugin bundling global rules, lifecycle skills, and language-specific standards.

> *"Building the wrong thing is still a waste even when building is faster."* — [Gergely Orosz, LDX3 New York 2026](https://www.youtube.com/watch?v=uErdlVHb8ow)
>
> *"The old system scaled consistency. The new system must scale intent."* — [Katie Dill, Stripe](https://www.youtube.com/watch?v=GLvFTMtw4Jk)

---

## Approach

`agy-standards` structures development with `agy` around one habit: **agree on what you want to build and put it in writing before any code exists.**

That written record replaces long chat histories, where requirements get lost, and takes two forms:

- **Specifications (`docs/spec/`)**: Living requirements and design documents that describe how the project works (CLI flags, APIs, configuration, and behavior).
- **Issues**: Small, well-defined tasks that describe what needs to change, the edge cases to handle, and how to test it.

Each coding or review step can then start in a fresh session and read what it needs from these files. *(See [docs/philosophy.md](docs/philosophy.md) for why the workflow is built this way.)*

---

## Workflow

```mermaid
flowchart LR
    Spec["Spec<br/>(/standards:00-spec)"]

    subgraph Daily ["Daily development loop"]
        Plan["Plan mode<br/>(/plan)"] --> Issue["Issue<br/>(/standards:01-issue)"] --> Code["Implement<br/>(/standards:02-implement)"]
    end

    subgraph Ship ["Release pipeline"]
        Docs["Docs<br/>(/standards:03-docs)"] --> Audit["Audit<br/>(/standards:04-audit)"] --> Release["Release<br/>(/standards:05-release)"]
    end

    Spec -.->|"New project / module"| Plan
    Code -->|"Next issue"| Plan
    Code -->|"Ready to ship"| Docs
```

1. **Write or update specs (`/standards:00-spec`)**: Create initial `docs/spec/` files when starting a new project, adding a new module, or documenting existing code.
2. **Daily development loop (`Plan` $\rightarrow$ `/standards:01-issue` $\rightarrow$ `/standards:02-implement`)**:
   - **Plan (`--mode plan` / `/plan`)**: Discuss the problem and agree on the approach before touching code.
   - **Create issue (`/standards:01-issue`)**: Turn the agreed plan into a structured issue draft (`.issue-draft.md`) with acceptance criteria, edge cases, and test cases.
   - **Implement (`/standards:02-implement`)**: Create a topic branch, update `docs/spec/` and the code together, run tests and linters, and open a PR.
3. **Release pipeline (`/standards:03-docs` $\rightarrow$ `/standards:04-audit` $\rightarrow$ `/standards:05-release`)**: When ready to release, review user docs, run a read-only audit across specs, code, and tests, and cut the release tag.

*(For a step-by-step example of how a full session looks in practice, see [docs/walkthrough.md](docs/walkthrough.md).)*

### Companion tool: [`git-brx`](https://github.com/bvarnai/git-brx) (optional)

`agy-standards` shapes how the agent plans, writes specs, codes, audits, and writes commits. `git-brx` serves a different purpose in the same workflow: it covers your side of it. It brings its own branching conventions and turns them into concrete commands (`git brx create`, `sync`, `publish`, `review`, and so on) on top of Git, so you can follow the workflow without being fluent in Git. It also ships its own agent skill; see its [AI agent integration guide](https://github.com/bvarnai/git-brx/blob/main/docs/user-guide/getting-started.md).

If `git-brx` is on `PATH`, the agent uses it for branches, syncing, and PRs. If not, the agent follows the project's own conventions.

### Working effectively

- **Use plan mode as an interview.** Give a rough 2-sentence idea and ask `agy` to challenge the edge cases and trade-offs. Answering a few sharp questions triggers the same design thinking that used to happen while coding by hand. Then let `/standards:00-spec` or `/standards:01-issue` draft the Markdown from your answers.
- **Prototype when the design isn't clear yet.** Build a rough, throwaway version to learn what works, then write the real design into `docs/spec/`.
- **Use the fast path for trivial changes.** Small fixes, doc edits, and dependency bumps don't need an issue.

---

## Repository organization

```text
agy-standards/
├── plugin.json                         # Native agy plugin manifest
├── assets/logo.svg                     # Monochrome plugin mark
├── rules/                              # Universal always-on rule entrypoint
│   └── GEMINI.md                       # @-imports fragments + formatting & tool policy
├── standards/                          # Stack-specific standards (routed via fragments/stack-routing.md)
│   ├── go-cli.md                       # Go CLI architecture, error wrapping & testing
│   ├── java.md                         # Modern Java idioms, null-safety, Maven/Gradle verification
│   ├── embedded.md                     # Embedded C/C++ deterministic memory, ISR safety, HAL
│   └── shell.md                        # Bash/Shell naming (kebab-case CLI), safety, shellcheck & Bats
├── skills/                             # Lifecycle slash commands (/standards:00-spec .. /standards:05-release)
│   └── <NN-skill>/SKILL.md             # Skill definition (+ templates/ where applicable)
└── fragments/                          # Shared rule & prompt fragments (spec-driven, git, stack-routing, tone)
```

---

## Universal lifecycle skills

Run these skills inside an interactive `agy` session (assuming the plugin is linked as `standards`):

| Stage | Slash command | Purpose |
| :--- | :--- | :--- |
| **0. Spec** | `/standards:00-spec` | Author or update technical specs in `docs/spec/`. |
| **1. Issue** | `/standards:01-issue` | Draft strict, spec-driven issues. |
| **2. Implement** | `/standards:02-implement` | Implement features/fixes with spec sync & tests. |
| **3. Docs** | `/standards:03-docs` | Audit docs for tone and factual accuracy. |
| **4. Audit** | `/standards:04-audit` | Read-only pre-release readiness checklist. |
| **5. Release** | `/standards:05-release` | Prepare changelog/manifests and tag default branch. |

### Model & effort mental compass

Start your `agy` session in **Build** (`medium`) as the default, then shift up to **Design** (`high`) or down to **Ship** (`low`) when the stage calls for it:

| Gear | When to use | Stages | Model profile | Effort & mode |
| :--- | :--- | :--- | :--- | :--- |
| **1. Design** | Designing architecture, catching subtle drift, or stuck on a hard bug | `00-spec` | Strongest reasoning model | `high` *(+ `plan` for `00-spec`)* |
| **2. Build** *(default)* | Day-to-day coding, tests, issue drafting, and doc reviews | `01-issue`, `02-implement`, `03-docs` | Fast, surgical coding model | `medium` |
| **3. Ship** | Deterministic checklists and release steps where deep reasoning adds latency | `04-audit`, `05-release` | Fastest available (or same model) | `low` |

For concrete model picks and per-stage rationale, see [docs/models.md](./docs/models.md).

---

## Multi-stack domain rule routing

Always-on [`fragments/stack-routing.md`](fragments/stack-routing.md) (imported by [`rules/GEMINI.md`](rules/GEMINI.md)) inspects the current working directory (`CWD`) during planning, issue drafting, implementation, and auditing to load the matching standard:

- **Go**: Triggered by `go.mod` $\rightarrow$ enforces [standards/go-cli.md](standards/go-cli.md) (`go test -race ./...`, `go vet ./...`).
- **Java**: Triggered by `pom.xml` / `build.gradle` $\rightarrow$ enforces [standards/java.md](standards/java.md) (`./mvnw test` / `./gradlew check`).
- **Embedded C/C++**: Triggered by `CMakeLists.txt` / `platformio.ini` / `Kconfig` / `*.ld` $\rightarrow$ enforces [standards/embedded.md](standards/embedded.md) (deterministic memory, fixed-width types, ISR safety).
- **Shell / Bash**: Triggered by `*.sh` / `*.bats` / `.bashrc.d/` $\rightarrow$ enforces [standards/shell.md](standards/shell.md) (`kebab-case` public interfaces, `shellcheck`, `bats`).

To add a new stack (e.g. Rust, Python), create `standards/<stack>.md` and add one row to `fragments/stack-routing.md`.

---

## Configuring Antigravity (`agy`)

`agy-standards` is packaged as a native Antigravity plugin (`plugin.json`).

- **Install from GitHub (`agy` >= 1.5.11):**

  ```bash
  agy plugin install https://github.com/bvarnai/agy-standards
  ```

- **Install from a local clone (symlink):** Linking the repository into `~/.gemini/config/plugins/` registers both the global rules (`rules/`) and all lifecycle skills (`skills/`) with a single directory symlink:

  ```bash
  mkdir -p ~/.gemini/config/plugins
  ln -sfn "<path-to-agy-standards>" ~/.gemini/config/plugins/standards
  ```

---

## 🤖 AI Disclosure

![AI Assisted](https://img.shields.io/badge/AI-Assisted-blue?style=flat-square)

This project was developed with the assistance of AI tools. All AI-generated code has been reviewed, tested, and manually refined.
