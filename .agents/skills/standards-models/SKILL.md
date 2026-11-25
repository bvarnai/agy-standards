---
name: standards-models
description: Review currently available AI models in Antigravity (agy), evaluate their coding precision vs. deep reasoning/synthesis strengths, and update docs/models.md. Activate when asked to review models, refresh model recommendations, or running /standards-models.
metadata:
  mode: "interactive"
  recommended_flags: "--effort medium"
---

# Model & effort compass review

## Constraints & role
- **Tone:** Follow [strict-technical-tone.md](../../../fragments/strict-technical-tone.md).
- **Permission:** None. Propose the `docs/models.md` table and apply it after confirmation.
1. **Evidence-based evaluation**: Evaluate models on practical engineering execution—surgical code-edit precision, multi-step tool-loop speed, instruction adherence, and multi-file architectural synthesis—rather than marketing claims.
2. **Scope**: Keep `skills/*/SKILL.md` frontmatter model-agnostic (`--effort` and `--mode` only); record concrete model name recommendations exclusively in [docs/models.md](../../../docs/models.md).

## Discovery & evaluation procedure

### 1. Discover available models
Gather the active candidate models from:
1. **User arguments / active selector**: Any model names or observations passed by the user (e.g., models visible in the `agy` `/model` selector across `personal` and `work` profiles).
2. **Current baseline**: Read the existing recommendations table in [docs/models.md](../../../docs/models.md).
3. **Live documentation & release notes**: Use `search_web` to check the latest Antigravity changelog (`https://antigravity.google/changelog`), CLI reference (`https://antigravity.google/docs/cli/reference`), and current frontier model releases/benchmarks for agentic coding and reasoning.
4. **Pre-flight check**: If internal or preview model names exist in the user's `/model` menu that are not publicly documented, ask the user to list the models currently shown in their `/model` selector before finalizing.

### 2. Evaluate against lifecycle task profiles
Classify the available models across the five `agy-standards` execution profiles (ordered chronologically by lifecycle stage):

| Lifecycle stages | Task profile | Primary bottleneck | Key selection criteria | Default effort |
| :--- | :--- | :--- | :--- | :--- |
| `00-spec`<br>`01-roadmap` | **1. Architecture, specs & epic slicing** | Multi-file synthesis, slicing & invariant design | Deepest reasoning, legacy reverse-engineering accuracy, long-context coherence, vertical slice decomposition. | `--mode plan --effort high` |
| `02-issue`<br>`03-implement` | **2. Hands-on coding, TDD & tool loops** | 10–20 step tool-loop latency & surgical diff quality | Strict adherence to existing code style, minimal over-abstraction, fast `edit_file` $\rightarrow$ test cycles. | `--effort medium`<br>*(or `high` / `xhigh` for race/concurrency bugs)* |
| `04-docs` | **3. Documentation tone & accuracy review** | Natural developer prose + technical verification | Avoids stiff AI phrasing while accurately cross-checking flags and examples against code. | `--effort medium` |
| `05-audit` | **4. Pre-release readiness checklist** | Deterministic checklist execution (working tree, tests/linters, manifests, CI) | Low false-positive rate and fast turnaround on the release branch without re-auditing prose or re-reading every source file. | `--effort low` |
| `06-release` | **5. Mechanical release cut** | Turnaround speed & strict procedure compliance | Fast, deterministic execution (SemVer bump, `CHANGELOG.md` update, Git tag staging) without re-auditing. | `--effort low` |

## Output contract
1. **Model evaluation summary**:
   - List the evaluated models and any changes compared to the previous [docs/models.md](../../../docs/models.md) table (which model leads for **Deep reasoning / architecture** vs. **Hands-on coding / tool loops** vs. **Fast mechanical tasks**, and why).
2. **Update [docs/models.md](../../../docs/models.md)**:
   - Propose and apply the updated recommendations table in [docs/models.md](../../../docs/models.md) (sorted chronologically by development phase `00-spec` $\rightarrow$ `06-release`) upon user confirmation.
