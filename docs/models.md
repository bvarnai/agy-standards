# Model & effort recommendations

Detailed model picks and per-stage rationale for `agy` sessions. For the high-level 3-gear summary (**Think / Build / Mechanical**), see [README.md](../README.md#model--effort-mental-compass).

Choose the **model (`--model` / `/model`)** for its base instincts (**Gemini 4 Argon** for deep architectural synthesis, **Gemini 4 Hydrogen** for surgical coding and fast tool loops, **Gemini 3.8 Flash** for low-cost mechanical tasks) and **effort (`--effort`)** for per-turn thinking depth:

| Engineering goal | Lifecycle stages | Recommended model | Effort & mode | Why this combination works best |
| :--- | :--- | :--- | :--- | :--- |
| **System architecture, specs & cross-repo uplift** | `00-spec`<br>`06-harmonize` | **Gemini 4 Argon** | `--mode plan --effort high` | Deepest multi-file reasoning, legacy reverse-engineering, and interface/invariant synthesis before writing code. |
| **Pre-release & cross-artifact audit** | `04-audit` | **Gemini 4 Argon**<br>*(or **Gemini 4 Hydrogen** for branch diffs)* | `--effort high`<br>*(or `medium` for branch diffs)* | High-recall cross-checking across specs, code, docs, and CI with low false positives; use `Hydrogen` on routine diffs to save cost. |
| **Hands-on coding, TDD & tool execution loops** | `02-implement`<br>`01-issue` | **Gemini 4 Hydrogen**<br>*(or **Orchestrator (Argon)** for large multi-module plans)* | `--effort medium`<br>*(or `high` / `xhigh` for race/concurrency bugs)* | Built for fast 10–20 step `edit_file` $\rightarrow$ test loops with surgical diffs at much lower cost than `Argon`. |
| **Documentation tone & accuracy review** | `03-docs` | **Gemini 4 Hydrogen**<br>*(or **Gemini 3.8 Flash**)* | `--effort medium` | Verifies flags/examples against code while keeping developer prose direct, natural, and free of stiff AI phrasing. |
| **Mechanical release cut** | `05-release`<br>*(plus trivial `chore:`/`docs:` issues)* | **Gemini 3.8 Flash**<br>*(or **Gemini 4 Hydrogen**)* | `--effort low` | Lowest cost and fastest turnaround for deterministic steps (`CHANGELOG.md`, manifest/CI checks, Git tag) without over-thinking. |
