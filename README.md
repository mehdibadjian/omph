# OMPH — Zero-Cost Resilient Agentic Workflow (`Oh My Pi` + `Oh My Hermes` + OpenRouter Free Tier)

This repository contains the Business Requirements Document (BRD), Technical Architecture Specification, and production configuration templates for running **Oh My Pi (`omp`)** and **Oh My Hermes (`omh`)** exclusively on **OpenRouter's `:free` models** with multi-vendor rate-limit retry and automatic failover cascades.

## Repository Contents

- [`BRD.md`](./BRD.md) — Business Requirements Document (`BRD-OMH-OMP-OR-2026-001`)
- [`SPEC.md`](./SPEC.md) — Technical Architecture & Implementation Specification (`SPEC-OMH-OMP-OR-2026-001`)
- [`omp-config.example.yml`](./omp-config.example.yml) — `~/.omp/agent/config.yml` template for **Oh My Pi (`omp`)** with free-only allowlists, concurrency caps, and 7-hop `:free` fallback chains
- [`omh-model-chains.json`](./omh-model-chains.json) — `~/.omh/routing/model-chains.json` (`mixture_chain_overrides/v1`) mapping all 12 **Oh My Hermes** workflow lanes to free OpenRouter model chains
- [`hermes-config.example.yaml`](./hermes-config.example.yaml) — `~/.hermes/config.yaml` template with OpenRouter `max_price: { prompt: 0, completion: 0 }` gateway enforcement
- [`.env.example`](./.env.example) — Environment variable template for `OPENROUTER_API_KEY`

## Quick Setup

```bash
mkdir -p ~/.omp/agent ~/.omh/routing ~/.hermes
cp omp-config.example.yml ~/.omp/agent/config.yml
cp omh-model-chains.json ~/.omh/routing/model-chains.json
cp hermes-config.example.yaml ~/.hermes/config.yaml
```
