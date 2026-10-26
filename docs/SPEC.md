# Technical Architecture & Implementation Specification
## Oh My Pi (`omp`) + Oh My Hermes (`omh`) with OpenRouter Free-Tier Cascade & Rate-Limit Recovery

| Specification Metadata | Details |
| :--- | :--- |
| **Spec ID** | `SPEC-OMH-OMP-OR-2026-001` |
| **Related BRD** | [BRD.md](./BRD.md) (`BRD-OMH-OMP-OR-2026-001`) |
| **Version** | `1.1.0` (Verified Live against OpenRouter `/api/v1/models`) |
| **Status** | Live & Smoke-Tested (`OMP_FREE_ROUTER_READY`) |

---

## 1. System Architecture Overview

The architecture separates the **Cognitive Orchestration & Workflow Layer** (**Hermes Agent + Oh My Hermes (`omh`)**) from the **Terminal Execution & Code Manipulation Harness** (**Oh My Pi (`omp`)**), while routing 100% of LLM inference requests through **OpenRouter's Live Free-Tier Model Pool** with multi-stage rate-limit retry and automatic model failover.

```mermaid
flowchart TD
    User["User / Operator (Termux CLI / TUI)"]

    subgraph ControlPlane["1. Orchestration & Workflow Control Plane (Oh My Hermes + Hermes Agent)"]
        OMHRouter["OMH Meta-Router & Capability Projection<br/>(omh_capability_projection/v1)"]
        DAG["/dag-ulw Graph Scheduler<br/>(Disjoint Write-Scope Lanes)"]
        OMHChains["OMH Category Model Resolver<br/>(~/.omh/routing/model-chains.json)"]
        OMHMemory["Audited Long-Term Memory & Evidence Gate<br/>(Plan -> Code -> Test · verified)"]
    end

    subgraph ExecutionPlane["2. Coding & Tool Execution Harness (Oh My Pi - omp v18.12.6)"]
        OMPRoles["OMP Role Dispatcher<br/>(default | smol | slow | plan)"]
        OMPTools["Native Rust & IDE Tooling<br/>(hashline edit, grep, glob, LSP, omp-shell)"]
        OMPRetry["Rate-Limit Retry & Fallback Engine<br/>(Exponential Backoff + Cooldown Reversion)"]
    end

    subgraph InferencePlane["3. OpenRouter Live Free-Tier Cascade (Strict $0.00 Spend)"]
        Tier1["Tier 1 Primary Free Models<br/>thinkingmachines/inkling:free | poolside/laguna-s-2.1:free | nemotron-3-ultra-550b:free"]
        Tier2["Tier 2 Secondary Free Fallback<br/>cohere/north-mini-code:free | nemotron-3.5-super-vl:free | dots-3-note-preview:free"]
        Tier3["Tier 3 Tertiary Free Fallback<br/>recursive/flint-v1.0:free | gemma-4-31b-it:free | inkling-small:free"]
        Tier4["Tier 4 Terminal Meta-Router<br/>openrouter/openrouter/free"]
    end

    User -->|"/dag-ulw or /omh"| OMHRouter
    User -->|"Direct CLI / TUI"| OMPRoles
    OMHRouter --> DAG
    DAG --> OMHChains
    OMHChains -->|"Prepared Handoff"| OMPRoles
    OMPRoles --> OMPTools
    OMPRoles --> OMPRetry
    OMPRetry -->|"1. Head Attempt"| Tier1
    Tier1 -.->|"HTTP 429 / 503"| OMPRetry
    OMPRetry -->|"2. Fallback Hop 1"| Tier2
    Tier2 -.->|"HTTP 429 / 503"| OMPRetry
    OMPRetry -->|"3. Fallback Hop 2"| Tier3
    Tier3 -.->|"HTTP 429 / 503"| OMPRetry
    OMPRetry -->|"4. Last Resort"| Tier4
    OMPTools -->|"Execution & Test Receipts"| OMHMemory
    OMHMemory -->|"Verified Report"| User
```

---

## 2. Live Verified OpenRouter Free-Tier Catalog (`tools=true`, `reasoning=true`)

Queried directly from `https://openrouter.ai/api/v1/models` and verified against `omp`'s catalog, the following **14 live `:free` tool-calling models** form our active zero-cost pool:

