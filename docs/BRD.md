# Business Requirements Document (BRD)
## Zero-Cost Resilient Agentic Engineering Platform: Oh My Pi (`omp`) + Oh My Hermes (`omh`) on OpenRouter Free Tier

| Document Metadata | Details |
| :--- | :--- |
| **Document ID** | `BRD-OMH-OMP-OR-2026-001` |
| **Version** | `1.0.0` |
| **Date** | October 26, 2026 |
| **Target Environment** | Android / Termux (`aarch64` + `glibc-runner`) & Linux Workspaces |
| **Core Stack** | Oh My Pi (`omp` v18.12.6), Hermes Agent + Oh My Hermes (`omh`), OpenRouter API Gateway |
| **Cost Policy** | **Strict $0.00 Variable Token Spend (`*:free` & `openrouter/free` Only)** |

---

## 1. Executive Summary

Modern agentic coding and multi-agent software delivery workflows (such as parallel DAG execution, architectural planning, automated code review, and iterative test repair) consume millions of tokens per session. When bound to commercial pay-per-token APIs, parallel subagent fanout (`dag-ulw` in Oh My Hermes or `task` subagents in Oh My Pi) creates unpredictable and prohibitive operational expenses.

At the same time, relying on a single free-tier model endpoint introduces severe availability bottlenecks: upstream providers enforce strict Requests-Per-Minute (RPM), Tokens-Per-Minute (TPM), and daily quota limits (`HTTP 429 Too Many Requests`, `HTTP 502/503 Provider Overloaded`).

This **Business Requirements Document (BRD)** defines the requirements for a **Zero-Cost, Rate-Limit-Resilient Agentic Workflow Architecture** that combines:
1. **Oh My Hermes (`omh`) + Hermes Agent** as the **Orchestration, Memory, and DAG Workflow Control Plane** (`Understand ➔ Design ➔ Build ➔ Check ➔ Ship ➔ Learn`, `/dag-ulw` parallel graph execution, evidence-gated completion, and task-scoped capability projection).
2. **Oh My Pi (`omp`)** as the **High-Efficiency Terminal Coding & Execution Harness** (hashline edits, native Rust `pi-natives` grep/glob/AST tools, LSP integration, and role-based subagent execution).
3. **OpenRouter Free-Tier Multi-Model Cascade** as the **Unified Inference Layer**, strictly enforcing `$0.00` token spend via `:free` model allowlists while guaranteeing continuous workflow execution through exponential backoff, usage-reset waiting, and deterministic multi-provider model fallback chains.

---

## 2. Business Objectives & Success Metrics

### 2.1 Primary Business Objectives
1. **Zero Variable Inference Cost ($0.00/Token Guarantee):**
   - Strictly restrict all agent roles, subagents, and workflow lanes across both `omp` and `omh` to OpenRouter's 70+ free-tier models (`openrouter/*:free` and `openrouter/openrouter/free`).
   - Prevent accidental fallback or escalation to paid models under all runtime conditions.
2. **High-Availability Under Rate Limits (99.5%+ Task Completion):**
   - Absorb upstream `HTTP 429` (rate limit), `HTTP 402/403` (quota exhaustion), and `HTTP 502/503` (provider capacity) errors automatically without aborting active coding turns or `/dag-ulw` graph nodes.
   - Automatically rotate across heterogeneous free model families (Qwen, DeepSeek, GLM, MiniMax, Kimi, Nemotron, GPT-OSS, StepFun, Gemma) hosted by independent upstream providers so a single provider's rate limit never halts the workflow.
3. **Structured, Evidence-Gated Agentic Delivery (`oh-my-hermes`):**
   - Replace ad-hoc prompting with `oh-my-hermes`'s deterministic lifecycle (`Plan · not run` ➔ `Code · reported done` ➔ `Test · verified`).
   - Enforce strict write-scope isolation across parallel `/dag-ulw` graph lanes and require verifiable test/check output before marking any task complete.
4. **Native Execution on Constrained Edge/Mobile Hardware (Termux ARM64):**
   - Run the complete agentic stack natively inside Android Termux (`aarch64` with `glibc-runner`) without requiring heavy cloud VMs.

### 2.2 Key Performance Indicators (KPIs)

