# Model & effort recommendations

Detailed model picks and per-stage rationale for `agy` sessions. For the high-level 3-gear summary (**Design / Build / Ship**), see [README.md](../README.md#model--effort-mental-compass).

Choose the **model (`--model` / `/model`)** for its base instincts (**Gemini 4 Argon** for deep architectural synthesis, **Gemini 4 Hydrogen** / **Gemini 4 Neon** for surgical coding and fast tool loops, **Gemini 3.8 Flash** for low-cost mechanical tasks) and **effort (`--effort`)** for per-turn thinking depth:

| Lifecycle stages | Engineering goal | Recommended model | Effort & mode | Why this combination works best |
| :--- | :--- | :--- | :--- | :--- |
| `00-spec`<br>`01-roadmap` | **System architecture, specs & epic slicing** | **Gemini 4 Argon**<br>*(or **Gemini 4 Neon** / **Claude Opus 5.5**)* | `--mode plan --effort high` | Deepest multi-file reasoning, legacy reverse-engineering, interface/invariant synthesis, and vertical slice decomposition before writing code. |
| `02-issue`<br>`03-implement` | **Hands-on coding, TDD & tool execution loops** | **Gemini 4 Hydrogen** / **Gemini 4 Neon**<br>*(or **Orchestrator (Argon / Neon)** for multi-module plans)* | `--effort medium`<br>*(or `high` / `xhigh` for race/concurrency bugs)* | Built for fast 10–20 step `edit_file` $\rightarrow$ test loops with surgical diffs on daily-reset quota buckets while preserving `Argon` quota. |
| `04-docs` | **Documentation tone & accuracy review** | **Gemini 4 Hydrogen**<br>*(or **Gemini 3.8 Flash**)* | `--effort medium` | Verifies flags/examples against code while keeping developer prose direct, natural, and free of stiff AI phrasing. |
| `05-audit` | **Pre-release & cross-artifact audit** | **Gemini 4 Argon**<br>*(or **Gemini 4 Hydrogen** / **Neon** for branch diffs)* | `--effort high`<br>*(or `medium` for branch diffs)* | High-recall cross-checking across specs, code, docs, and CI with low false positives; use `Hydrogen` or `Neon` on routine diffs to save `Argon` quota. |
| `06-release`<br>*(plus trivial `chore:`/`docs:`)* | **Deterministic release cut** | **Gemini 3.8 Flash**<br>*(or **Gemini 4 Hydrogen**)* | `--effort low` | Lowest cost and fastest turnaround for deterministic steps (`CHANGELOG.md`, manifest updates, Git tag) without over-thinking. |