| Model Selector (`omp` / `omh`) | Vendor Family | Context Window | Supported Thinking Levels | Primary Agentic Role |
| :--- | :--- | :--- | :--- | :--- |
| `openrouter/thinkingmachines/inkling:free` | Thinking Machines | 1,048,576 | `minimal,low,medium,high,max` | **OMP `default` & `plan`** / OMH `ultrabrain` & `deep` |
| `openrouter/thinkingmachines/inkling-small:free` | Thinking Machines | 1,048,576 | `minimal,low,medium,high,max` | **OMP `smol`** / OMH `unspecified-low` |
| `openrouter/nvidia/nemotron-3-ultra-550b-a55b:free` | NVIDIA | 1,000,000 | `medium,high` | **OMP `slow`** / OMH `architect` |
| `openrouter/poolside/laguna-s-2.1:free` | Poolside | 262,144 | `minimal,low,medium,high` | OMH `capable` & `unspecified-high` / Code specialist |
| `openrouter/poolside/laguna-xs-2.1:free` | Poolside | 262,144 | `minimal,low,medium,high` | Fast code subagent fallback |
| `openrouter/cohere/north-mini-code:free` | Cohere | 256,000 | `minimal,low,medium,high` | Code generation & edit fallback |
| `openrouter/nvidia/nemotron-3.5-super-vl:free` | NVIDIA | 1,000,000 | `minimal,low,medium,high` | OMH `visual-engineering` & multimodal reasoning |
| `openrouter/nvidia/nemotron-3.5-lightning:free` | NVIDIA | 1,000,000 | `minimal,low,medium,high` | OMH `quick` & low-latency subagents |
| `openrouter/dots-studio/dots-3-note-preview:free` | Dots Studio | 512,000 | `minimal,low,medium,high` | OMH `writing` & documentation synthesis |
| `openrouter/recursive/flint-v1.0:free` | Recursive | 262,144 | `minimal,low,medium,high` | Multi-step code & reasoning fallback |
| `openrouter/xingchen-agi/xing4.0-29b-a4b:free` | XingChen | 262,144 | `minimal,low,medium,high` | General reasoning fallback |
| `openrouter/google/gemma-4-31b-it:free` | Google Gemma | 262,144 | `minimal,low,medium,high` | OMH `simple-work` & general fallback |
| `openrouter/google/gemma-4-26b-a4b-it:free` | Google Gemma | 262,144 | `minimal,low,medium,high` | Fast MoE fallback |
| `openrouter/openrouter/free` | OpenRouter Meta | 200,000 | `minimal,low,medium,high` | **Terminal Catch-All Free Router** |

---

## 3. Rate-Limit Retry & Fallback State Machine Specification

```mermaid
stateDiagram-v2
    [*] --> DispatchPrimary: Select Role/Category Head (:free)
    DispatchPrimary --> Success: HTTP 200 (Stream Complete)
    DispatchPrimary --> RateLimited: HTTP 429 / Quota / Overloaded (502/503)
    DispatchPrimary --> TransientError: Network Timeout / ECONNRESET

    TransientError --> BackoffRetry: Attempt <= maxRetries (10)
    BackoffRetry --> DispatchPrimary: Wait baseDelayMs * 2^attempt (1.5s..60s)

    RateLimited --> MarkCooldown: Record Provider Cooldown + Check Retry-After
    MarkCooldown --> AdvanceChain: retry.modelFallback == true (fallbackOn: any)
    AdvanceChain --> DispatchFallback: Select Next Candidate in fallbackChains[role]
    DispatchFallback --> Success: HTTP 200 (Stream Complete)
    DispatchFallback --> MarkCooldown: HTTP 429 on Fallback Candidate
    AdvanceChain --> TerminalFreeRouter: Chain Exhausted -> openrouter/openrouter/free
    TerminalFreeRouter --> WaitUsageReset: If all free backends throttled (waitForUsageReset: true)
    WaitUsageReset --> DispatchPrimary: Cooldown Expired (fallbackRevertPolicy: cooldown-expiry)
    Success --> [*]
```

---

## 4. Active Configuration Files

- **OMP Global Config:** [`~/.omp/agent/config.yml`](file:///data/data/com.termux/files/home/.omp/agent/config.yml)
- **OMP Credential Environment:** [`~/.omp/env`](file:///data/data/com.termux/files/home/.omp/env)
- **OMH Category Model Chains:** [`~/.omh/routing/model-chains.json`](file:///data/data/com.termux/files/home/.omh/routing/model-chains.json)
- **Hermes Agent Config:** [`~/.hermes/config.yaml`](file:///data/data/com.termux/files/home/.hermes/config.yaml)