| Metric ID | KPI Name | Target Metric | Measurement Method |
| :--- | :--- | :--- | :--- |
| **KPI-01** | Token Spend per Session | **$0.00 USD** | OpenRouter activity log & `omp usage --json` |
| **KPI-02** | Rate-Limit Recovery Rate | **≥ 99.5%** of `HTTP 429` events recovered automatically | `omp` retry telemetry & `~/.omh/routing/route-provenance.json` |
| **KPI-03** | Mean Failover Latency | **< 3.5 seconds** to switch to next free model in chain | `omp` turn logs (`~/.omp/logs/`) |
| **KPI-04** | False "Done" Hallucination Rate | **0%** (strictly blocked by OMH evidence gates) | `omh` verification gate receipts |
| **KPI-05** | Parallel Lane Write Collisions | **0** concurrent write conflicts during `/dag-ulw` runs | OMH DAG scheduler write-scope lock validation |

---

## 3. Scope Definition

### 3.1 In-Scope
- **Strict Free-Model Governance:** Catalog filtering (`enabledModels`), provider locking (`enabledProviders: ["openrouter"]`), and zero-cost verification across `omp` and `omh`.
- **Multi-Tier Rate-Limit Retry & Fallback Mechanism:**
  - In-place exponential backoff with jitter and `Retry-After` / usage-reset header awareness.
  - Role-specific (`default`, `smol`, `slow`, `plan`) and category-specific (`ultrabrain`, `deep`, `architect`, `quick`, `visual-engineering`, etc.) ordered fallback chains across diverse `:free` models.
  - Terminal catch-all fallback to `openrouter/openrouter/free` (OpenRouter's dynamic free meta-router).
  - Automatic cooldown reversion (`cooldown-expiry`) back to the primary free model once rate-limit windows reset.
- **Oh My Hermes (`omh`) Agentic Workflow Integration:**
  - Full category-to-free-model mapping (`~/.omh/routing/model-chains.json`).
  - `/dag-ulw` parallel graph decomposition, write-scope isolation, and independent verification node execution.
  - Task-scoped capability projection (`omh capabilities project`) and curated long-term memory admission.
  - Delegation handoff from `oh-my-hermes` to `oh-my-pi` (`omp`) for code execution.

### 3.2 Out-of-Scope
- Paid model provisioning, credit auto-top-up, or commercial API key billing integration.
- Hosting local high-VRAM LLM weights (e.g., local 70B `ollama`/`llama.cpp` servers) on the mobile device.
- Unsupervised destructive operations in production environments (OMH human-in-the-loop guardrails remain mandatory for irreversible actions).

---

## 4. Personas & End-to-End User Journeys

### 4.1 Personas
1. **Lead Developer / Operator (User):** Initiates high-level goals (`/dag-ulw`, `omp --goal`), reviews architectural plans, and approves irreversible operations.
2. **OMH Orchestrator & Meta-Router (`oh-my-hermes`):** Classifies incoming requests across 6 capability families, decomposes goals into DAG nodes, assigns work categories (`architect`, `ultrabrain`, `deep`, `quick`, etc.), and enforces evidence boundaries.
3. **OMP Coding Executor (`oh-my-pi`):** Executes scoped coding tasks using token-efficient hashline edits, native Rust search (`grep`/`glob`), LSP diagnostics, and managed bash execution.
4. **Verifier / Advisor Subagent:** Independently runs tests, linters, and verification gates on completed code nodes before allowing a task to transition to `Test · verified`.

### 4.2 Core User Journey: Parallel Feature Implementation via `/dag-ulw`
1. **Trigger:** User submits a feature request via `omh` (`/dag-ulw "Implement JWT authentication middleware and unit tests"`) or `omp --goal`.
2. **Task-Scoped Projection & Decomposition:**
   - `omh` projects only the relevant skills within the task's context budget (`omh_run_context_budget/v1`).
   - The `architect` lane (`openrouter/qwen/qwen3.6-plus:free`) decomposes the goal into non-overlapping DAG nodes:
     - **Node A (Write Scope: `src/auth/**`):** Implement middleware (`deep` lane ➔ `openrouter/qwen/qwen3-coder:free`).
     - **Node B (Write Scope: `docs/auth.md`):** Draft API documentation (`writing` lane ➔ `openrouter/minimax/minimax-m3:free`).
     - **Node C (Depends on A, Write Scope: `tests/auth/**`):** Implement & run test suite (`capable` lane ➔ `openrouter/deepseek/deepseek-v4-flash:free`).
3. **Rate-Limit Encounter & Seamless Failover:**
   - Mid-execution on Node A, `openrouter/qwen/qwen3-coder:free` returns `HTTP 429 Too Many Requests` (upstream provider RPM limit reached).
   - The retry engine inspects the error, places `qwen3-coder:free` on a temporary cooldown timer, and immediately routes the turn to the next candidate in the chain (`openrouter/qwen/qwen3.6-plus:free` ➔ `openrouter/deepseek/deepseek-v4-flash:free`) without losing conversation state or tool outputs.
4. **Verification Gate & Evidence Boundary:**
   - Once Nodes A, B, and C report `Code · reported done`, a dedicated verification node executes the test suite.
   - Only after the test runner exits `0` does OMH transition the status to `Test · verified` and present the final delivery report.

---

## 5. Business & Functional Requirements

### 5.1 Model & Cost Governance Requirements (`FR-GOV`)

| Req ID | Priority | Requirement Description | Acceptance Criteria |
| :--- | :--- | :--- | :--- |
| **FR-GOV-01** | **P0 (Critical)** | **Strict Free-Model Allowlist:** The system MUST restrict all model discovery, interactive selection (`/model`), and automated role resolution to free OpenRouter models (`openrouter/*:free` and `openrouter/openrouter/free`). | `omp config get enabledModels` returns `["openrouter/*:free", "openrouter/openrouter/free"]`; paid models cannot be selected by subagents. |
| **FR-GOV-02** | **P0 (Critical)** | **Provider Isolation:** Only `openrouter` SHALL be enabled as an active inference provider unless explicitly reconfigured by the operator. | `omp config get enabledProviders` returns `["openrouter"]`. |
| **FR-GOV-03** | **P1 (High)** | **Tool Calling & Context Floor:** Every primary free model assigned to an agentic coding role MUST support native tool/function calling and provide a minimum context window of **128,000 tokens** (with 256K–1M preferred for planning/deep roles). | All configured role heads and fallback candidates have `contextWindow >= 131072` in the OpenRouter catalog. |
| **FR-GOV-04** | **P1 (High)** | **Upstream Provider Diversity:** Each fallback chain MUST span at least **4 distinct model families/providers** (e.g., Qwen, DeepSeek, Z.ai GLM, MiniMax, Moonshot Kimi, NVIDIA Nemotron, OpenAI GPT-OSS) so a provider-wide rate limit does not exhaust the chain. | No two adjacent fallback candidates in a chain depend on the exact same single upstream model endpoint. |

### 5.2 Rate-Limit Retry & Fallback Requirements (`FR-RES`)

| Req ID | Priority | Requirement Description | Acceptance Criteria |
| :--- | :--- | :--- | :--- |
| **FR-RES-01** | **P0 (Critical)** | **Automatic Retry with Exponential Backoff:** On transient network errors or rate limits (`HTTP 429`, `502`, `503`, `504`), the agent MUST retry up to `maxRetries = 10` times with exponential backoff (`baseDelayMs = 1500`, `maxDelayMs = 60000`). | Transient 429/5xx errors do not terminate the session; retries log backoff intervals in `~/.omp/logs/`. |
| **FR-RES-02** | **P0 (Critical)** | **Multi-Model Fallback Cascade:** When a model hits a rate limit or quota wall, `retry.modelFallback = true` (`fallbackOn: any`) MUST automatically advance to the next `:free` model in that role's `fallbackChains`. | When the primary model returns 429, the turn completes using the secondary free model in the chain. |
| **FR-RES-03** | **P1 (High)** | **Usage-Reset Synchronization:** When an upstream response includes rate-limit reset metadata (`Retry-After` or `X-RateLimit-Reset`), `retry.waitForUsageReset = true` SHALL respect the reset window before re-probing the throttled model. | Throttled models are not spammed before their cooldown window expires. |
| **FR-RES-04** | **P1 (High)** | **Automatic Cooldown Reversion:** Once a throttled primary model's cooldown expires (`fallbackRevertPolicy: cooldown-expiry`), subsequent turns MUST automatically revert from the fallback model back to the primary head model. | Session automatically returns to the optimal head model after cooldown expiry. |
| **FR-RES-05** | **P1 (High)** | **Meta-Router Last Resort:** Every fallback chain MUST terminate with `openrouter/openrouter/free` so OpenRouter's server-side free router can dynamically select any healthy free backend if all named candidates are saturated. | `openrouter/openrouter/free` is the final element of every role/category fallback array. |

### 5.3 Oh My Hermes (`omh`) Workflow Requirements (`FR-OMH`)

| Req ID | Priority | Requirement Description | Acceptance Criteria |
| :--- | :--- | :--- | :--- |
| **FR-OMH-01** | **P0 (Critical)** | **Category-Based Free Model Chains:** All 13 `oh-my-hermes` routing categories (`ultrabrain`, `deep`, `architect`, `unspecified-high`, `unspecified-low`, `quick`, `writing`, `visual-engineering`, `artistry`, `capable`, `simple-work`, `deep-work`, `last_resort.any`) MUST be configured in `~/.omh/routing/model-chains.json` using free OpenRouter models and valid reasoning effort rungs. | `omh model-chains show` (or JSON validation against `mixture_chain_overrides/v1`) succeeds with zero paid models. |
| **FR-OMH-02** | **P0 (Critical)** | **DAG-ULW Parallel Execution & Write Isolation:** Complex goals invoked via `/dag-ulw` MUST decompose into a Directed Acyclic Graph where concurrently executing nodes never share overlapping file write scopes. | Parallel nodes operate on disjoint file globs/directories; write-scope violations are rejected prior to dispatch. |
| **FR-OMH-03** | **P0 (Critical)** | **Strict Evidence Boundaries:** Agents MUST NOT report planned or unverified code as completed. Every task status MUST distinguish between `Plan · not run`, `Code · reported done`, and `Test · verified`. | Tasks only reach `verified` after an independent verification node or test command exits `0`. |
| **FR-OMH-04** | **P1 (High)** | **Task-Scoped Capability Projection:** Rather than injecting all 100+ `omh-*` skills into every prompt (which wastes context tokens and triggers free-tier TPM limits), `omh` MUST project only task-relevant capabilities within `omh_run_context_budget/v1`. | Context overhead from skill definitions remains within the per-task byte budget. |
| **FR-OMH-05** | **P1 (High)** | **Audited Long-Term Memory Admission:** Memory candidates MUST be explicitly evaluated (kept, refused, or deferred with a written reason) and recalled within a strict token budget. | Memory writes include admission rationale; next-session recall packs respect token caps. |
| **FR-OMH-06** | **P1 (High)** | **OMP Executor Handoff:** When `oh-my-hermes` delegates coding implementation work, it SHALL support dispatching tasks to `omp` (`oh-my-pi`) using hashline edits (`edit.mode: hashline`) to minimize output token consumption on free models. | Coding handoffs execute via `omp` with `hashline` edit mode enabled. |

---

## 6. Non-Functional Requirements (`NFR`)

1. **NFR-PERF-01 (Token Efficiency):** Code edits MUST use `omp`'s `hashline` edit mode (`edit.mode = hashline`) and context compaction (`snapcompact`) to reduce output token volume by 40–70%, directly reducing the likelihood of hitting OpenRouter free-tier Tokens-Per-Minute (TPM) limits.
2. **NFR-SEC-01 (Credential Hygiene):** `OPENROUTER_API_KEY` MUST be stored with `0600` file permissions (`~/.omp/env`) and/or inside `omp`'s local SQLite credential vault (`~/.omp/agent/agent.db`). No secrets may be written to `config.yml`, `model-chains.json`, or git-tracked files.
3. **NFR-SEC-02 (Human-in-the-Loop Guardrails):** Irreversible operations (`git push --force`, production deployments, external publishing, destructive filesystem deletes outside workspace) MUST require explicit human confirmation.
4. **NFR-COMPAT-01 (Termux ARM64 Compatibility):** All CLI binaries and subprocess shells MUST operate cleanly inside Android Termux (`aarch64`) via `glibc-runner` (`ld.so`) with Bionic `libtermux-exec.so` restored for child shell processes (`omp-shell`).

---

## 7. Risk Matrix & Mitigations

| Risk ID | Risk Description | Likelihood | Impact | Mitigation Strategy |
| :--- | :--- | :--- | :--- | :--- |
| **RSK-01** | All free models from a single vendor (e.g., Qwen) hit global rate limits simultaneously. | High | Medium | Interleave distinct vendors in every fallback chain (`Qwen ➔ DeepSeek ➔ GLM ➔ MiniMax ➔ Kimi ➔ Nemotron ➔ openrouter/free`). |
| **RSK-02** | Free-tier account hits OpenRouter's per-minute or daily request cap across `:free` endpoints. | Medium | High | Enable `retry.waitForUsageReset: true`, `retry.maxDelayMs: 60000`, and `edit.mode: hashline` + `read.summarize.enabled: true` to minimize requests per task. |
| **RSK-03** | A free model does not support a requested `reasoning_effort` rung (e.g., `xhigh` on a model that only supports `low,medium,high`). | Medium | Low | Both `omp` and `omh` (`EFFORT_FLOOR_KIND` / ladder normalization) automatically clamp or map off-ladder reasoning efforts to the nearest supported rung. |
| **RSK-04** | Parallel `/dag-ulw` lanes burst too many concurrent requests to OpenRouter's free tier, triggering immediate `HTTP 429` cascades. | High | Medium | Cap `providers.maxInFlightRequests` (`openrouter: 3`) and `task.maxConcurrency: 4` in `omp` so parallel subagents queue gracefully instead of stampeding the API. |
